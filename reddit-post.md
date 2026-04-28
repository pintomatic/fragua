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

## Why Sonnet suddenly works

The old problem wasn't Sonnet's capability. It was that I was asking Sonnet to simultaneously figure out *what* to build and *how* to build it, in a context window that was already bloated with everything from the last two hours of conversation.

FRAGUA separates those things. By the time Sonnet (as the coder) sees the task, CRITICON has already validated the design across multiple Opus rounds. The plan is airtight. Sonnet doesn't need to reason about architecture — it just executes a precise spec. That's exactly what it's good at.

The reviewer catches correctness issues. CRITICON catches runtime issues. You apply judgment. Each role does one thing in a clean context.

The result: Sonnet executing a CRITICON-approved plan produces better output than Opus winging it from a vague prompt. And it costs a fraction.

---

## The token math

The real saving isn't CRITICON or MANAYER individually. It's that you eliminate the most expensive thing in AI development: mid-implementation discovery that the design was wrong.

When you realize halfway through a 3-agent build that you had the wrong mental model of an API, the cost isn't just the tokens you spent — it's the tokens to undo it, explain what happened, course-correct, and rebuild. That spiral is where budgets go.

CRITICON runs before the build. Plan critiques are free to fix. A Markdown document is weightless. The same discovery in implementation costs hours.

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
