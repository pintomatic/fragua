---
name: fragua
description: "FRAGUA — four-phase build protocol. Orchestrates the full lifecycle: Plan → CRITICON (design) → MANAYER (build) → CRITICON (implementation). Invoke with /fragua for any production-critical feature that touches 3+ files. Requires criticon and manayer skills."
---

# FRAGUA

> *Fragua: Spanish for forge. Raw material goes in, hardened steel comes out.*

Orchestrates the full four-phase build lifecycle. Calls CRITICON and MANAYER in sequence.

```
Plan → CRITICON (plan) → MANAYER (build) → CRITICON (implementation)
```

---

## Usage

```
/fragua                                        # interactive — asks for project + what to build
/fragua code/my-project "add auth endpoint"    # project path + description
/fragua code/my-project docs/plan-auth.md      # project path + existing plan file
```

---

## When to Use

| Build type | Use |
|------------|-----|
| Simple fix, 1-2 files, familiar pattern | Skip — just build |
| Multi-file feature, 3-8 files | MANAYER only (`/manayer`) |
| New infrastructure, auth, payments, concurrency | **FRAGUA** |
| Replacing a system component | **FRAGUA** |
| Production-critical path | **FRAGUA** |

If you'd throw it away tomorrow, don't FRAGUA it. The overhead (~45-60 min of structured review) pays off when correctness matters more than speed.

---

## Phase 0: Establish Context

Before anything: read the project. CLAUDE.md, relevant source files, existing docs.

If a plan file is given: read it. If a description is given: draft the plan now.

Plan template:
```markdown
# Implementation Plan: [Feature Name]

## Goal
[What and why — 1-2 sentences]

## File Map
| File | Action | Changes |
|------|--------|---------|
| `path/to/file.ts` | MODIFY | Add X endpoint, update Y function |

## Detailed Changes
[Per-file specifics]

## Data Flow
[Input → transform → output]

## Constraints
- [ ] No new dependencies unless necessary
- [ ] Auth preserved
- [ ] Files under 500 lines

## Known Pitfalls
[Edge cases, gotchas, things that burned you before]

## Verify
[Build command, test command, smoke test]
```

Save to `docs/plan-[feature].md`.

---

## Phase 1: CRITICON on the Plan

Run `/criticon docs/plan-[feature].md`

This stress-tests the design before a line of code is written. Plan critiques are free. Implementation critiques cost hours.

Target: 2-3 rounds, SHIPPABLE. Do not hand a NEEDS REVISION plan to MANAYER.

**What CRITICON catches here:** wrong API assumptions, missing edge cases, scope wider than documented, security gaps obvious in design but invisible during build.

---

## Phase 2: MANAYER

Run `/manayer [project-path] docs/plan-[feature].md`

Use the CRITICON-approved plan (not the original draft). The coder agent gets the validated spec, not the conversation history.

Wait for MANAYER to complete fully before starting Phase 3.

---

## Phase 3: CRITICON on the Implementation

After MANAYER fixes are applied and the build is clean:

Run `/criticon [path/to/core-changed-file]`

Add context to the Opus prompt: what the file does, runtime, concurrency expectations, what callers pass in.

**What CRITICON catches here that MANAYER reviewer misses:** race conditions, resource leaks on error paths, env-specific issues (path casing, encoding), control flow correct on happy path but wrong under exhaustion.

Target: SHIPPABLE with no Critical or Important findings. Then verify + deploy.

---

## Audit Trail

```
criticon/<plan-name>-YYYY-MM-DD/     ← Phase 1
manayer/<build-name>-YYYY-MM-DD/    ← Phase 2
criticon/<impl-name>-YYYY-MM-DD/    ← Phase 3
```

---

## Phase Sequencing Rules

- Never hand a NEEDS REVISION plan to MANAYER — fix CRITICON criticals first
- Never run Phase 3 until MANAYER fixes are applied and build is green
- If Phase 3 CRITICON finds Critical issues, apply fixes before shipping — do not skip
- If the build scope changes significantly during MANAYER (coder found something unexpected), loop back to Phase 1 on the revised plan

---

## Example

```
You: /fragua code/kernal-cloud "add multi-env support"

FRAGUA:
  Phase 0 — Reads project, drafts plan → docs/plan-multi-env.md
  Phase 1 — CRITICON on plan: NEEDS REVISION (2 Critical: FK ordering, auth bypass)
           → fixes applied → Round 2: SHIPPABLE
  Phase 2 — MANAYER: coder builds, reviewer audits, manager fixes HIGH findings
  Phase 3 — CRITICON on worker.ts + kernal-agent.ts
           → NEEDS REVISION (1 Critical: transactionSync race condition)
           → fix applied → Round 2: SHIPPABLE
  
  Build complete. Audit trail:
    criticon/multi-env-plan-2026-04-27/
    manayer/multi-env-2026-04-27/
    criticon/multi-env-impl-2026-04-27/
```

---

## What FRAGUA Is Not

FRAGUA is not a replacement for tests, monitoring, or human judgment. It's a structured process that makes AI-assisted development more predictable — not a guarantee. The manager (you) still applies judgment at every phase.

---

*FRAGUA v1.0 — April 2026*
*Sub-skills: [CRITICON](../criticon/SKILL.md) · [MANAYER](../manayer/SKILL.md)*
*Named after the Spanish word for forge. Part of the same family as CRITICON (críticón) and MANAYER (manager).*
