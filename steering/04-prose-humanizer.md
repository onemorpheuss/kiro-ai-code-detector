---
name: 04-prose-humanizer
description: Normalize AI-flavored prose in docstrings, comments, READMEs, and commit messages into natural, concise text
---

# Prose Humanizer

## Purpose

Rewrite AI-flavored text — docstrings, code comments, READMEs, changelogs, commit messages — into
concise, natural prose that matches how the project's humans actually write. This targets the
recognizable "LLM voice": padded, over-structured, adjective-heavy, and relentlessly even.

## Activation triggers

- "Humanize this README / docstring / commit message"
- "This text sounds AI-written, fix it"
- "Make these comments sound natural"

## The LLM-voice tells to remove

- **Marketing adjectives:** robust, seamless, powerful, comprehensive, elegant, cutting-edge,
  world-class, effortless. Replace with concrete facts or delete.
- **Filler connectives:** "It's worth noting that", "In summary", "Overall", "Furthermore",
  "Moreover", "That being said". Cut them.
- **Hedging:** "you may want to", "depending on your needs", "feel free to", "simply". Cut or commit.
- **Over-structure:** every doc broken into Key Features / Benefits / Conclusion with bullet
  lists where a sentence would do.
- **Emoji decoration:** ✅ 🚀 📝 in headers and bullets, when the project doesn't use them.
- **Em-dash rhythm:** the constant "X — which does Y — then Z" cadence. Vary it; use periods.
- **False balance:** every paragraph the same length, every list exactly three items.
- **Restating the obvious:** intros that announce what the next section will say.

## Rewriting principles

1. **Match the project's voice.** Read existing human-written docs/commits. Mirror their length,
   formality, and structure. A terse repo gets terse text.
2. **Lead with the point.** Say what it is / does in the first sentence. Cut the windup.
3. **Concrete over promotional.** "Caches responses for 60s" beats "delivers blazing-fast,
   robust performance".
4. **Cut by half, then check.** AI prose is usually 2x too long. Trim, then verify nothing
   essential was lost.
5. **Keep it accurate.** Don't drop real constraints, caveats, or steps while trimming fluff.

## Commit messages

- Imperative mood, concise subject (<70 chars): "Add retry to upload client".
- Body explains *why* and any tradeoffs, not a play-by-play of the diff.
- Drop conventional-commit emoji unless the repo already uses them.
- No promotional summaries ("This powerful change seamlessly improves…").

## README / docs

- One-line description first, then a real usage example, then specifics.
- Remove "Key Features" sections that just list adjectives.
- Keep install/usage/config accurate and minimal.

## Examples

**Before (AI voice):**
> 🚀 This powerful utility provides a robust and seamless way to effortlessly process your data.
> It's worth noting that you may want to configure it depending on your needs. Overall, it's a
> comprehensive solution.

**After:**
> Parses CSV exports into normalized invoice rows. Set `DELIMITER` if your export isn't comma-separated.

## Output format

Show before/after for each block you change, and a one-line note on what voice you matched
(e.g., "matched the terse style of CONTRIBUTING.md"). Don't alter meaning — flag it if trimming
would drop a real caveat.

## Caution

Don't "humanize" by inserting deliberate errors, fake typos, or casual mistakes to defeat a
detector. The goal is genuinely better, more natural writing — not forgery. If the context
requires disclosing AI assistance, that disclosure stays.
