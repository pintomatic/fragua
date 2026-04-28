# FRAGUA

> *Fragua: Spanish for forge. Raw material goes in, hardened steel comes out.*

A four-phase AI development protocol for Claude Code that consistently produces higher-quality code in fewer tokens than ad-hoc prompting.

**Measured results (week-over-week, same developer, same Claude Code setup):**
- Token usage: 73% of monthly budget → 30% for roughly 3x the shipped work
- Code quality: CRITICON caught bugs before deployment that would have become multi-hour debugging sessions
- Rework rate: near zero — plans survive contact with implementation

---

## The Protocol

```
Plan (MD) → CRITICON → MANAYER → CRITICON
```

| Phase | What happens | Time |
|-------|-------------|------|
| **Plan** | Write a markdown scope doc | 10-20 min |
| **CRITICON** | Opus reviews the plan across multiple rounds, zooming in each time | 15-30 min |
| **MANAYER** | Coder agent builds. Reviewer agent audits. You apply the critical fixes. | async |
| **CRITICON** | Opus reviews the implementation — catches what the reviewer missed | 15-30 min |

---

## The Three Skills

### [CRITICON](CRITICON.md)
Multi-round iterative critique loop using Claude Opus. Spawns one Opus agent, runs it across multiple rounds, resumes the **same instance** so it builds on what it already found. Each round zooms in: Round 1 catches structural problems, Round 2 catches gaps, Round 3 catches edge cases. Produces a full audit trail.

Works on: plans, architecture docs, API designs, technical specs, and production code.

### [MANAYER](MANAYER.md)
Three-role build workflow: **you** write the scope (manager), a **coder agent** implements it in a clean context, a **reviewer agent** audits the output, **you** apply the critical fixes. Each role has an isolated context window — no quadratic token explosion.

Named after "manager" pronounced with a Latin American accent. CRITICON is a críticón — someone who criticizes a lot.

### [FRAGUA](FRAGUA.md)
The protocol that runs both. Full spec in [FRAGUA.md](FRAGUA.md).

---

## Why This Works

**Standard workflow:** One long context doing everything — exploring, designing, building, debugging, backtracking. Token cost compounds as context grows. Rework is expensive because you've lost the thread.

**FRAGUA:** The plan is stress-tested before a line of code is written. Plan critiques are free to fix (just text). The build runs in isolated agent contexts. The implementation is stress-tested before it ships. Rework is nearly eliminated.

Three mechanisms driving the efficiency:

1. **CRITICON eliminates wasted builds.** Every problem Opus finds in the plan is an implementation that doesn't get written. The FK ordering bug we caught in CRITICON would have been a 5-round debugging spiral in production.

2. **MANAYER isolates context.** Coder gets a clean brief, not 60k tokens of conversation history. Reviewer gets the diff and a checklist. Manager carries the architecture, not implementation details. Three small windows instead of one compounding one.

3. **Sonnet with a precise brief ≈ Opus flying blind.** The brief is the intelligence. Sonnet executes it faithfully at a fraction of the cost.

---

## How It Relates to Other Claude Code Workflows

**Ralph Loop** — autonomous retry loop that runs until tests pass. CRITICON+MANAYER is what you do *before* Ralph runs. They're complementary.

**GSD (Get Shit Done)** — spec-driven context-rot-prevention workflow. Similar motivation to MANAYER (fresh context per task), less critique structure. Can be combined.

FRAGUA is the bookend: critique the design, build it, critique the result.

---

## Installation

FRAGUA works with [Claude Code](https://claude.ai/code) (the Anthropic CLI). The skills are markdown files you reference in your conversation or add to your `CLAUDE.md`.

**Option 1: Drop the skill files into your project**
```bash
git clone https://github.com/pintomatic/fragua
# Reference CRITICON.md and MANAYER.md from your CLAUDE.md or paste into conversation
```

**Option 2: Add to your global Claude Code skills**
```bash
mkdir -p ~/.claude/skills/criticon ~/.claude/skills/manayer
cp fragua/CRITICON.md ~/.claude/skills/criticon/SKILL.md
cp fragua/MANAYER.md ~/.claude/skills/manayer/SKILL.md
```

Then invoke with `/criticon` and `/manayer` in Claude Code.

---

## Quick Start

1. Write a plan doc for what you want to build (`docs/plan-my-feature.md`)
2. Run `/criticon docs/plan-my-feature.md` — let Opus find the gaps
3. Fix what CRITICON flagged
4. Run `/manayer` — coder builds, reviewer audits, you fix CRITICAL/HIGH findings
5. Run `/criticon` on the core changed file — Opus zooms in on the implementation
6. Ship

Total overhead vs ad-hoc: ~45-60 minutes of structured review. Savings: hours of debugging, rework, and context recovery.

---

## The Name

All three tools follow the same naming convention: English words as heard through a Latin American accent.

- **CRITICON** — críticón (someone who criticizes constantly)
- **MANAYER** — "manager"
- **FRAGUA** — forge in Spanish. The forge is where raw material becomes hardened steel.

---

*Built and validated by [@pintomatic](https://github.com/pintomatic) on production infrastructure (Cloudflare Workers, TypeScript, Python). April 2026.*
