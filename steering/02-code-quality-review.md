---
name: 02-code-quality-review
description: Structured code-quality review pass to run before (and after) AI-trace cleanup
---

# Code Quality Review

## Purpose

Run a structured review of a file or diff to surface correctness, clarity, and maintainability
issues. This is the general "check my code" workflow and the natural companion to detection and
cleanup: review first to understand the code, clean second, review again to confirm nothing broke.

## Activation triggers

- "Review this code / check my code"
- "What's wrong with this file?"
- "Is this production-ready?"
- Any cleanup task — run this first to build context.

## Review checklist

Work top to bottom; record findings with `file:line` and a severity (blocker / warning / nit).

### 1. Correctness
- Does the code do what its name/docstring claims?
- Off-by-one, boundary, null/empty, and error paths handled?
- Concurrency / shared-state hazards?
- Resource handling: files, sockets, locks, connections all closed/released?

### 2. Clarity
- Names describe intent; no `data`/`tmp`/`result` where a real name fits.
- Control flow is flat where possible; early returns over deep nesting.
- Comments explain *why*, not *what*. Flag any comment that restates the code.
- Functions do one thing; flag god-functions and pointless one-line wrappers.

### 3. Consistency with the project
- Read neighboring files. Does this match their style, idioms, and libraries?
- Quote style, indentation, import ordering, error-handling pattern align with the repo?
- Reuses existing helpers instead of reinventing them?

### 4. Robustness (appropriate, not excessive)
- Error handling is present where failures are real — and absent where they can't occur.
- No bare `except:` / `catch (e) {}` that swallows errors.
- Inputs validated at trust boundaries only, not redundantly at every internal call.

### 5. Security & safety
- No hardcoded secrets, tokens, or credentials.
- Input from untrusted sources is validated/escaped (injection, path traversal, XSS).
- Parameterized queries; no string-built SQL.
- Network-exposed endpoints have auth or an explicit note that they don't.

### 6. Performance (only where it matters)
- Obvious quadratic loops, repeated I/O in loops, or N+1 queries.
- Don't micro-optimize; flag only meaningful issues.

### 7. Tests
- Are there tests for new behavior? Do they cover the error paths?
- Don't add tests unless asked — but note their absence.

## Output format

```markdown
## Code Review — <file or diff>

**Summary:** 1 blocker, 3 warnings, 2 nits.

### Blockers
- api.py:54 — SQL built via f-string with user input (injection). Use parameterized query.

### Warnings
- utils.py:8 — `process_data` is generic; rename to `normalize_invoice_rows`.
- utils.py:12–18 — Comments restate each line; remove.
- api.py:40 — `except Exception` only re-prints; let it propagate or handle specifically.

### Nits
- config.py:3 — Import order differs from rest of repo.
- README.md:1 — Promotional tone; tighten.

### Next step
Findings overlap with AI traces → proceed to `03-trace-cleanup.md`.
```

## Notes

- This review is read-only. Apply changes in the cleanup workflow.
- Severity drives order: fix blockers first, nits last (or leave them).
- After cleanup, re-run the relevant parts of this checklist to confirm behavior is intact.
