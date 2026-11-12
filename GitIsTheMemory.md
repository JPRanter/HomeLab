# On LLMs: Git Is the Memory
*Or: the most boring answer was the right one*

---

I pay for a frontier model now. Not to talk to it. To build the thing that means I won't need it for this.

That's a change from where this series started. *Context Is the Cheat Code* was built on the free plan, and the free plan proved the method: write down who you are, paste it in, and the model starts from somewhere useful. It also hit a ceiling I couldn't engineer around. A build this size keeps a lot of context live at once (decisions, code, the state of three machines, whatever the last session broke), and the free tier runs out of room long before the work does.

So the subscription is the build budget. And the most important tool in the build is older than every model in it.

---

## The Commute, Again

*The Persistence Pivot* fixed the paste. I moved my context into Gemini's persistent memory, skipped the morning briefing, and said plainly what it cost: the memory now lived in someone else's building. I called it an unfair deal and took it with open eyes.

It worked. It also had two costs I didn't price in.

The first is that a memory I rent is a memory I can lose: to a pricing change, a policy change, a product that gets folded into something else next quarter. Everything I'd taught it was sitting on a shelf I didn't own.

The second is that persistent memory is selective, and the model does the selecting. Say my hair is red for half the year and green for the other half, and I want that remembered. The memory decides it's irrelevant and quietly drops it. To me, the human, it matters. It's presumptuous for a chat engine to decide what I think is important, when that call is supposed to run the other way.

Put those together and the shape of the deal gets clear. The faceless megacorp I trusted with my whole personal database has decided what's worth keeping, which is another way of saying it's decided the data is theirs, not mine.

Offline is owned. Not rented, and not given away. The memory should live where I live.

---

## Too Many Copies

The first attempt at owning it went about how you'd expect.

Context files on the desktop. Context files on the laptop. A copy on a flash drive, a copy in a cloud session, a copy pasted into a chat three days ago that was now quietly wrong. Each one went stale on its own schedule. A session would start cold, or worse, start warm from an old copy and argue with decisions the last session had already made.

I'd solved the paste and recreated it at scale. The only way to get a new session current was to feed it sixteen hours of context copypasta, and the only way to know which copy was current was to remember. Remembering was the job I was trying to hand off.

---

## Git Is the Memory

The fix was the most boring tool on the machine.

One git repository is the hub, and it's the only memory. That's the rule, and it's written into the repo itself: nothing durable lives only in a chat, a vendor's memory, or my head. If it matters, it's committed. If it isn't committed, it didn't happen.

What that buys is a running record instead of a snapshot. Every decision has a date and a name on it. Every change is a diff. The log is the history of the project and the history of the thinking behind it, and keeping it costs nothing extra. A new session doesn't need to be told what happened. It reads what happened.

Then the hooks. Every session pulls everyone else's work the moment it starts, and again on every prompt. A resume file says where the last session stopped and what's next. Nobody briefs anybody. I open a session, say where I'm working, and we're moving inside of a sentence.

The real test came when I started multitasking. A terminal session on one machine and a cloud session somewhere else, both working the same brain at the same time. Commits collided. Two sessions incremented the same counter and both were sure they were right. Git did what git does: it refused to guess. It stopped, showed me both sides, and the merge got resolved by someone who could read them. That's the kind of bug you only get when a system is actually being used, and it's exactly the kind git was built for.

And it carries. The whole brain, every branch and all of its history, fits in a single bundle file on a flash drive and restores with GitHub nowhere in the picture.

None of this is clever. Git is twenty years old, it's on every machine I own, and it was designed to keep an honest record across people who aren't in the same room. Sessions that aren't in the same room have the same problem. The most boring answer was the right one.

(For the record on who did what: the hooks, the sync scripts and the merge plumbing are Claude's work. The rule that the repo is the only memory is mine.)

---

## Not Obsidian

Every second brain needs a face, and the obvious one was Obsidian. A colleague liked it. So did I. It's a good program and it reads markdown natively, which is most of the job.

I didn't bundle it, and the reason was arithmetic. Every copy of the brain I hand out would have to carry the program with it, and keep it current, on every machine it lands on. Meanwhile every one of those machines already has a web browser. Shipping someone else's application to replace something that's already installed is a cost that only goes up.

So we built the front end instead. Three panes: the file tree on the left, the page or a graph in the middle, chat on the right. Fully offline. My web dev skills are rusty enough to qualify as archaeology; Claude Code's aren't. What would have been weeks of me relearning CSS by hand turned into me making the decisions and Claude writing the code.

There's one detail I like more than I should. The front end has a hub view: a map of the brain itself, every file and every link between them. The tool that replaced Obsidian's graph now draws the graph of the memory that built it.

---

## Hub and Spokes

The brain is called Kiki, and Kiki is the hub. Everything else is a spoke.

A spoke is a copy scoped to a machine or a job, and git is what carries work between them. A spoke gets only what it should see. It lives on a branch with no shared history, so nothing it shouldn't have can ride along in the git objects. It runs offline, with a local model, on nearly any PC, Windows or Linux.

Say the job is a camera install at a cabinet shop. The spoke goes to the site carrying that job's context and nothing about anyone else's. It works without the internet. When it comes home, it syncs back to the hub, and the next job starts current. That's the design, and the round trip is the whole point of it.

The proof fits in my pocket. I keep a fully featured demo spoke on a flash drive, Windows and Linux both, because a system you can only describe is a system nobody believes.

---

## What's Actually Being Replaced

I want to be careful here, because this is where pieces like this usually start lying.

An 8B model on a flash drive does not replace a frontier model for building things. It can't. The frontier model wrote most of the code in this system, and it's still the one I reach for when something needs designing.

What Kiki replaces is the dependency. My context, my decisions and my record, answerable offline, on hardware I own, by a model that doesn't need a subscription to remember me. The frontier model is the contractor. Kiki is the house. Git is the deed.

That distinction gets sharper the moment the memory holds anything that isn't mine. A floor plan. A camera layout. Somebody else's network. Keeping my own notes out of a vendor's cloud is a preference. Keeping someone else's out of it is an obligation.

---

## The Misses Are on the Record

AI makes mistakes. I see it all the time, and I have my own frustrations with LLMs that this piece isn't going to pretend away.

The second brain doesn't stop hallucinations. What it does is make them minor and trackable. Auditable. A wrong answer doesn't vanish into a chat window. It lands in a diff with a date on it, gets caught, and gets fixed in a commit that's still sitting in the log.

Two from the same afternoon I outlined this piece. I asked Claude to reconnect to the brain, and it went looking for the repo as a folder on my C: drive before it found the actual repository. Later it "fixed" a label in my own writing spec into the wrong thing. Both caught in minutes. Both fixed in one commit each. Both still on the record, which is the point.

Resuming with the right context has become almost thoughtless. Carefree, even. The model reads where we are instead of me reciting it.

Engineering this tool has given me an appreciation for how these models are designed and how they interact with the world. Not a conversion. A working respect, earned the way you earn it for any tool: by watching it fail and seeing what it takes to make the failure cheap.

---

## Give It Somewhere to Live

*Context Is the Cheat Code* ended by telling you to give the model something real. That's still the advice. This is the second half of it: give it somewhere to live that belongs to you.

If you're paying for a subscription, it's worth asking what you're actually spending it on. Mine is being spent on its own exit, one commit at a time, in a repo I can carry out the door.

Nothing about that is specific to me. Anyone whose memory shouldn't live in someone else's building can have one. The tools are free, the oldest one matters most, and the record keeps itself.

---

*Written with Claude (Anthropic). The rules, the architecture and the calls are mine: git as the only memory, no bundled Obsidian, hub and spoke. The plumbing, the front end and the tests are Claude's. The commit log keeps the receipts, the good ones and the bad ones.*
