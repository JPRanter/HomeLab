# On Using LLMs as Engineering Tools

*Or: why your ego is the most expensive thing in your stack*

---

The word "AI" is doing a lot of heavy lifting right now, and most of it is fiction. AI is the behavior of enemies in a video game. AI is a Spielberg movie. AI is Skynet, it's the machines from the Matrix, it's every science fiction shorthand for technology that thinks, wants things, and eventually decides humans are the problem. What we actually have — Claude, ChatGPT, and the others — is something far less cinematic and far more useful: a massive reference library spanning an almost incomprehensible range of subjects that you can query in plain language and get a best-guess answer back in seconds. That's it. That's the mechanism. It reads what you wrote, it pattern-matches against everything it was trained on, and it returns the most probable useful response. It does not think. It does not have opinions about you. It is a tool.

That distinction matters, because once you see it clearly, the debate about whether using these tools is legitimate collapses into something much simpler: do you use reference materials? Do you use documentation? Do you use Google? Then you already understand the value proposition. The difference is the interface, the speed, and the fact that it will tell you when you missed a curly brace at 3am without making you feel bad about it.

---

## 3am

Here is a scenario every engineer and administrator knows. You were asleep thirty minutes ago. Something is broken, or you're building something and you've hit a wall, and the wall is some random bit of syntax you knew last Tuesday and cannot locate in your brain right now. Your options are: wake up a colleague who probably just got home from the bar. Call your boss. Read the man page for the hundredth time with eyes that don't fully work yet. Or query an LLM and have your answer in the time it took you to finish that sentence.

The answer is obvious. It has always been obvious. The only thing that ever made it complicated was the question of whether using the tool meant something unflattering about you as an engineer. It doesn't. Nobody questions whether you should read the man page. Nobody suggests that using documentation is cheating. An LLM is faster documentation that you can have a conversation with, and at 3am that is exactly what you need.

There's something else worth naming here. LLMs don't judge you when you call them out on a wrong answer. They don't get defensive when you find an error in their code. They don't scoff when you can't remember how to make a directory. You don't have to manage anyone's ego or apologize for not knowing something. You ask, you get an answer, you verify it, you move on. That frictionlessness is underrated and it matters more than people admit.

Yes, they hallucinate. Yes, they make mistakes. That's why you still need to know enough to catch when the answer is wrong. The skill requirement doesn't disappear — it shifts. You need to know what a good answer looks like. You need to know when to push back. That's a different skill than memorizing syntax, and it's a more durable one.

---

## The Conference

I started taking LLMs seriously at a cybersecurity conference. A speaker made a point I haven't stopped thinking about: the adversary — of any flavor, at any scale — is already using these tools to outpace defenders. They're using them to write malware faster, to find vulnerabilities faster, to automate the parts of an attack that used to require time and expertise. If you're not playing at that same level, you're not really in the game. You may as well have never shown up.

That reframed everything for me. This isn't a question of personal preference or philosophical comfort with new technology. It's a question of whether you're keeping pace with the threat landscape. Engineers and administrators who let ego or skepticism keep them from adopting useful tools are making a unilateral decision to fall behind, and the people on the other side of that equation are not making the same decision.

---

## The Legitimate Concerns

None of this means adoption is unconditional. There are real questions that deserve real answers before you integrate these tools into professional workflows.

**Security:** what data are you feeding into a query? In a regulated environment, sensitive information has no business going into an external LLM. Know what you're submitting and know where it goes.

**Compliance:** some industries and organizations have explicit policies governing LLM tool usage. Know your policy. If your organization doesn't have one yet, that's a conversation worth starting.

**Access and ownership:** code and configuration generated with LLM assistance still needs to be understood, owned, and maintainable by the person deploying it. If you can't explain what it does or fix it when it breaks, you don't have a solution — you have a liability.

These aren't arguments against using the tools. They're arguments for using them deliberately.

---

## What This Is Not

This is not about vibe-coding. It's not about generating entire applications you don't understand and shipping them because the demo looked good. It's not about replacing expertise with autocomplete. The people doing that are building technical debt at a pace that will eventually catch up with them badly.

What it is about is automating the low-value mechanical work so your actual expertise can go toward the decisions that matter. Syntax errors at 3am are not where your value as an engineer lives. Neither is remembering the exact flag order for a command you use twice a year, or building a throwaway script wrapper when you need to get back to the actual problem. Let the tool handle that. Save your attention for the architecture, the reasoning, the judgment calls that a reference library — however sophisticated — cannot make for you.

Nobody wants to keep working on easy-to-submit, hard-to-resolve tickets at 3pm on a Friday. The tools that get you out of that loop faster are not a threat to your professionalism. They are an argument for it.

---

## The Bottom Line

Use the tools. Understand what they are and what they aren't. Know their failure modes. Keep your hands on the wheel. Document what you built and why you built it, because when something breaks at 2am next month the LLM isn't on call — you are.

The engineers who will do the most with these tools are the ones who are honest about using them and rigorous about what that means for their own expertise. The ones who pretend the tools don't exist, or that using them is somehow beneath them, are making a choice. It's not a choice I'd recommend.

The world is evolving. The only expensive thing about that is your ego.

---

*A note on how this was written: Did I sit down and write this article from scratch? No. I had a conversation. I dumped my stream of consciousness, my tangents, my opinions, and my 3am war stories into Claude and it predicted what I wanted you to see. The thoughts, the experience, and the opinions are mine. The polish is the tool's.*

*That's the whole argument, demonstrated. If that bothers you, you may have missed the point.*
