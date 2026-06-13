---
name: 01-ai-detection-signals
description: Detect and score the likelihood that code was AI-generated using weighted, evidence-based signals
---

# AI Detection Signals

## Purpose

Scan a file, a diff, or a snippet and produce an evidence-based assessment of how likely it
is to have been AI-generated. Output a likelihood band plus the concrete signals that fired,
with file:line references. Never claim certainty — detection is probabilistic.

## Activation triggers

- "Is this code AI-generated?"
- "Check this file / PR / diff for AI"
- "Score how AI-written this is"
- "What gives away that this was written by an LLM?"

## How to run a detection pass

1. Read the target code (and 1–2 neighboring files to establish the project's baseline style).
2. Walk the signal categories below. For each hit, record `file:line` + a one-line reason.
3. Weight the hits and produce a likelihood band.
4. Report findings; do not modify code in this step (cleanup is `03-trace-cleanup.md`).

## Signal catalogue

Each signal has a weight: **high** (strong tell), **medium**, **low** (weak, needs corroboration).

### Comments & documentation
- **high** — Comments that restate the code line-by-line (`# increment counter` above `i += 1`).
- **high** — Numbered narration comments (`# Step 1: ...`, `# Step 2: ...`) tracking prose, not logic.
- **medium** — Every function has a full docstring even for trivial one-liners.
- **medium** — Docstrings in a rigid uniform template (Args/Returns/Raises) across an otherwise casual codebase.
- **medium** — Comments explaining *what* obvious code does rather than *why*.
- **low** — "Note:", "Here we", "We'll now", "Let's" lead-ins inside comments.

### Naming & structure
- **medium** — Hyper-generic names: `data`, `result`, `temp`, `item`, `value`, `helper`, `process_data`.
- **medium** — Placeholder examples left in: `foo`, `bar`, `example.com`, `your_api_key_here`, `John Doe`.
- **medium** — Over-modularization: tiny one-call wrapper functions that add no abstraction.
- **low** — Suspiciously even, "textbook" structure with perfectly balanced sections.

### Defensive / redundant code
- **high** — Try/except (or try/catch) wrapping code that cannot throw, or catching `Exception` then `print`.
- **medium** — Re-validating inputs already validated upstream; redundant null checks.
- **medium** — Unnecessary type checks / `isinstance` guards on internal calls.
- **low** — Logging added at every step regardless of usefulness.

### Prose tells (in comments, docstrings, READMEs, commit messages)
- **high** — Marketing tone: "robust", "seamless", "powerful", "comprehensive", "elegant", "leverage".
- **high** — Emoji decoration in code comments or section headers (✅, 🚀, 📝) where the project uses none.
- **medium** — Em dashes and "It's worth noting that" / "In summary" / "Overall" connective phrasing.
- **medium** — Section headers like "Conclusion", "Key Features", "Benefits" inside a code file's comments.
- **low** — Hedging phrases: "you may want to", "depending on your needs", "feel free to".

### Formatting & artifacts
- **high** — Literal leftover scaffolding: `# TODO: implement`, `# Your code here`, `pass  # placeholder`.
- **high** — Markdown code fences accidentally left inside source (```), or "Here is the code:" preamble.
- **medium** — Uniform style that ignores the repo's existing conventions (tabs vs spaces, quote style).
- **medium** — Trailing explanatory text that doesn't belong in source (e.g., a paragraph after the last line).
- **low** — Perfectly consistent, never-abbreviated identifiers in a codebase that abbreviates.

### Version-control tells
- **medium** — A single massive commit introducing a fully-formed feature with uniform style.
- **medium** — Commit messages in promotional tone or with conventional-commit emoji where none existed.

## Scoring

Tally weighted hits:

| Weight | Points |
|--------|--------|
| high   | 3      |
| medium | 2      |
| low    | 1      |

Sum the points across distinct signals (count a recurring signal once per category, but note
its frequency).

| Total points | Likelihood band |
|--------------|-----------------|
| 0–2          | Unlikely AI-generated |
| 3–6          | Possibly AI-assisted |
| 7–12         | Likely AI-generated |
| 13+          | Very likely AI-generated |

Adjust down if the codebase is genuinely junior/heavily-templated by humans, and up if multiple
**high** signals co-occur. Always state the adjustment reasoning.

## Output format

```markdown
## AI Detection Report — <file or diff>

**Assessment:** Likely AI-generated (score 9)

### Signals
| # | Signal | Weight | Location | Evidence |
|---|--------|--------|----------|----------|
| 1 | Line-by-line restating comments | high | utils.py:12–18 | `# loop over items` above `for item in items:` |
| 2 | Generic naming | medium | utils.py:8 | `def process_data(data):` |
| 3 | Redundant try/except | high | api.py:40–47 | catches `Exception`, only re-prints |
| 4 | Marketing prose in docstring | high | README.md:1 | "a robust, seamless solution" |

### Notes
- No single signal is conclusive; the score reflects co-occurrence of 2 high-weight tells.
- Baseline: neighboring files use terse comments and abbreviated names, so the uniform
  docstrings stand out.

### Recommendation
Proceed to cleanup → load `03-trace-cleanup.md`.
```

## Cautions

- Well-written human code can trip low-weight signals. Require corroboration before concluding.
- A clean score does **not** prove human authorship; it means few traces remain.
- Report evidence, not vibes. Every claim needs a `file:line`.
