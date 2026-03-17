# On LLMs: Context Is the Cheat Code
*Or: I didn't know it was called prompt engineering*

---

I didn't know it was called prompt engineering when I started doing it. I thought of it as Lego.

The first block goes down and it has to be solid, because everything else is going to stack on top of it. Get the foundation wrong and the whole house of cards comes down. Get it right and you can keep building indefinitely, each new piece snapping into place because it knows exactly what it's connecting to.

That's what I've been building for the last week. Across multiple sessions, multiple domains, multiple versions of documents that got too small for what they needed to hold and had to be rebuilt from the ground up. It didn't happen in a day. It started with a pile of dead leaves and accumulated into something that actually works.

---

## The Problem With LLMs Is Us

There's a common dismissal that goes something like: AI is just fancy autocorrect. And sure, at the mechanical level, these models are doing something predictive. But autocorrect doesn't have the entire length and breadth of human knowledge available at the press of enter.

Every session starts from zero. The model doesn't know who it's talking to, so it responds accordingly: generically, carefully, hedged for an audience of everyone. The output is only as specific as the input, and most people's input is a question with no context attached. You get a smart answer to a dumb prompt and wonder why it doesn't quite fit.

The fix isn't complicated. It's just work.

You explain yourself. In writing, by voice, however the words come out — the interface doesn't matter. What matters is that you build something that tells the model who you are, how you think, what you're trying to accomplish, and how you want to be talked to. Then you paste it at the start of every session, and instead of starting from scratch, you start from somewhere useful.

Here's the part that stops people before they start: they don't know what to say. Or they're worried about saying the wrong thing. Or they're worried, genuinely, about making it mad at them.

Nobody sees your context blocks but you and the model. You can pour your heart into them. You can be completely honest about who you are, where you came from, what you're afraid of, what you want. The model doesn't judge it. It uses it. And when the session ends, the model forgets everything — the only thing that persists is what you chose to write down.

Claude doesn't know I called it fucking dumb three days ago because it nearly wiped the data off my NAS with some bad CIFS commands. Today, it doesn't matter. The relationship that exists is the one I built in the documents. Everything else evaporated when the session closed.

Start honest. That's the only rule. The first block doesn't have to be big. It just has to be true.

---

## The Tree

The pile of dead leaves that preceded this system tells its own story. Dozens of one-off context paragraphs scattered across notes apps. Sticky notes that made sense when I wrote them and nothing three days later. The same background re-explained in session after session because nothing was persisting anywhere useful.

The mess came first. The system emerged from the mess, the way most good systems do. I took those dead-leaf prompts, fed them to the model, and asked it to help me build something coherent out of them. That was the first block.

What I landed on is a tree.

Layer 1 is the root. Who I am, how I communicate, what my voice sounds like when I write, what I'm building toward professionally and why. It doesn't change often. It's the thing everything else grows from, and it has to be solid before anything else goes on top of it.

From there, branches. One for the homelab. One for career work. One for certifications and learning. One for writing. Each branch knows its domain, what's currently active, what's been resolved, and where the last session left off. Below the branches, leaves — specific projects, active topics, granular concept work that gets updated constantly and eventually closes when the topic is done.

The deeper you go, the more specific and volatile it gets. The root updates maybe twice a year. The leaves update every session. Version numbers on every document, timestamps on every update, a naming convention that tells you exactly where in the tree something lives just by reading the filename.

The part that still makes me smile: I specified what I wanted those conventions to look like, and the LLM wrote them into every document automatically from that point forward. The system documents how to maintain the system. The context block instructs the model to version, timestamp, and name correctly, so the model does. The human never has to think about it. It just happens.

The context block built itself.

When a document gets too small for what it needs to hold, you rebuild it. That's what the version numbers are for. Not bureaucracy. Iteration.

---

## The Free Plan Proof of Concept

I built this on the free plan.

No persistent memory. No continuity between sessions. Every time I open a new conversation, the model has never met me. What I paste into that first message is the entirety of what it knows about who it's talking to.

That sounds like a limitation. It's actually the proof of concept. If the context blocks are good enough, the model doesn't need to remember you. The documents do the remembering. The model just needs to read them.

What a week of that work looks like on disk: twenty markdown files in a repo. Layer 1 through Layer 4. Career branches. Learning branches split by certification track. Homelab state. Writing standards. Each one built on top of the last, each one made easier to build because the model already had the context it needed to help grow the next branch. Less guesswork. Less hallucination. More signal, less noise.

And all I have to do now is copy and paste.

---

## The Compounding Effect

There's a compounding effect that doesn't show up until you're a few layers deep.

The first session is the hardest. You're building from nothing, explaining everything, establishing the foundation while simultaneously trying to get actual work done. The second session starts from where the first one ended. The tenth session starts from somewhere the first session couldn't have imagined.

The work gets faster. The output gets sharper. The model starts anticipating correctly instead of guessing. When it gets something wrong, you know exactly which document needs updating, because the system tells you where that information lives.

Paste three layers of context into the opening message and you're at top speed inside of a paragraph. No re-explaining the homelab architecture. No re-establishing what your voice sounds like. No re-litigating the cert stack. That context is already loaded, already versioned, already waiting. The session picks up where the documents say you are.

That's not something I could have built in one sitting. It accumulated. Session by session, layer by layer, each piece of context making the next piece easier to write because the foundation was already there to build on.

---

## The Terminology Caught Up

I didn't set out to learn prompt engineering. I set out to stop wasting the first twenty minutes of every session getting the model up to speed on who it was talking to. The Lego metaphor came naturally because that's what it felt like: find the right block, make sure it's solid, build up from there.

Turns out there's a whole discipline built around the same instinct. The terminology just caught up to the approach.

If you're using these tools and the output still feels generic, the context is probably the problem. Not the model. Not the prompt. The context. The model doesn't know you yet.

That's fixable. You don't need fifteen hours and a perfectly engineered system. You need one honest paragraph about who you are and what you're trying to do. Start there. The tree grows from the first block, and the first block just has to be true.

The model isn't keeping score. It isn't judging you. It isn't waiting for you to say the wrong thing. It's waiting for you to tell it something real.

Give it something real and watch what happens.

---

*Written with Claude (Anthropic). The instinct, the time at the keyboard, the Lego metaphor, and the reminder of what I wrote three hours ago are all accounted for between the two of us. The thoughts are mine. Claude helped me find the words and kept track of the ones I'd already used.*
