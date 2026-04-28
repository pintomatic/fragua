# CRITICON

> *"A plan reviewed by a different mind is a better plan."*

Multi-round iterative critique loop using Claude Opus. Spawns one Opus agent, runs it across multiple rounds, resumes the **same instance** each time so it builds on what it already found. Each round zooms in: structural problems first, gaps second, edge cases third.

Proven on: architecture plans, API designs, technical specs, marketing briefs, and production code (especially auth, concurrency, LLM wrappers, payment flows).

---

## When to Use

**On plans (before building):**
- Any scope document that governs significant work
- Before running MANAYER — critique the plan first
- When a design feels right but you want it stress-tested
- Architecture docs, API contracts, technical specs

**On implementation code (after building):**
- New infrastructure modules: auth, payments, concurrency, subprocess management
- Production-critical paths: data pipelines, trading logic, webhook handlers
- Replacing a system component (SDK migration, provider swap)
- Any code where the MANAYER reviewer found nothing but the code feels complex

**CRITICON on code vs MANAYER reviewer:** The reviewer scans everything once and classifies findings. CRITICON zooms in iteratively on the same file — it catches race conditions, resource leaks on error paths, env-specific issues, and control flow that's correct on the happy path but wrong on exhaustion. Different tools for different depths.

**Don't use for:** single-file edits, config changes, quick fixes, decisions you can reverse cheaply.

---

## CRITICON on Plans vs Code

| Target | What Opus looks for | Typical rounds |
|--------|---------------------|----------------|
| Plan doc | Design gaps, wrong assumptions, missing edge cases | 2-3 |
| Implementation code | Runtime bugs, race conditions, resource leaks, security | 3-4 |

---

## Audit Trail

Every CRITICON session creates a dedicated folder:

```
criticon/<plan-name>-YYYY-MM-DD/
  v0-initial.md      ← original document before critique
  r1-critique.md     ← Round 1 Opus verdict
  v1-revised.md      ← document after Round 1 fixes
  r2-critique.md     ← Round 2 Opus verdict
  v2-revised.md      ← document after Round 2 fixes
  final.md           ← approved version
  summary.md         ← round count, verdicts, key changes
```

---

## The Loop

### Step 0: Set up + get the document

1. Create the session folder: `criticon/<slug>-YYYY-MM-DD/`
2. Get the document (read the file, or draft it if starting from a topic)
3. Save it as `v0-initial.md`

### Step 1: Spawn Opus (Round 1)

Name the agent so it can be resumed. Opus reads the full document cold.

```
Agent(
  subagent_type: "reviewer",
  model: "opus",
  name: "opus-critic",
  prompt: """
You are a critical design reviewer running multiple rounds. Each round you zoom in:
- Round 1: find structural and critical issues
- Round 2: find important but non-blocking gaps (assume Round 1 issues are fixed)
- Round 3+: polish, edge cases, wording

Read this document carefully:

---
[PASTE FULL DOCUMENT]
---

This is Round 1. Focus on structural problems and critical issues.

Return your critique in this exact format:

## VERDICT: [SHIPPABLE | NEEDS REVISION]

## 🔴 Critical (blocks shipping)
[Issues that would cause this plan to fail, produce the wrong thing, or miss essential requirements. If none: write "None."]

## 🟡 Important (should fix before shipping)
[Gaps, contradictions, missing edge cases, assumptions that need validation. If none: write "None."]

## 🟢 Minor (improve if time allows)
[Wording, structure, nice-to-haves. If none: write "None."]

## Reasoning
[2-3 sentences on why you gave this verdict]

Be ruthless. A false SHIPPABLE that ships a broken plan is worse than a harsh critique that catches a problem early.
"""
)
```

### Step 2: Review + revise

- **SHIPPABLE, no Critical/Important:** loop exits. Save `final.md`.
- **NEEDS REVISION:** fix Critical issues first, then Important. Rewrite the document.

Save the critique as `r1-critique.md`. Save the revised document as `v1-revised.md`.

### Step 3: Resume the same Opus agent (Rounds 2+)

**Do NOT spawn a new agent.** Use `SendMessage` — the same instance remembers what it found, can confirm fixes, and zooms into finer detail.

```
SendMessage(
  to: "opus-critic",
  message: """
I've revised the document based on your Round [N] critique. Here is the updated version:

---
[PASTE UPDATED DOCUMENT]
---

This is Round [N+1]. Assume the Critical issues from Round [N] are addressed.
Focus on Important issues and anything new introduced by the revision.
Return the same verdict format.
"""
)
```

### Exit conditions

1. Opus returns **SHIPPABLE** with no Critical issues, OR
2. You approve manually ("ship it", "good enough"), OR
3. After 3 rounds — if still NEEDS REVISION, flag for judgment

---

## Verdict Format

Keep a running log:

```
Round 1: NEEDS REVISION — 2 Critical, 3 Important
Round 2: NEEDS REVISION — 0 Critical, 1 Important
Round 3: SHIPPABLE — approved
```

---

## Prompt Variants

**Technical design:** add to the prompt:
> "Also check: Is the implementation order correct? Are there missing dependencies? Does this create any security or data integrity risks?"

**Code review:** replace the standard prompt opener with:
> "Review this [language] file. Context: [what it does, runtime, concurrency model, what callers pass in]. This is Round 1. Focus on structural problems and critical issues that would cause failures, hangs, or wrong output at runtime."

For code: fixes are applied directly (`Edit` the file + rebuild). Apply after each round before continuing.

**Strategy or brief:** add to the prompt:
> "Also check: Is the argument defensible? Are there counterarguments that aren't addressed? Is the ask clear?"

---

## Output

When SHIPPABLE:
1. Save `final.md` (copy of approved version)
2. Write `summary.md`:
   ```
   # Critique Session: <name>
   Date: YYYY-MM-DD | Rounds: N

   | Round | Verdict | Critical | Important | Key changes |
   |-------|---------|----------|-----------|-------------|
   | 1 | NEEDS REVISION | 2 | 3 | Added token expiry, clarified rollback |
   | 2 | SHIPPABLE | 0 | 0 | — |
   ```

---

## Example

```
You: /criticon docs/plan-cloud-auth.md

Round 1: NEEDS REVISION
  🔴 Critical: token expiry not handled — refresh loop will run forever
  🟡 Important: no rate limiting on /api/keys endpoint
  🟢 Minor: error messages expose internal stack traces

[You fix Critical + Important, rewrite plan]

Round 2: SHIPPABLE
  No critical or important issues found.

SHIPPABLE after 2 rounds. Audit trail in criticon/cloud-auth-2026-04-28/
```

---

*CRITICON v1.0 — April 2026*  
*Part of the [FRAGUA](FRAGUA.md) protocol.*
