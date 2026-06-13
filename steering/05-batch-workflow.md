---
name: 05-batch-workflow
description: Repeatable end-to-end workflow to detect and clean AI traces across a whole repo or a diff
---

# Batch Workflow

## Purpose

A repeatable pipeline for running detection + cleanup across a whole repository, a directory, or a
single diff/PR — instead of one file at a time. Optimized for safety (behavior preserved) and for
delegating wide scans to sub-agents.

## Activation triggers

- "Scan the whole repo for AI code"
- "Clean AI traces across src/"
- "Run this on my PR / diff"

## Pipeline

### Step 0 — Scope & safety
- Confirm the scope: whole repo, a folder, staged changes, or a specific PR/diff.
- Confirm the working tree is committed/clean (so changes are reversible via VCS). If dirty, ask
  before proceeding.
- Confirm build/tests pass *before* starting; capture the baseline. If they fail, note it.

### Step 1 — Inventory
- List candidate files in scope. For a diff, restrict to changed files/hunks.
- Identify the project's build, test, and lint commands (read package.json / Makefile / pyproject /
  cargo / pom, etc.).

### Step 2 — Detect (parallelizable)
- For large scopes, delegate detection to sub-agents: split files into batches and have each batch
  return an AI Detection Report (per `01-ai-detection-signals.md`).
- Aggregate into a ranked list: files by likelihood band + signal count.

### Step 3 — Triage
- Present the ranked list. Let the user pick: clean everything ≥ "Likely", a specific subset, or
  review first.
- Cleaning code you don't own or aren't authorized to edit is out of scope — confirm ownership for
  unexpected files.

### Step 4 — Clean (sequential, verify between batches)
- Apply `03-trace-cleanup.md` per file/batch, smallest-risk category first.
- After each batch: run build + tests + lint. Stop on first failure, fix or revert, then continue.
- Handle prose files (README/docs/commits) via `04-prose-humanizer.md`.

### Step 5 — Report
- Summary table: files touched, traces removed by category, verification status.
- Note anything intentionally left (load-bearing error handling, ambiguous style).
- Suggest the user review the diff before committing.

## Using sub-agents

- Detection fans out well — use `general-task-execution` or `context-gatherer` sub-agents to scan
  file batches in parallel and return structured reports.
- Keep **cleanup sequential** on the main agent so verification stays coherent and diffs don't
  collide.

## Optional: a hook

To re-check on every save, suggest a `fileEdited` hook that runs the linter, or an `agentStop` hook
that reminds you to run detection on changed files. Keep hooks advisory; don't auto-rewrite files
on save without the user opting in.

## Output format

```markdown
## Batch AI Cleanup — src/ (37 files)

**Baseline:** tests green (210 passed), lint clean.

### Detection ranking
| File | Band | Score |
|------|------|-------|
| src/api/handlers.py | Very likely | 15 |
| src/utils/io.py | Likely | 9 |
| ... | ... | ... |

### Cleaned (12 files)
| Category | Count |
|----------|-------|
| Restating comments removed | 63 |
| Generic names renamed | 11 |
| Redundant try/except removed | 7 |
| Marketing prose rewritten | 4 docs |

**Verification:** tests green (210 passed), lint clean. No behavior change.

### Left intentionally
- src/db/pool.py:88 — retry/except is load-bearing.

Review the diff before committing.
```

## Cautions

- Never run destructive VCS commands (reset --hard, clean -f) as part of this flow.
- Don't push or commit unless explicitly asked.
- Stop and ask if the scope includes vendored/third-party code or generated files.
