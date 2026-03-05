# Network Segmentation: From Flat LAN to Structured Infrastructure

*A homelab journey — discovery, planning, and implementation*

---

## Background

This document tells the story of a single evening's work: auditing a production homelab network, identifying every device and service on it, and laying the groundwork to segment it into something structured, secure, and maintainable.

The homelab in question is a three-node Proxmox cluster running 13 LXC containers and 3 VMs. It grew organically — out of necessity, available hardware, and a series of problems that needed solving. The network it sat on grew the same way: flat, functional, and increasingly difficult to reason about as the number of services expanded.

The immediate trigger for this project was a simple one: the DHCP pool was getting messy. But the real motivation runs deeper. Several of the services running on this cluster are public-facing through a Cloudflare tunnel. A compromised container on the same flat network as a NAS with 14TB of personal data is an unacceptable risk — not theoretical, just not yet addressed. Segmentation was the answer.

A new OPNsense appliance had just been provisioned. Before touching anything, the first task was to understand exactly what was on the network. That meant running a proper port scan and working through the results methodically.

---

## Step 1: Network Discovery with nmap

The starting point was a basic nmap scan of the entire existing flat subnet. The first attempt returned nothing useful:

```bash
sudo nmap 192.168.1.0
```

Result: *host seems down.* This is a common stumbling block. nmap's default behavior is to send a ping probe first and skip the host if it doesn't respond. Many devices — especially firewalls and consumer electronics — block ICMP ping by default. The fix is the `-Pn` flag, which tells nmap to assume the host is up and scan anyway. But the more important mistake was the target address itself: `192.168.1.0` is the network address, not a host. The correct target for a subnet scan is the CIDR range.

```bash
sudo nmap -Pn 192.168.1.0/24
```

That returned 32 live hosts. A second, more detailed scan with service version detection, OS fingerprinting, and aggressive probing filled in the picture:

```bash
sudo nmap -sV -O -A -T4 192.168.1.0/24 -oN network_map.txt
```

- `-sV` — probes open ports to identify the actual service and version running
- `-O` — attempts OS detection
- `-A` — combines OS detection, version detection, script scanning, and traceroute
- `-T4` — faster timing template
- `-oN` — saves output to a file for reference

What came back was a complete picture of the network — not just IP addresses and open ports, but application banners, hostnames, device types, and in several cases the exact software version running on each service.

---

## Step 2: Making Sense of the Results

Working through the scan results took time. Some devices identified themselves immediately. Others required cross-referencing MAC address OUI prefixes, banner strings, and open port combinations.

32 live hosts came back across the subnet — a spectrum of devices: network infrastructure, Proxmox cluster nodes, an array of LXC containers running self-hosted applications, Windows workstations, game servers, smart TVs, IoT devices, and a handful of mobile phones on the dynamic DHCP pool. Each one needed to be identified, documented, and assigned a target segment.

A few findings worth highlighting:

The `BC:24:11` MAC prefix appears on the majority of hosts. This is the OUI prefix assigned to Proxmox virtual machines — every address with that prefix is an LXC container or VM running on the cluster. Identifying that pattern reduced the unknown host count significantly and made it clear just how much of the subnet was cluster traffic.

Several hosts that were assumed to be mobile phones turned out to be smart TVs. The verbose scan fingerprinted them by model number via UPnP and AirTunes service banners — a good reminder that assumptions about a flat network are unreliable and that a scan is worth running before making architectural decisions based on what you think is there.

The game servers share identical SSH host keys across all instances — a tell that they were cloned from a template rather than provisioned individually. Pterodactyl, the game server management panel, handles provisioning for all of them.

The network edge container revealed its full service stack in the scan. This is the container that handles DNS, ad blocking, and reverse proxying for the entire LAN — seeing it laid out in a port scan is a useful reminder of how much responsibility sits in a single LXC.

---

## Step 3: Planning the Segmentation

With a complete inventory in hand, the segmentation plan came together around a clear threat model: the public-facing LXCs are all reachable from the internet through a Cloudflare tunnel. They have no business being on the same flat network as the NAS, the Proxmox management interfaces, or the Windows workstations.

The solution is an internal segmentation firewall. OPNsense sits inside the existing network with its WAN interface on the trusted LAN and routes downstream into purpose-built segments. The Netgear router remains the internet gateway and DHCP server for the trusted LAN. Nothing changes for existing devices. OPNsense only handles traffic destined for the new segments.

The topology:

```
Internet → Netgear (gateway) → SG250 (L2) → Proxmox cluster (trusted LAN)
                                                        ↓
                                                   OPNsense
                                                        ↓
                                  DMZ (10.10.10.x) | Game Servers (10.10.20.x) | IoT (10.10.30.x)
```

The planned segments:

| Segment | Subnet | Purpose |
|---|---|---|
| Trusted LAN | 192.168.1.0/24 | Workstations, NAS, Proxmox nodes, network appliances — catch-all trusted devices |
| DMZ | 10.10.10.0/24 | Public-facing LXCs routed through Cloudflare and NPM |
| Game Servers | 10.10.20.0/24 | Isolated game server LXCs — LAN and VPN access only |
| IoT | 10.10.30.0/27 | Smart TVs, Ring, Nintendo Switch — untrusted consumer devices |
| IP Cameras | 10.10.30.32/27 | Future — isolated camera traffic |

The IoT and camera ranges are carved out of a single /24 using /27 subnets. This keeps similar device types in adjacent address space without wasting a full /24 on 30 smart home devices. It also reflects intentional CIDR discipline — thinking in terms of actual host requirements rather than defaulting to /24 for everything.

The game servers sit in their own segment with a specific access requirement: they need to be reachable via direct IP for in-game friend connections, but should not be reachable from the internet for anything else, and should have no path to the NAS or management interfaces. The solution is port forwarding on OPNsense for specific game ports, with a default-deny policy for everything else.

---

## Step 4: DNS and Reverse Proxy — How Internal Routing Works

One of the more conceptually interesting pieces of this architecture is how internal service access works once everything is segmented. The answer is the combination of Pi-hole and Nginx Proxy Manager that already exists on the network edge container.

Pi-hole handles local DNS for the entire LAN. Instead of accessing services by IP address and port — which would break every time a service moved to a new segment — local DNS records point friendly hostnames to the NPM instance. NPM receives the request, reads the hostname, and proxies it to the correct backend regardless of what subnet it lives on.

```
client → Pi-hole (local DNS resolves hostname to NPM) → NPM → backend service on any subnet
```

A service can move from the trusted LAN to the DMZ without any change to how users access it. The DNS record and proxy rule stay the same — only the backend address in NPM changes. This pattern decouples users and firewall rules from backend topology. Moving a service is an infrastructure operation, not a user-facing event.

The firewall implication is clean: OPNsense only needs to allow traffic from the NPM instance into the DMZ. Nothing else from the trusted LAN needs a path to the public-facing containers. That is a single, auditable rule rather than a sprawling allow list.

---

## Step 5: Validating the OPNsense Configuration

With the plan established, the OPNsense appliance needed to be validated before any production services were moved. A temporary management VM on the DMZ network served as the test client.

The validation sequence:

**Confirm OPNsense is reachable on the LAN.** An nmap scan of its assigned address confirmed the host was up, with all ports filtered — correct behavior for a firewall's WAN interface. A firewall that responds to pings on its WAN interface is advertising its presence unnecessarily.

**Confirm the upstream gateway is reachable.** From OPNsense's diagnostics, a ping to the Netgear confirmed the WAN-side path was working. The initial gateway configuration showed an IPv6 address rather than the expected IPv4 upstream — this required creating a static gateway entry manually under System → Gateways before the WAN interface would route correctly.

**Confirm the management VM has internet access.** Opening a website in a browser from the VM confirmed that NAT, routing, firewall rules, and DNS resolution were all working end to end.

The management VM's purpose was fulfilled at that point. It will be decommissioned — it exists only to validate the DMZ configuration from inside the segment.

---

## What Comes Next

The groundwork is complete. OPNsense is stable, the DMZ segment is validated, and every device on the network has a documented home. The remaining work is execution:

- Create the remaining VLAN interfaces in OPNsense for game servers and IoT
- Configure VLAN-aware bridges in Proxmox
- Migrate LXCs to their target segments, starting with non-critical services
- Add a static route on the Netgear pointing `10.10.0.0/16` traffic to OPNsense
- Update Pi-hole DNS records as services move to new addresses

The Cisco SG250 stays in pure L2 mode for now. Its port-based VLAN assignment and PoE capabilities become relevant later, when IP cameras and additional access points need to land on specific segments based on which physical port they connect to.

The broader goal remains what it has always been in this environment: build something that works, understand why it works, and document the reasoning clearly enough that it can be explained, maintained, and rebuilt if necessary.

---

## A Note on Process

This entire session — from the first failed nmap command to a validated OPNsense deployment — was conducted collaboratively with Claude (Anthropic) as a technical partner. The architecture decisions, the hardware, the choice of tools, and the direction of the project are mine. Claude provided real-time technical guidance, helped interpret scan results, caught configuration issues, and served as the kind of knowledgeable colleague you can think out loud with at 10pm when something unexpected shows up in a port scan.

That is documented here for the same reason it is documented in the main infrastructure overview: using AI tooling as an accelerant — without losing architectural ownership or surrendering your own judgment — is a legitimate and increasingly relevant engineering skill. The person who understands what they built and why they built it owns the stack. The tool that helped them build it faster does not.
