# JP Panter — Homelab

This repository is a practitioner's log. Not a showcase. Not a portfolio dressed up to look like one.

Every piece here started as a problem. Something didn't work, or worked badly, or worked until it didn't. I dug into it, figured out what was actually happening, built something that fixed it, and wrote it up — because the next person who hits the same wall deserves a shorter detour than I got. That's the whole model. It's been consistent since the first commit.

If you're here because of a resume or a cover letter: welcome. The writing is the extension of the application. What a bullet point can't show you, this can.

---

## On Using Claude

This lab was built with Claude (Anthropic) as an active collaborator, and that belongs at the top of this document.

The hardware decisions are mine. The architecture is mine. The reasoning behind every major call — Proxmox over ESXi, workload isolation across three nodes, the network edge design — those came from my understanding of what I was building and what I needed it to do. Claude did not design this environment.

What Claude did was work alongside me through the implementation. Troubleshooting sessions that might have taken an evening of trial and error got resolved through a conversation where I described the behavior and we reasoned through it together. Configuration that would have meant hours of documentation diving got done in a focused session. When something broke in a way I hadn't seen before, Claude was the knowledgeable colleague I could think out loud with at any hour.

That is not a crutch. That is how good engineers work — they use the tools available to them, they stay in the driver's seat, and they make sure they understand what they're building well enough to own it when something goes wrong. I own this stack. I can troubleshoot it, explain every decision in it, and rebuild any part of it. Claude helped me build it faster and better than I would have alone.

I say this openly because the industry is moving toward this model whether it's ready to acknowledge it yet or not. Knowing how to use AI tooling as an accelerant — without losing architectural ownership or surrendering your own judgment — is a skill. This repository is evidence of what that looks like in practice.

---

## What's Here

### Infrastructure and Operations

**[The Sysadmin Stick](AdminStick.md)**
A persistent live Linux environment on a Ventoy USB drive — one stick, any machine, familiar environment. Three documented dead ends before the architecture that actually worked. The dead ends are in there because they're useful, not because I'm proud of them.

**[Fixing HTTPS in a Homelab](TunneledMigrane.md)**
Five separate failure modes across two debugging sessions on the same Nginx Proxy Manager and Cloudflare tunnel stack. The diagnostic approach that worked each time: map the full traffic path, identify which layer the failure is actually at, check the logs there — not at the layer surfacing the symptom.

**[Network Segmentation](NetworkSegmentation.md)**
From a flat LAN to structured infrastructure. Threat model reasoning throughout, because "I should add VLANs" is not a decision — it's a starting point. The reasoning that gets you to the actual decision is the useful part.

**[KDE Wayland Per-Monitor Wallpaper](WallpaperChanger.md)**
Narrow problem, specific environment. Three tools failed before landing on the native DBus interface as the correct path. Two non-obvious requirements that aren't in KDE's documentation, found through the debugging session. Written up because this problem has no good answer anywhere and it should.

### On LLMs

**[On LLMs: Using Them as Engineering Tools](EngineeringwithLLM.md)**
*Or: why your ego is the most expensive thing in your stack*
The practical case for treating LLMs as reference tools rather than magic or threat. What they're good at, where they fail, and why the engineers who pretend they don't exist are making a choice they'll eventually regret.

**[On LLMs: Context Is the Cheat Code](ContextIsTheCheatCode.md)**
*Or: I didn't know it was called prompt engineering*
How the context block system in this repo works, why it works, and what a week of building it on the free plan actually produced. The Lego metaphor came naturally because that's what it felt like.

### Reference

**[The Admin Stick Setup Guide](AdminStick.md)**
Covered above. Worth calling out separately because the configuration files are complete and reusable — Ventoy JSON, persistence.conf, Firefox user.js for RAM-only cache. Take what's useful.

---

## The Infrastructure

Three-node Proxmox cluster. Each node reflects the nature of its workload and the hardware available to run it.

**pve-home** — Core node. Frankenstein tower, spare parts, built over several years. The node that cannot go down. Nginx serving the personal site via Cloudflare tunnel, Nextcloud replacing Google Drive and Docs, a 14TB NAS, a management VM that lives inside the cluster it manages.

**pve-games** — Repurposed gaming laptop with a functioning GPU and a non-functioning screen. Runs Pterodactyl and the game server containers it provisions. The failed screen is the only thing wrong with it.

**pve-util** — Salvaged workstation, RAM-upgraded from a donor machine. Runs the network edge container, Trilium, a book manager, a finance tracker, Mumble, and a Rocky Linux VM for RHCSA prep.

The network edge container is where Pi-hole, Nginx Proxy Manager, Homarr, Tailscale, and WireGuard all live — each in its own Docker container inside the LXC, sharing the network namespace, independently manageable. That architecture came from a dependency coupling problem that took the whole stack down when any single service went sideways.

Nothing is publicly exposed except the personal website. Remote access goes through Tailscale. The attack surface is small by design, not by accident.

---

## What's Next

VLAN segmentation is decided and the hardware exists — managed switch and an OpenWRT-flashed router are both in the environment. Implementation is the next project, not an unknown quantity.

Caddy is replacing Nginx Proxy Manager. The tunnel piece documents why NPM became the problem. The Caddy migration will get its own write-up when it's done.

RHCSA is in progress on the utility node.

---

*Written and maintained with Claude (Anthropic) as a documentation collaborator. The decisions, the dead ends, and the opinions are mine. Claude helped me build it faster and write it more clearly than I would have alone.*
