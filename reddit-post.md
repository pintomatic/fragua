# Reddit Post — r/ClaudeAI

**Title:** I can now trust Sonnet as my daily driver with better code quality and one third the tokens. Here's how.

---

For months I defaulted to Opus for anything complex. Sonnet felt like a gamble — sometimes great, sometimes it would confidently build the wrong thing and I'd spend an hour unwinding it. So I'd reach for Opus, burn tokens, and still end up debugging things that should have been caught earlier.

Last week something flipped. I was at 30% of my monthly budget on a Friday and had shipped roughly three times what I'd done the previous week at 73%. The code was cleaner. Zero rework. I'd been running Sonnet the entire time.

The difference wasn't the model. It was the structure I put around it.

---

## What changed: FRAGUA

I built a four-phase protocol I'm calling FRAGUA (Spanish for forge). It's two skills — CRITICON and MANAYER — run in a specific order:

```
Plan → CRITICON → MANAYER → CRITICON
```

**Step 1: Write a plan in markdown.** File map, what changes, data flow, known pitfalls. 15 minutes. This sounds obvious but most people (including me, before) skip it or write it too vague.

**Step 2: CRITICON on the plan.** Spawn Claude Opus as a dedicated critic with one job: find what's wrong with the plan. It returns a verdict — SHIPPABLE or NEEDS REVISION — with findings sorted 🔴 Critical / 🟡 Important / 🟢 Minor. You fix the Criticals, send the revised plan back to the *same* Opus instance (it remembers what it found and zooms in on what's left). 2-3 rounds until nothing critical remains.

**Step 3: MANAYER.** Three isolated roles. Coder agent builds from the approved plan — clean context, no conversation history, just the spec. Reviewer agent audits the output. You apply the CRITICAL/HIGH fixes yourself. Each agent starts fresh. No context explosion.

**Step 4: CRITICON on the implementation.** Same iterative Opus loop, but now on the actual changed code. Catches what the reviewer missed: race conditions, resource leaks on error paths, edge cases that only show up under load.

---

## What actually forced this

I was happily running Opus 4.6 as my default. Then Opus 4.7 dropped and I hit 70% of my monthly budget in a single day. One day.

That's when I realised I had a structural problem, not a model problem. Every new model generation is going to be more capable and more expensive. If your workflow requires the most powerful model to function, you're on a treadmill that only gets more expensive. The answer can't be "wait for prices to drop." The answer has to be "fix the workflow."

The uncomfortable truth is that what I was doing — defaulting to Opus because I didn't trust Sonnet — was actually a symptom of bad process. I wasn't trusting Sonnet because my context was a mess. The model was being asked to simultaneously explore, design, implement, and debug in a single thread. That's a hard job. Of course Opus did it better. Of course Sonnet stumbled.

The fix turned out to be software engineering principles that have existed for 50 years: spec before build, separation of concerns, design review, code review. We just hadn't applied them consistently to AI-assisted development.

---

## Why Sonnet works now

The old problem wasn't Sonnet's capability. It was that I was asking Sonnet to simultaneously figure out *what* to build and *how* to build it, in a context window already bloated with everything from the last two hours of conversation.

FRAGUA separates those things. By the time Sonnet (as the coder) sees the task, CRITICON has already stress-tested the design across multiple Opus rounds. The plan is airtight. Sonnet doesn't need to reason about architecture — it just executes a precise spec. That's exactly what it's good at.

The reviewer catches correctness issues. CRITICON catches runtime issues. You apply judgment. Each role does one thing in a clean context.

The result: Sonnet executing a CRITICON-approved plan consistently produces better output than Opus winging it from a vague prompt. And costs a fraction.

---

## The token math

The real saving isn't CRITICON or MANAYER individually. It's eliminating the most expensive thing in AI development: mid-implementation discovery that the design was wrong.

When you realize halfway through a build that you had the wrong mental model of an API, the cost isn't just the tokens you spent — it's the tokens to undo it, explain what happened, course-correct, and rebuild. That spiral is where budgets go.

CRITICON runs before the build. Plan critiques are free to fix. A Markdown document is weightless. The same discovery in implementation costs hours.

This also future-proofs the workflow. FRAGUA gets *more* efficient as models improve, not less — because better Opus means sharper critiques, and better Sonnet means cleaner execution. The process scales. The "throw the best model at everything" approach doesn't.

---

## How it relates to Ralph Loop / GSD

These aren't the same and they're complementary.

**Ralph Loop** — autonomous retry until tests pass. FRAGUA is what you run before Ralph so Ralph has something solid to iterate on.

**GSD** — spec-driven context-rot prevention, fresh window per task. MANAYER does the same context isolation plus the critique layer.

FRAGUA bookends the build: critique the design, build it in isolated contexts, critique the implementation.

---

## Install

GitHub: **github.com/pintomatic/fragua**

Drop `CRITICON.md` and `MANAYER.md` into `~/.claude/skills/` and invoke with `/criticon` and `/manayer` in Claude Code. MIT licensed.

---

Happy to answer questions about how any phase works in practice. The skills include full prompt templates — no assembly required.
