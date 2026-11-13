# JP Panter

**I decide what gets built, and I've built enough of it to know what that costs.**

Most of what's here was paid for in the trenches: salvaged hardware, outages at bad hours, and documentation I swore I'd write later and didn't. That last one is the reason this repository exists.

I'm an engineer. I don't like writing documentation, and I've spent enough nights rebuilding things nobody wrote down to know that not liking it isn't an excuse. So I did the lazy thing, which turned out to be the smart thing. I built a bot to do it for me.

---

## What I'm Building

**Kiki** is an offline second brain. My notes, decisions and history live in markdown and git, and a local model answers from them on hardware I own, from a flash drive if it has to. No subscription, and no vendor deciding what's worth remembering. When the notes don't have the answer, she says so instead of guessing.

The **Project Journal** is the record of that build. One decision at a time: the problem, the requirements, the options, the call and what it cost. Not a code tour. The reasoning behind it.

---

## How I Work

I build with AI the way you'd run a crew. The problem, the requirements, the architecture and the final call are mine. Claude (Anthropic) writes a lot of the code and most of the first drafts. I review what ships, I can explain every piece of it, and when it breaks at 2am, I'm the one on call, not the model.

I say that out loud because pretending otherwise is the expensive choice. The skill isn't avoiding these tools. It's using them without handing over the wheel.

---

## The Work

### Project Journal: Building Kiki
Decisions, not code.

**[Entry 01: Scope Before Code](ProjectJournalKiki01.md)**
Before the build went any further: the problem in one sentence, who it's for, what it must never become, and the requirement nobody writes down.

### On LLMs
Essays on using AI as an engineering tool, in the order they were written.

**[On Using LLMs as Engineering Tools](EngineeringwithLLM.md)**
*Or: why your ego is the most expensive thing in your stack.* Treat LLMs as a reference tool, not magic and not a threat. What they're good at, where they fail, and why pretending they don't exist is a choice.

**[On LLMs: Context Is the Cheat Code](ContextIstheCheatCode.md)**
*Or: I didn't know it was called prompt engineering.* Writing down who you are so the model starts from somewhere useful. Built on the free plan, one block at a time.

**[On LLMs: The Persistence Pivot](PersistencePivot.md)**
*Or: trading privacy for a better memory.* Moving my context into a vendor's persistent memory to stop pasting it every morning, and naming what that cost.

**[On LLMs: Git Is the Memory](GitIsTheMemory.md)**
*Or: the most boring answer was the right one.* Taking the memory back. One git repo as the only memory, hubs and spokes, and the misses left on the record.

### Field Notes
Things that broke, what was actually wrong, and how they got fixed. The dead ends are included because they're the useful part.

**[Network Segmentation](NetworkSegmentation.md)**
From a flat LAN to a segmented one, with the threat model reasoning written out. "I should add VLANs" isn't a decision. It's where the decision starts.

**[Fixing HTTPS in a Homelab](TunneledMigrane.md)**
Five failure modes across two debugging sessions on one reverse proxy and tunnel stack. What worked every time: map the whole traffic path, find the layer that's actually failing, and read the logs there.

**[The Sysadmin Stick](AdminStick.md)**
A persistent live Linux environment on one USB stick that boots on any machine. Three dead ends before the design that worked, and complete config files you can take.

**[KDE Wayland Per-Monitor Wallpaper](WallpaperChanger.md)**
A narrow problem with no good answer anywhere. Three tools failed before the native DBus interface worked, and it needed two requirements KDE doesn't document.

**[Dolphin Thumbnails Were Fine. The JPEGs Were Not.](FakeJPGs.md)**
Missing image previews on a network share, traced to files that were never JPEGs in the first place.

---

## The Lab

It died. Three machines built from salvaged parts (a spare-parts tower, a gaming laptop with a dead screen, a workstation rebuilt from a donor) ran everything from a NAS to game servers to the network edge. Then a power outage broke the cluster, a disk failed outright, and Frankenstein hardware sent its bill.

The rebuild is slow on purpose: one machine at a time, each verified before the next one starts. The difference this time is that the lab has a memory to read from. Recovering a documented system is a checklist. Recovering an undocumented one is archaeology.

---

## Where This Is Going

More Project Journal entries as Kiki grows. The lab comes back with its documentation written first, not last. And the cert stack keeps climbing toward CISSP, because whoever owns the whole thing has to answer for its security too.

---

*Written with Claude (Anthropic). The decisions, the dead ends and the opinions are mine. Claude helped me build it faster and say it more clearly than I would have alone.*
