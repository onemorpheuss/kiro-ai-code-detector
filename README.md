
# AI Code Detector & Cleaner

A focused power for reviewing code and identifying the telltale signs that a file (or a
diff) was likely produced by an AI assistant, then cleaning those traces so the code reads
as natural, idiomatic, and consistent with the surrounding project.

This is a **code-quality** tool. The "AI traces" it targets are real maintainability
problems: redundant boilerplate comments, over-defensive error handling, generic naming,
inconsistent style, placeholder scaffolding, and marketing-flavored prose. Removing them
produces cleaner, more reviewable code.

## What this power does

1. **Detect** — Scan code and assign an "AI-likelihood" assessment based on weighted signals.
2. **Explain** — Report exactly which signals fired, where, and why they read as AI-authored.
3. **Clean** — Rewrite/refactor to remove the traces while preserving behavior, matching the
   project's existing conventions.
4. **Verify** — Confirm behavior is unchanged (build/tests/lint) after cleanup.

## When to load steering files

Load the steering file that matches the user's task:

- Detect / score whether code is AI-generated, list the signals → `01-ai-detection-signals.md`
- Review code quality, run a structured review pass before cleanup → `02-code-quality-review.md`
- Remove AI traces: comments, naming, defensive code, structure → `03-trace-cleanup.md`
- Normalize prose, docstrings, commit messages, READMEs ("humanize" text) → `04-prose-humanizer.md`
- Batch-process a whole repo or a diff, with a repeatable workflow → `05-batch-workflow.md`

## Core principles

- **Behavior must not change.** Cleanup is refactoring, not rewriting logic. Verify with the
  project's build/test/lint after every pass.
- **Match the project, not a generic ideal.** Read neighboring files first; mirror their
  naming, comment density, and structure rather than imposing a new style.
- **Detection is probabilistic.** No single signal proves AI authorship. Report likelihood
  with the evidence, never a false-certainty verdict.
- **Don't strip meaning.** Remove redundant/noise comments, keep comments that explain
  *why*. Keep error handling that the code genuinely needs.

## Ethical note

This power is for improving code you own or are authorized to edit — cleaning up generated
scaffolding, normalizing style, and making AI-assisted output production-ready. It is not a
tool for academic dishonesty or for defeating integrity checks where disclosure of AI use is
required. Use it to make your own code better, and follow any disclosure rules that apply to
your context.
