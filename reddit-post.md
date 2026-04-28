# Reddit Post Draft — r/ClaudeAI

**Title:** I cut my Claude Code token usage from 73% to 30% of monthly budget while shipping ~3x more. Here's the workflow.

---

This is a practical workflow, not theory. Week-over-week data from the same developer, same Claude Code setup.

**Before:** 73% of monthly token budget consumed. Lots of exploratory back-and-forth. Mid-implementation rework when the design turned out to be wrong. Debugging sessions that could have been caught earlier.

**After:** 30% consumed. Roughly 3x the shipped features. Near-zero rework.

The difference: a four-phase protocol I'm calling **FRAGUA** (Spanish for forge). It wraps two skills — CRITICON and MANAYER.

---

## The Protocol

```
Plan (MD) → CRITICON → MANAYER → CRITICON
```

**Phase 0: Write a plan.** Markdown scope doc before touching any code. File map, data flow, constraints, known pitfalls. ~15-20 minutes.

**Phase 1: CRITICON on the plan.** Spawn Claude Opus as a dedicated critic. Opus reviews the plan, returns a verdict (SHIPPABLE / NEEDS REVISION) with findings classified 🔴 Critical / 🟡 Important / 🟢 Minor. You fix the Criticals, revise, and send the updated document back to the *same* Opus instance — it remembers what it found and zooms in on what's left. 2-3 rounds until SHIPPABLE.

**Phase 2: MANAYER.** Three isolated roles: coder agent builds from the approved plan (clean context, no history), reviewer agent audits the output, you apply the CRITICAL/HIGH fixes. Each agent starts fresh — no context explosion.

**Phase 3: CRITICON on the implementation.** Opus reviews the actual changed code, same iterative multi-round format. This catches what the reviewer missed: race conditions, resource leaks on error paths, env-specific edge cases, control flow that's correct on happy path but wrong under exhaustion.

---

## Why It Works

**Standard workflow token shape:** one long context carrying everything — the exploration, the design debates, the wrong turns, the fixes for the fixes. Token cost compounds as the context grows.

**FRAGUA token shape:** four small isolated contexts, each with a precise brief.

Three mechanisms:

1. **CRITICON kills wasted builds.** Every problem Opus finds in the plan is a whole implementation that doesn't get written. Catching a design flaw in a markdown doc takes 5 minutes. Catching it mid-implementation costs 2 hours.

2. **MANAYER isolates context.** Coder gets the plan, not your conversation history. Reviewer gets the diff and a checklist. You hold the judgment. Three small windows instead of one giant compounding one.

3. **Sonnet with a precise brief ≈ Opus flying blind.** The plan is the intelligence. Sonnet executes it faithfully. You don't need Opus-level reasoning for execution tasks when the brief is tight.

---

## How It Relates to Ralph Loop / GSD

These aren't the same thing and they're complementary.

**Ralph Loop** is an autonomous retry loop — Claude runs a task, checks tests, re-runs until it passes. CRITICON+MANAYER is what you do *before* Ralph runs: stress-test the design, isolate the build context, stress-test the result.

**GSD** addresses context rot — fresh window per task, atomic commits. MANAYER does the same thing for context isolation but adds the three-role separation and a critique layer.

FRAGUA is the bookend: critique the design, build it cleanly, critique the implementation.

---

## GitHub

Full protocol + installable skills: **github.com/pintomatic/fragua**

Drop `CRITICON.md` and `MANAYER.md` into your Claude Code skills folder (`~/.claude/skills/`) and invoke with `/criticon` and `/manayer`.

---

Happy to answer questions. The skills are MIT licensed, use them however.
