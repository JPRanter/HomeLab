# Network Segmentation: From Flat LAN to Structured Infrastructure

A three-node Proxmox cluster running 13 LXC containers and 3 VMs sat on a flat /24 for long enough that it became difficult to reason about. Public-facing containers shared a network with a 14TB NAS. Game servers shared a broadcast domain with workstations. The DHCP pool was a mess of devices nobody had catalogued in months.

This is the story of auditing that network, designing a segmentation architecture, learning exactly where the limits of a single-NIC node are, and stopping cleanly when the hardware couldn't support the design.

---

## Step 1: Know What You Have

The starting point was a full nmap scan of the existing /24. The naive first attempt returned nothing:

```
sudo nmap 192.168.1.0
```

Two problems. First, `192.168.1.0` is the network address, not a host. Second, nmap's default behavior is to ping-probe first and skip hosts that don't respond — which describes most firewalls, consumer electronics, and anything with ICMP blocked. The correct invocation:

```
sudo nmap -Pn -sV -O -A -T4 192.168.1.0/24 -oN network_map.txt
```

- `-Pn` — skip the ping probe, assume hosts are up
- `-sV` — probe open ports for service and version
- `-O` — attempt OS detection
- `-A` — combines OS detection, version detection, script scanning, and traceroute
- `-T4` — faster timing

That returned 26 live hosts. A few things came out of the results worth noting.

The `BC:24:11` MAC prefix appears on the majority of hosts. That OUI belongs to Proxmox — every address showing that prefix is an LXC container or VM on the cluster. Identifying that pattern reduced the unknown host count significantly.

Several devices assumed to be phones turned out to be smart TVs. The verbose scan fingerprineted them by model via UPnP and AirTunes service banners. Assumptions about a flat network are unreliable. A scan before making architectural decisions is not optional.

The game servers share identical SSH host keys. They were cloned from a template. Pterodactyl manages all of them.

Working through the results took time. Every host got documented: IP, MAC, hostname, OS, services, and a target segment assignment. That inventory became the source of truth for everything that followed.

---

## Step 2: Design the Segments

With a complete inventory, the threat model was straightforward. Public-facing containers are reachable from the internet through a Cloudflare tunnel. They have no business being on the same flat network as the NAS, Proxmox management interfaces, or workstations. Segmentation was the answer.

The architecture: an internal segmentation firewall (OPNsense) with its WAN interface on the trusted LAN, routing downstream into purpose-built segments. The existing gateway remains the internet gateway and DHCP server for trusted LAN devices. Nothing changes for them. The firewall only handles traffic destined for new segments.

Planned segments:

| Segment | Subnet | VLAN | Purpose |
|---------|--------|------|---------|
| Trusted LAN | 192.168.1.0/24 | 1 | Workstations, NAS, Proxmox nodes |
| Management | 10.10.10.0/26 | untagged | Management VMs |
| Public-facing LXCs | 10.10.10.192/26 | 10 | Cloudflare tunnel ingress, NPM, public services |
| Game Servers | 10.10.20.0/24 | 20 | Isolated game server LXCs |
| IoT | 10.10.30.0/26 | 30 | Smart TVs, Ring, Nintendo Switch |
| IP Cameras | 10.10.30.128/26 | 35 | Future — isolated camera traffic |

IoT and cameras are carved from a single /24 using /26 subnets. Two adjacent ranges for similar device types, no wasted /24 on 30 smart home devices. The game servers need specific ports forwarded for in-game connections but nothing else — default deny, explicit port-forward allows.

The DMZ is split into two /26 tiers: management at .0/26 and public-facing services at .192/26. The middle two /26 blocks (.64/26 and .128/26) are reserved for future expansion.

---

## Step 3: DNS and Reverse Proxy

Once services are on different subnets, internal access could get complicated. It doesn't, because of an existing pattern: Pi-hole handles local DNS for the entire LAN, and every hostname resolves to an Nginx Proxy Manager instance. NPM proxies to the correct backend regardless of subnet.

```
client → Pi-hole (local DNS) → NPM (reverse proxy) → backend on any subnet
```

A service moving from trusted LAN to the DMZ changes one field in NPM — the backend address. The DNS record stays. The user experience doesn't change. This decouples users and firewall rules from backend topology. Moving a service is an infrastructure operation, not a user-facing event.

The firewall implication is clean: the only path from the trusted LAN into the public-facing segment is NPM. One auditable rule. Nothing else needs a cross-segment path.

---

## Step 4: VLAN Plumbing

Three-node Proxmox cluster, one physical NIC per node. The switch (a Cisco managed switch) handles VLAN separation at L2. The architecture is standard 802.1Q trunk — one physical NIC per node carries both untagged trusted LAN traffic and tagged VLAN traffic. The switch sorts by tag. This is router-on-a-stick, not a workaround.

Each node gets a single VLAN-aware bridge (`vmbr0`) with `bridge-vids 2-4094`. LXC and VM interfaces attach to `vmbr0` with the appropriate tag for their segment. The switch trunk ports carry all relevant VLANs to all three nodes.

Validation: a test Alpine LXC with `tag=20` on `vmbr0`, confirmed with tcpdump:

```
tcpdump -i vmbr0 -n -e port 67 or port 68
```

Output showed `ethertype 802.1Q (0x8100), vlan 20` on outbound frames. Tagged traffic leaving the node correctly. Architecture validated.

---

## Step 5: Where It Stopped

Here is where the write-up diverges from "and then it worked."

OPNsense as a VM on a single-NIC Proxmox node cannot cleanly separate WAN from LAN when LAN needs to carry multiple VLANs.

The problem is specific. A Proxmox VM interface with `tag=X` receives only that one VLAN — it cannot serve as a trunk. A VM interface with no tag receives all VLANs untagged — which is identical behavior to the WAN interface. With one physical NIC and one bridge, there is no Proxmox-level mechanism to hand OPNsense a tagged multi-VLAN trunk on one interface while keeping WAN distinct on another.

What was observed: DHCP requests from VLAN 20 arriving on OPNsense's WAN interface (`vtnet0`) instead of the LAN interface. dnsmasq logging `no address range available for DHCP request via vtnet0` — receiving the broadcasts, finding no range defined for the WAN, dropping them. Frames were reaching OPNsense. They were arriving on the wrong interface.

Every workaround examined required either compromising the WAN/LAN separation or accepting that the firewall can't enforce segment boundaries in any meaningful way. Neither is acceptable.

The blocker is hardware, not configuration. A second NIC on the node, or a dedicated firewall appliance with proper interface separation, resolves it cleanly. The current attack surface is acceptable in the interim: Cloudflare tunnel handles all public ingress, LXC isolation handles internal separation, and no service migration happens until the architecture can be implemented correctly.

---

## Current State

The switch is configured and ready. VLAN trunk ports are set. Subnets are designed. The single-NIC trunk architecture works correctly for per-VLAN LXC tagging — that part is validated and production-ready. The segmentation firewall is parked until proper hardware is available.

The skills built here — VLAN design, 802.1Q trunking, OPNsense interface configuration, traffic path diagnosis with tcpdump — are what gets deployed correctly when the hardware supports it. Building it wrong to say it's done is not the point.

---

*Written with assistance from Claude (Anthropic) as part of an ongoing homelab documentation practice. The architecture decisions, the hardware, the direction, and the choice to stop at the right point are mine. Claude helped build it faster and document it more clearly than I would have alone.*
