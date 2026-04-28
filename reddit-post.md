# Reddit Post — r/ClaudeAI

**Title:** I trust Sonnet as my daily driver now — better code, one-third the tokens. Here's how.

---

For months I defaulted to Opus for anything complex. Sonnet felt like a gamble — sometimes great, sometimes it would confidently build the wrong thing and I'd spend an hour unwinding it. So I'd reach for Opus, burn tokens, and still end up debugging things that should have been caught earlier.

Last week: 30% of my monthly budget consumed by Friday, roughly 3x the shipped work compared to the previous week at 73%. Code was cleaner. Barely any rework. Sonnet the entire time.

I can't give you a controlled study — this is one developer, one week, real production work (Cloudflare Workers + TypeScript). But the specific thing that changed was the structure around the model, not the model itself.

---

## What changed: FRAGUA

I built a four-phase protocol I'm calling FRAGUA (Spanish for forge). It's two skills — CRITICON and MANAYER — run in a specific order:

```
Plan → CRITICON → MANAYER → CRITICON
```

**Step 1: Write a plan in markdown.** File map, what changes, data flow, known pitfalls. 15 minutes. This sounds obvious but most people (including me, before) skip it or write it too vague to be useful.

**Step 2: CRITICON on the plan.** Spawn Claude Opus as a named subagent with one job: find what's wrong with the plan. It returns a verdict — SHIPPABLE or NEEDS REVISION — with findings sorted 🔴 Critical / 🟡 Important / 🟢 Minor. You fix the Criticals, send the revised plan back to the *same named subagent instance* (it retains context between calls and zooms into what's left rather than starting cold). 2-3 rounds until nothing critical remains.

**Step 3: MANAYER.** Three isolated roles. Coder agent builds from the approved plan — clean context window, no conversation history, just the spec. Reviewer agent audits the output. You apply the CRITICAL/HIGH fixes yourself. Each agent starts fresh. No compounding context.

**Step 4: CRITICON on the implementation.** Same iterative Opus loop, now on the actual changed code. This catches what a single-pass reviewer misses: race conditions, resource leaks on error paths, edge cases that only surface under load.

---

## When NOT to use FRAGUA

Single-file edits, config changes, quick fixes, exploratory prototyping, research spikes — skip it entirely and just build. The overhead (~45-60 min of structured review) only pays off when correctness matters more than speed and the build touches 3+ files. If you'd throw it away tomorrow, don't FRAGUA it.

---

## What actually forced this

I was running Opus 4.6 as my default. Opus 4.7 dropped and I hit 70% of my monthly budget in a single day. That forced a question I should have asked earlier: *is the problem the model, or is the problem how I'm using it?*

The answer was the process. Every new model generation will be more capable and more expensive. If your workflow requires the best available model just to function, you're on a treadmill. The answer isn't "wait for prices to fall." It's "stop needing the most expensive model for every task."

The uncomfortable part: defaulting to Opus was a symptom of bad process. I wasn't trusting Sonnet because my context was a mess — exploration, design, implementation, and debugging all tangled in one long thread. That's a genuinely hard job. Of course Opus handled it better. Of course Sonnet stumbled.

The fix was spec before build, separation of concerns, design review, code review. Software engineering principles from the 1970s, applied to AI assistants.

---

## The cost of CRITICON itself

The honest question: "Aren't you just moving Opus from the build phase to the review phase?"

Partially, yes. CRITICON runs 2-3 Opus rounds before the build and 2-3 after. That's roughly 30-50k Opus tokens per phase. It's not free.

The math works because of what it eliminates. When CRITICON catches a design flaw in the plan, that's a whole multi-agent build that doesn't happen. When it catches a runtime bug in the implementation, that's a debugging spiral that doesn't start. The most expensive token in AI development is the one you spend re-explaining context to fix something that should have been caught earlier.

On the week I measured: the two main CRITICON sessions cost roughly the equivalent of one hour of unfocused Opus usage. They prevented approximately three hours of rework I can specifically identify — one FK ordering bug that would have been a 5-round debugging session, one API assumption that would have required rebuilding a module.

---

## Why Sonnet works inside FRAGUA

By the time Sonnet (as the coder) sees the task, Opus has already validated the design across multiple rounds. The plan is airtight. Sonnet doesn't need to reason about architecture — it executes a precise spec. That's what it's actually good at.

Sonnet executing a CRITICON-approved plan consistently outperforms Opus winging it from a vague prompt. And costs a fraction.

---

## Prior art — what I looked at first

**Ralph Loop** — autonomous retry loop that runs until tests pass. FRAGUA is what you run *before* Ralph so it has something solid to iterate on. These pair naturally.

**GSD (Get Shit Done)** — spec-driven, fresh context window per task, atomic commits. Addresses the same context rot problem MANAYER does. MANAYER adds the critique layer; GSD has better commit discipline. I'd combine them.

**hamelsmu's claude-review-loop** — single-pass cross-model review. Good for quick audits. CRITICON is multi-round and iterates the same instance; they're different tools for different depths.

What I haven't found: anyone combining design critique + isolated execution + implementation critique in one workflow, or running the same Opus instance across multiple rounds so it builds on what it found rather than starting cold. Happy to be wrong about this.

**What else is out there that plugs into this?** Especially curious if anyone has combined FRAGUA + Ralph Loop in practice, or has a better approach to the implementation review phase. Throw it in the comments.

---

## Install

GitHub: **github.com/pintomatic/fragua**

Drop `CRITICON.md` and `MANAYER.md` into `~/.claude/skills/` (Linux/Mac) or `C:\Users\<you>\.claude\skills\` (Windows) and invoke with `/criticon` and `/manayer` in Claude Code. MIT licensed, full prompt templates included.

---

Happy to answer questions. The skills are annotated — no assembly required, just drop and go.
