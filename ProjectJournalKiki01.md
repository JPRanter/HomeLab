# Project Journal: Kiki
## Entry 01: Scope Before Code

---

The most useful thing I built for Kiki wasn't code. It was a list of things she isn't allowed to be.

I had a working setup already: a local model on a flash drive that could answer questions from a folder of notes. What I didn't have was a reason for the next line of code, or the one after that. I've watched enough projects (mine included) die of the same disease. Somebody keeps building before anybody has written down what the thing is for, and six weeks later there's a lot of software and no answer to the question. So before the build went any further, there was a page. What problem, for whom, what she must do, what she must never do, and what I didn't have yet.

This journal is the record of that build, one decision at a time. Not the code. The calls behind it. *[On LLMs: Git Is the Memory](GitIsTheMemory.md)* covers the plumbing (git as the memory, hubs and spokes), so I won't redo it here. This is the part that came first.

---

## The Problem, Stated Plainly

My context was scattered, and it wasn't mine.

Years of notes, decisions, configs and half-finished thinking, spread across vendor memory, old chat logs, flash drives and every machine I own. Each new session with a model started either cold or warm from a stale copy. The vendor memory that fixed the paste also decided on its own what was worth remembering, and it lived in a building I don't own.

The problem statement I wrote down was one sentence: **I need my own context, answerable on demand, on hardware I own, with no subscription standing between me and it.**

Every requirement after that is that sentence with the details filled in.

---

## Who It's For

Two users, and only one of them exists yet.

The first is me. A sysadmin with a homelab, a second brain's worth of notes, and a habit of writing everything down because the next 2am outage won't care what I remember.

The second is a job. If I take side work (a camera install, a small office network), the context for that job should travel with me to the site, and nothing about anyone else's job should ride along. That user is hypothetical today. I scoped for it anyway, because bolting isolation onto a system that was built without it is how somebody's floor plan ends up in somebody else's answer.

Writing down a user who doesn't exist yet felt like overreach at the time. It shaped more of the design than anything else on the page.

---

## Non-Negotiables

Four things she has to be. If a design choice breaks one of these, the choice loses.

- **Offline.** No internet, no answers is a failure mode, not a feature. Field sites, basements and a dead WAN link are exactly when you need the notes.
- **Portable.** She runs from a flash drive, on nearly any PC, Windows or Linux. If she only works on my desk, she's a desk.
- **Owned.** Files I can read in a text editor, history I can read with git. No database only she can open, no format that dies with a vendor.
- **Modular.** Every feature sits behind a switch, and any piece (the model included) should be swappable without tearing up the rest.

The last one outranks almost everything, speed and cleverness included. A modular system that's a little slow can be fixed one part at a time. A fast one that's welded together gets thrown out whole. That's the requirement, anyway. Whether the build lives up to it is what the later entries are for.

---

## What She Isn't, and What I Didn't Have

The list of what Kiki isn't did more work than the list of what she is. She isn't a frontier model replacement: an 8B model on a flash drive won't design software, and the frontier model is still the contractor. She isn't a cloud product. No account, no sync service, and if it needs a server I don't own, it's out of scope. She isn't tied to one vendor; the local model, the AI helping to build her and the tools around her are all replaceable. And she doesn't decide what's worth keeping. I do. Every shiny idea since has been checked against that list first.

The constraints were just as plain. One person, nights and weekends, on a gaming desktop whose 16 GB graphics card caps how smart she can be. No developer background: I can read code, debug it and tell you when it's wrong, but writing a web app from scratch is a different trade. And the build budget is a frontier model subscription, a rented tool spent on building an owned one. On purpose.

That last one turned up the gap I didn't expect. A chat window can talk about files, but it can't touch them. It can't clone a repo, run a test, read a log or commit a fix, and every step of this build was going to be exactly that. I'd been carrying the work by hand between a chat and a shell: copy, paste, run, copy the error back. Me as the network cable. So the scope got one more line: **the build needs an agent that can act on the machine, under my supervision.** That's when Claude Code came in. Not because it was new. Because the requirements said the job couldn't be done without it.

Knowing what you don't have is a requirement too.

---

## Three Options, Two Already Tried

There were three ways to get there, and I'd already lived with two of them.

**Keep pasting.** Context blocks in a chat, every session. It's where *[Context Is the Cheat Code](ContextIstheCheatCode.md)* started, and it works. It also doesn't scale past what one person can paste, and it answers nothing when the chat isn't open.

**Rent the memory.** Vendor persistence, the *[Persistence Pivot](PersistencePivot.md)*. It fixed the paste and handed the keys to someone else.

**Build it.** Markdown and git for the memory, a local model to answer from it, a flash drive to carry it. Slower to get to, cheaper to keep, and mine.

Build it won, and it won on the non-negotiables, not on taste. The first two options fail "owned" and "offline" before you get to anything else.

---

## What I'd Tell the Next Person

Write the non-goals first. Everybody knows what they want the thing to do. Almost nobody writes down what it must never become, and that's the list that says no for you at 11pm when a feature looks fun.

If you can see a user coming, name them before they arrive. It's cheaper to build the wall before the room fills up.

And inventory your tools like they're part of the requirements, because they are. If the tool you're planning with can't do the work, that's not a footnote. That's a finding.

Next entry: the build process, and the rule that has saved the most time so far. Search for the decision before you build the thing.

---

*Project Journal is a running record of building Kiki, my offline second brain. Claude (Anthropic) helped turn the brain dumps into a written scope and helped write this entry up. The problem, the users, the requirements and the cuts are mine.*
