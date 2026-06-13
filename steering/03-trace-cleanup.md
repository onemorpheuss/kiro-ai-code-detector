---
name: 03-trace-cleanup
description: Remove AI traces from code (comments, naming, defensive bloat, structure) while preserving behavior
---

# AI Trace Cleanup

## Purpose

Refactor code to remove the AI tells identified in `01-ai-detection-signals.md` so it reads as
natural, idiomatic work that matches the surrounding project. This is **refactoring**: behavior
must not change. Every pass ends with a build/test/lint verification.

## Activation triggers

- "Clean up the AI traces / make this look human"
- "Remove the AI-generated smell from this file"
- "Normalize this to match our codebase"

## Golden rules

1. **Preserve behavior.** No logic changes. If a change could alter behavior, it's not cleanup —
   stop and ask.
2. **Establish the baseline first.** Read 2–3 neighboring files. Mirror their comment density,
   naming, quote style, and error-handling pattern. The target is *this repo*, not a generic ideal.
3. **Don't strip meaning.** Delete redundant/noise comments; keep the ones that explain *why*.
   Keep error handling the code actually needs.
4. **Verify after every pass.** Run the project's build, tests, and linter.

## Cleanup playbook

### A. Comments
- **Remove** comments that restate the code (`# increment i` over `i += 1`).
- **Remove** numbered narration (`# Step 1`, `# Step 2`) unless the steps are non-obvious domain logic.
- **Keep & tighten** comments that explain *why*, trade-offs, or non-obvious constraints.
- **Collapse** template docstrings on trivial functions to a single line, or remove if the name says it.
- **Match density** to neighbors: if the repo barely comments, don't leave dense docstrings everywhere.

### B. Naming
- Replace generic names (`data`, `result`, `temp`, `item`, `process_data`) with intent-revealing
  names drawn from the domain. Use `semantic_rename` so all references update.
- Replace leftover placeholders (`foo`, `your_api_key_here`, `John Doe`) with real values or
  project-appropriate fixtures.
- Match the repo's abbreviation and casing conventions.

### C. Defensive / redundant code
- Remove try/except (try/catch) around code that cannot throw.
- Remove redundant null/type checks already guaranteed upstream.
- Replace bare `except Exception: print(...)` with either specific handling or letting it propagate.
- Drop logging that adds no signal; keep logging at real boundaries.

### D. Structure
- Inline pointless one-call wrapper functions; keep abstractions that earn their place.
- Flatten needless nesting (early returns).
- Reuse existing project helpers instead of reinvented equivalents.

### E. Formatting & artifacts
- Delete leftover scaffolding: `# TODO: implement`, `# Your code here`, stray ``` fences,
  "Here is the code:" preambles, trailing explanatory paragraphs.
- Remove emoji from comments/headers if the project doesn't use them.
- Run the project's formatter (prettier, black, gofmt, etc.) so style matches automatically.

### F. Prose inside code
- See `04-prose-humanizer.md` for docstrings, READMEs, and commit messages. Strip marketing
  adjectives, em-dash connective tissue, and "In summary / Overall" filler.

## Workflow

1. Run detection (`01`) and/or review (`02`) to get the list of traces with locations.
2. Snapshot a baseline: ensure the build/tests pass *before* you start (so you can attribute
   any breakage to your changes). If they already fail, note it.
3. Apply cleanup category by category, smallest-risk first (comments → formatting → naming →
   structure → defensive code).
4. After each meaningful batch, run build + tests + linter.
5. Show a concise diff summary and the verification result.

## Verification

- Run the repo's build/compile step.
- Run the test suite (or the relevant subset). Do not add new tests unless asked.
- Run the linter/formatter.
- If anything fails, fix it before presenting — or revert the offending change and report it.

## Output format

```markdown
## Cleanup — <file>

**Behavior preserved:** yes — tests green (42 passed), lint clean.

### Changes
- Removed 14 line-restating comments (utils.py).
- Renamed `process_data` → `normalize_invoice_rows` (3 refs updated).
- Removed redundant try/except at api.py:40 (call cannot raise here).
- Stripped emoji + marketing tone from module docstring.
- Ran black; whitespace now matches repo.

### Not changed (intentionally)
- Kept the retry/except at db.py:88 — that call genuinely fails on transient errors.
```

## When to stop and ask

- A "cleanup" would change behavior (e.g., removing a check that's actually load-bearing).
- The baseline build/tests were already broken.
- The project style is itself inconsistent and there's no clear neighbor to mirror.
