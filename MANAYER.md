# MANAYER

> *"Swimming upstream against complexity, one phase at a time."*

Three-role build workflow: **you** write the scope (manager), a **coder agent** implements it in a clean isolated context, a **reviewer agent** audits the output, **you** apply the critical fixes.

Named after "manager" as pronounced with a Latin American accent. Part of the [FRAGUA](FRAGUA.md) protocol.

---

## When to Use

- Building a feature that touches 3+ files
- Implementing work from an existing plan document
- UI redesigns, refactors, or new modules
- Any build complex enough that doing it all in one context would exhaust your window

**Don't use for:** single-file edits, config changes, quick fixes — do those directly.

---

## Audit Trail

```
manayer/<build-name>-YYYY-MM-DD/
  scope.md          ← implementation plan (Phase 0)
  coder-summary.md  ← what the coder built (files touched, decisions)
  review.md         ← reviewer findings (CRITICAL/HIGH/MEDIUM/LOW)
  fixes.md          ← fixes you applied after review
  summary.md        ← build stats
```

---

## The Three Phases

### Phase 0: SCOPE (you, before spawning agents)

This is the most important phase. Agents are only as good as their brief.

Read the codebase first. Understand what's there before writing what to change.

Write an implementation plan:

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
### file.ts
- Add `newEndpoint()` after line ~120
- Update `existingFunction()` to accept new param

## Data Flow
[Where data comes from → transforms → where it goes]

## Constraints
- [ ] No new dependencies unless necessary
- [ ] Auth must be preserved
- [ ] Keep files under 500 lines

## Known Pitfalls
[Edge cases, framework gotchas, things that burned you before]

## Verify
[Build command, test command, restart command, smoke test]
```

Save as `docs/plan-[feature].md` and `manayer/<build>-YYYY-MM-DD/scope.md`.

---

### Phase 1: CODER Agent (background)

Spawn a coder with the full implementation plan. Background mode — do not poll.

```
Agent({
  description: "[Feature] coder",
  subagent_type: "coder",
  run_in_background: true,
  mode: "bypassPermissions",
  prompt: `You are implementing [Feature Name].

## Implementation Plan
[full plan from Phase 0]

## Project Location
[absolute path]

## Read First
[list key files the coder must read before editing]

## Rules
- Read every file before editing
- Keep files under 500 lines
- All imports at top — no inline imports
- Follow existing code patterns
- Preserve all existing functionality (auth, error handling)
- Run the verify commands when done and report results`
})
```

**While the coder runs:** do other work. Do NOT poll status or check for output early.

---

### Phase 2: REVIEWER Agent (background)

After the coder completes, spawn a reviewer. Background mode.

```
Agent({
  description: "[Feature] reviewer",
  subagent_type: "reviewer",
  run_in_background: true,
  prompt: `Review changes to [Project] at [path].

## What was built
[1-2 sentence summary from coder output]

## Files changed
[list from coder output]

## Checklist
- CRITICAL: Security (no secrets in output, auth preserved, input validation)
- CRITICAL: Data integrity (correct calculations, no data loss, race conditions)
- CRITICAL: Backward compatibility (existing features still work)
- HIGH: Import correctness (all at top, no unused)
- HIGH: Error handling (try/finally for resources, timeout handling)
- HIGH: Edge cases (empty data, API failures, null values)
- MEDIUM: Code style (consistent with project patterns)
- MEDIUM: Performance (no N+1 queries, pagination for large datasets)
- LOW: Naming, comments, formatting

## Known pitfalls
[project-specific gotchas you know about]

Classify each finding as CRITICAL / HIGH / MEDIUM / LOW.
For CRITICAL/HIGH: include exact file, line number, and fix.`
})
```

---

### Phase 3: YOUR FIXES (main context)

After the reviewer completes:

1. **Read CRITICAL and HIGH findings** — non-negotiable
2. **Save reviewer output** → `manayer/<build>-YYYY-MM-DD/review.md`
3. **Fix them yourself** — you have the context to judge what's real vs noise. Don't spawn a third agent.
4. **Save your fixes** → `manayer/<build>-YYYY-MM-DD/fixes.md`
   ```markdown
   # Fixes Applied
   ## CRITICAL
   - [file:line] what was fixed
   ## HIGH
   - [file:line] what was fixed
   ## Skipped
   - [reason LOW/MEDIUM items were skipped]
   ```
5. **Skip LOW** unless trivial
6. **Verify:** build → tests → restart service → smoke test
7. **Write summary** → `manayer/<build>-YYYY-MM-DD/summary.md`
   ```markdown
   # MANAYER Build: [Feature Name]
   Date: YYYY-MM-DD

   ## Stats
   - Files touched: N
   - Reviewer findings: C critical, H high, M medium, L low
   - Fixes applied: N
   - Verify: PASS / FAIL

   ## What was built
   [1-2 sentences]

   ## Key decisions
   [Anything non-obvious]
   ```

---

## Anti-Patterns

- **Don't skip Phase 0** — vague prompts produce generic wrong code
- **Don't spawn coder + reviewer together** — reviewer needs the coder's output
- **Don't spawn a third agent to apply fixes** — you fix, because you have the context to judge
- **Don't poll agent status** — wait for notification
- **Don't skimp on the scope** — 15 minutes of planning saves 2 hours of rework

---

## Why Three Roles

Each role has an isolated context window.

The **coder** gets the scope doc, not your conversation history. It doesn't know about the last 3 features you built, the false start from yesterday, or the architectural debate from last week. It just knows what to build. Clean context = clean execution.

The **reviewer** gets the diff and a checklist. It doesn't need to understand the full system — it needs to evaluate what changed.

**You** hold the architecture, the history, and the judgment. When the reviewer flags something, you know whether it's a real problem or a false positive. That judgment isn't delegatable.

Three small windows instead of one giant compounding one. That's where the token efficiency comes from.

---

## Stack-Specific Verify Commands

**TypeScript / Node.js:**
```bash
npm run build        # catch type errors
npm test             # run test suite
npm run lint         # style check
```

**Python:**
```bash
python -c "import your_module"   # catch import errors
pytest tests/                     # run test suite
```

**After building:** restart your service and run a smoke test against the changed endpoints.

---

*MANAYER v1.0 — April 2026*  
*Part of the [FRAGUA](FRAGUA.md) protocol.*
