# Homelab Infrastructure Overview

This is not a lab I built from a shopping list. It grew the way most 
real infrastructure grows — out of necessity, available hardware, and 
a series of problems that needed solving. What started as a Debian box 
with a lot of hard drives has become a three-node Proxmox cluster 
running 13 LXC containers and 3 VMs, each placed deliberately, each 
solving something specific.

This repository is the roof. Individual projects have their own 
repositories with their own documentation — this is where the 
architecture lives, and where the reasoning behind it is explained.

---

## Origin

My background is break/fix. I learned hardware by repairing it, and 
Linux the same way — something needed to work, and I figured out how 
to make it work. The machine that is now my core node started as a 
tower I built from spare parts because I needed network storage. So I 
built a NAS. It ran Debian, it did its job, and that was enough.

Then it wasn't enough. I started experimenting with virtual machines 
because the concept was interesting, and once I understood what 
virtualization could do, I wanted more of it with less overhead. That 
search led me to hypervisors, and I chose Proxmox because it is 
Debian-based. I already knew the conventions, the package manager, the 
file structure. That familiarity turned out to matter more than I 
expected — when something breaks at 11pm, knowing where to look is 
half the problem solved.

That decision has held up through every iteration of this environment.

---

## On Tooling: Claude as Collaborator

This homelab was built with Claude (Anthropic) as an active and 
significant collaborator, and that fact belongs in this document.

The hardware decisions are mine. The architecture is mine. The choice 
of Proxmox over ESXi, the decision to isolate workloads across three 
nodes, the network edge container and the reasoning that produced it — 
those are my calls, made from my understanding of what I was building 
and what I needed it to do. Claude did not design this environment.

What Claude did was work alongside me through the implementation. 
Configuration that would have meant hours of documentation diving got 
done in a focused session. Troubleshooting that might have taken an 
evening of trial and error got resolved through a conversation where I 
described the behavior and we reasoned through it together. When 
something broke in a way I hadn't seen before, Claude was the 
knowledgeable colleague I could think out loud with at any hour.

That is not a crutch. That is how good engineers work — they use the 
tools available to them, they stay in the driver's seat, and they make 
sure they understand what they're building well enough to own it when 
something goes wrong. I own this stack. I can troubleshoot it, explain 
every decision in it, and rebuild any part of it. Claude helped me 
build it faster and better than I would have alone.

I document this transparently because the industry is moving toward 
this model whether it acknowledges it yet or not. Knowing how to use 
AI tooling as an accelerant — without losing architectural ownership 
or surrendering your own judgment — is a skill. This homelab is 
evidence of what that looks like in practice.

---

## The Cluster

Three nodes, three distinct purposes. The separation is not arbitrary — 
each node reflects the nature of its workload and the hardware 
available to run it.

### pve-home — Core Node
*Frankenstein tower, built from spare parts over several years*

This is the node that cannot go down. It houses the services that 
define what this homelab actually is: a self-hosted alternative to 
cloud dependency, a personal web presence, and the storage backbone 
that everything else touches. It is as integrated into daily life as 
the fiber connection and the gateway router — it is infrastructure, 
not an experiment.

| Service | Type | Purpose |
|---|---|---|
| nginx | LXC | Personal website, served via Cloudflare tunnel |
| Nextcloud | LXC | Self-hosted replacement for Google Drive and Docs |
| NAS | LXC | SMB file server, 14TB of segmented storage |
| MX Linux | VM | Management desktop — internal browser, cluster access |

The Cloudflare tunnel exposes exactly one thing to public traffic: the 
website. Nextcloud, the NAS, and everything else on this node is 
LAN-only. That boundary is intentional. The NAS is purely storage — 
no compute workloads run on it, and nothing lives on it.

The management VM deserves a note. Rather than context-switching 
between a daily driver and infrastructure work, I run a lightweight 
desktop environment inside the cluster itself. It has bookmarks for 
every browser-based application in the environment, serves as the 
primary interface for cluster management, and keeps project research 
and testing contained. It lives inside the thing it manages, which 
keeps access simple and my personal machine out of the equation.

Nextcloud also represents something this homelab is fundamentally 
about: the question of whether cloud dependency can be replaced with 
something owned, understood, and controlled. Nextcloud replaced Google 
Drive and Docs. The answer to that question turned out to be yes, with 
tradeoffs I have chosen to accept and understand deeply.

### pve-games — Games Node
*Repurposed gaming laptop — functioning GPU, non-functioning screen*

This node did not start as a games node. It was the second machine I 
added to the cluster, originally running Nextcloud and serving as an 
Active Directory test environment. The hardware sat underutilized until 
someone asked if I could host a game server.

The answer was obvious once the question was asked. Multi-core 
hyperthreaded processor, adequate RAM, a GPU that works fine without a 
screen attached — this machine was built to run games. It just needed 
the right workload. The failed screen is the only thing wrong with it.

| Service | Type | Purpose |
|---|---|---|
| Pterodactyl | LXC | Game server management panel |
| Game servers | LXCs | Multiple game servers, provisioned and managed via Pterodactyl |

Pterodactyl handles provisioning, resource allocation, and management 
for individual game server containers. Each server runs in isolation.

### pve-util — Utility Node
*Salvaged workstation, RAM-upgraded from a donor machine*

Two old dual-core workstations. One donated its RAM to the other. The 
resulting machine has enough memory for applications that are RAM-heavy 
but do not demand sustained CPU — which describes most of what runs on 
it. It is also, by a comfortable margin, the most stable hardware in 
the cluster.

| Service | Type | Purpose |
|---|---|---|
| Network edge | LXC | See below — this one gets its own section |
| Trilium | LXC | Self-hosted personal knowledge base |
| Book manager | LXC | Self-hosted reading library |
| Finance manager | LXC | Self-hosted personal finance tracking |
| Mumble | LXC | Self-hosted voice communication |
| Rocky Linux | VM | RHCSA exam preparation, CLI only |

---

## The Network Edge Container

This container has its own section because it did not arrive at its 
current architecture all at once. It evolved, and the evolution is 
worth documenting because each step was driven by a real problem.

It started as separate containers — Pi-hole, Nginx Proxy Manager, and 
Homarr each running independently. The problem that emerged was 
dependency coupling. Pi-hole handles local DNS. NPM depends on DNS to 
resolve the services it proxies. Homarr depends on both. When Pi-hole 
went down, it did not just take DNS with it — it took the reverse proxy 
and the dashboard as well. Three services that failed together were 
functionally one service that did not know it yet.

The first solution was a single Docker Compose configuration. That 
solved the dependency problem and created a new one: any time a single 
service needed troubleshooting, the entire stack came down with it. For 
the container responsible for DNS and reverse proxying across the whole 
LAN, that was not acceptable.

The current solution is nested containers. Each service runs in its own 
Docker container inside the LXC, sharing the LXC's network namespace. 
They can be managed, restarted, and rebuilt independently. The overhead 
stays low, the isolation is real, and bringing down one service does not 
touch the others.

The LXC itself runs Tailscale and WireGuard directly, making it the 
single point of presence for both remote access and VPN policy routing. 
This is also where the work documented in the Tailscale repository 
lives — the subnet router, exit node, and WireGuard policy routing 
groundwork are all configured at this layer.

**Running inside the network edge LXC:**
- **Pi-hole** — DNS resolution and ad blocking for the entire LAN
- **Nginx Proxy Manager** — reverse proxy for all internal services
- **Homarr** — internal dashboard
- **Tailscale** — zero-trust remote access and subnet routing
- **WireGuard** — policy routing groundwork for commercial VPN integration

---

## Network Architecture

The current network is intentionally simple: flat topology, home router 
handling DHCP and serving as the default gateway, 500Mbps fiber with a 
non-ISP-supplied gateway router.

Internal service access follows a consistent path:
```
Client → Pi-hole (local DNS) → NPM (reverse proxy) → service
```

Public traffic follows a narrow, deliberate path:
```
Internet → Cloudflare tunnel → nginx → personal website
```

Nothing else is publicly exposed. Remote access from outside the LAN 
goes through Tailscale. The attack surface is small by design, not 
by accident.

A managed switch and an OpenWRT-flashed router are both present in the 
environment as proof of concept. VLAN segmentation is the natural next 
step — the hardware exists, the configuration has been tested, the 
implementation is a future project rather than an unknown quantity.

---

## What This Is Actually For

The homelab exists to answer questions. Can cloud dependency be 
replaced with something self-hosted? Can a broken gaming laptop become 
useful infrastructure? Can salvaged workstation hardware run a reliable 
network edge? The consistent answer has been yes, given enough 
understanding of what the hardware can do and what the software needs.

The deeper answer is that this environment exists because I learn by 
doing, and I have always learned by doing. Break/fix teaches you that 
systems are understandable — that there is always a reason something 
works or doesn't, and that finding the reason is a matter of knowing 
where to look and being willing to look there. That instinct built this 
lab, and this lab has sharpened that instinct considerably.

The secondary purpose is deliberate skill development that maps to 
professional infrastructure work. Hypervisor selection, workload 
isolation, service dependency management, zero-trust networking, 
reverse proxy configuration — none of this was studied in isolation. 
It was learned because something needed to work. The Rocky Linux VM 
running RHCSA prep on the utility node is the most explicit version 
of that, but it describes the whole environment.

---

## Repository Index

| Repository | Description |
|---|---|
| [tailscale-lxc-exit-node](../tailscale-lxc-exit-node) | Tailscale subnet router and exit node on Proxmox LXC with WireGuard policy routing groundwork |

*Additional repositories will be linked here as they are published.*

---

## What's Next

- VLAN segmentation using the existing managed switch and OpenWRT router
- Backup and disaster recovery documentation
- RHCSA certification

