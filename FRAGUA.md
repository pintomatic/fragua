# FRAGUA Protocol

> *The full four-phase lifecycle. Use for any build where correctness matters more than speed.*

```
Plan → CRITICON → MANAYER → CRITICON
```

---

## When to Use FRAGUA vs Simpler Workflows

| Build type | Use |
|------------|-----|
| Simple feature, 1-3 files, familiar pattern | No FRAGUA — just build |
| Multi-file feature, 3-8 files | MANAYER only |
| New infrastructure (auth, payments, concurrency, LLM wrappers) | **Full FRAGUA** |
| Replacing a system component | **Full FRAGUA** |
| Production-critical paths (data pipelines, trading logic) | **Full FRAGUA** |
| Architecture or design document | CRITICON only |

---

## Phase 0: Write the Plan

Write a markdown scope document before touching any code. This is the most important phase — agents are only as good as their brief.

```markdown
# Implementation Plan: [Feature Name]

## Goal
[What and why — 1-2 sentences]

## File Map
| File | Action | Changes |
|------|--------|---------|
| `path/to/file.ts` | MODIFY | Add X endpoint, update Y function |
| `path/to/new.ts` | CREATE | New module for Z |

## Detailed Changes
[Per-file: what exactly changes and where]

## Data Flow
[Where data comes from → transforms → where it goes]

## Constraints
- [ ] No new dependencies unless necessary
- [ ] Auth must be preserved
- [ ] Keep files under 500 lines

## Known Pitfalls
[Edge cases, framework gotchas, things that burned you before]

## Verify
[Build command, test command, smoke test]
```

Save as `docs/plan-[feature].md`.

---

## Phase 1: CRITICON on the Plan

Run CRITICON on the scope doc before handing it to the coder. Opus will find design gaps that are cheap to fix now and expensive to fix after three agents have touched the code.

See [CRITICON.md](CRITICON.md) for the full protocol.

**What CRITICON typically catches on plans:**
- Wrong assumptions about APIs or external systems
- Missing edge cases that would require architectural changes later
- Scope wider than documented (e.g. a shared module you didn't know about)
- Security or data integrity gaps that are obvious in review, invisible when building

**Target:** 2-3 rounds, SHIPPABLE verdict. If Opus finds nothing critical in Round 1, you're likely ready.

---

## Phase 2: MANAYER

Standard three-phase build: coder builds in a clean context, reviewer audits, you apply the critical fixes.

See [MANAYER.md](MANAYER.md) for the full protocol.

**Key discipline:**
- Give the coder your CRITICON-approved plan, not the original
- Do not spawn coder and reviewer simultaneously
- You apply CRITICAL and HIGH fixes — not a third agent

---

## Phase 3: CRITICON on the Implementation

After MANAYER fixes are applied and the build is clean, run CRITICON on the core changed files. This is a different kind of review than MANAYER's reviewer — Opus zooms in across rounds rather than scanning everything once.

See [CRITICON.md](CRITICON.md) — use the "code critique" variant.

**What CRITICON catches on implementation that MANAYER reviewer misses:**
- Race conditions and double-settle bugs
- Resource leaks on error paths (semaphores, connections, file handles)
- Environment-specific issues (path case sensitivity, encoding boundaries)
- Control flow that's correct on the happy path but wrong under exhaustion

**How to run it:**
```
/criticon path/to/core-changed-file.ts
```
Add context in the prompt: what the file does, what runtime it runs in, concurrency expectations.

**Exit:** SHIPPABLE with no Critical or Important findings. Then verify + deploy.

---

## Audit Trail

```
criticon/<plan-name>-YYYY-MM-DD/     ← Phase 1: CRITICON on plan
  v0-initial.md
  r1-critique.md
  v1-revised.md
  final.md
  summary.md

manayer/<build-name>-YYYY-MM-DD/    ← Phase 2: MANAYER build
  scope.md
  coder-summary.md
  review.md
  fixes.md
  summary.md

criticon/<impl-name>-YYYY-MM-DD/    ← Phase 3: CRITICON on implementation
  v0-initial.md
  r1-critique.md
  final.md
  summary.md
```

The audit trail is the permanent record of how the build evolved. When a bug appears in production, you can trace exactly what decisions were made and what was reviewed.

---

## Why CRITICON Bookends MANAYER

**Before build (Phase 1):** Plan critiques are free. Fixing a design gap in markdown takes 5 minutes. Fixing the same gap after three agents have touched the code takes 2 hours.

**After build (Phase 3):** The MANAYER reviewer is fast and broad — it scans everything once, classifies findings, and moves on. CRITICON on implementation is slow and deep — Opus zooms in iteratively on the same file, rounds 1-2-3, building on what it found. They catch fundamentally different things.

The MANAYER reviewer catches correctness at the feature level. CRITICON catches correctness at the runtime level.

---

## Token Efficiency

Running FRAGUA costs roughly:
- Phase 0: your time (~20 min), minimal tokens
- Phase 1: 2-3 CRITICON rounds (~30-50k tokens total)
- Phase 2: coder + reviewer in isolated contexts (~40-80k tokens)
- Phase 3: 2-3 CRITICON rounds on implementation (~30-50k tokens)

Compared to a single-context exploratory build that discovers its design problems mid-implementation: the upfront investment is roughly equal to the first major debugging spiral you avoid.

Measured result: 73% → 30% of monthly token budget while shipping approximately 3x the work in the same calendar week.

---

*FRAGUA v1.0 — April 2026*
