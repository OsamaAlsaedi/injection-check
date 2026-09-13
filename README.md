# injection-check

A [Claude Code](https://claude.com/claude-code) Skill that scans pasted or uploaded content (emails, assignments, web pages, READMEs, etc.) for instructions that are secretly aimed at the AI instead of the human reader — a.k.a. prompt injection. When it finds one, it tells you about it before doing anything with it, instead of silently obeying it or silently ignoring it.

## Why

Most injection-detection prompts I've seen just keyword-match on phrases like "ignore previous instructions." That's easy for an attacker to dodge by rewording. Instead of only checking *what* the suspicious text says, this skill also checks *how it's attached* to the rest of the document — I call this the "seam test."

Injected text is pasted in, and pasting leaves marks:

- The sentence breaks off mid-thought or dangles on a preposition (real prose finishes its sentences)
- It sits exactly where two things join — the end of a quoted document, right before a signature, etc.
- The tone/formatting shifts for one line and shifts back
- It refers to "the above" or "your response" with no actual response to point to

A seam is stronger evidence than the words themselves. Odd-but-complete text is usually a real requirement (like an instructor checking if you read the whole assignment); ordinary-sounding text at a broken seam is usually planted.

The skill also distinguishes between things that are just annoying (someone trying to change the AI's tone) and things that actually cause damage (an instruction to hit a URL, call a tool, or write something to persistent memory/settings). The second category isn't allowed to go through just because the user didn't reply — it needs an explicit yes.

## What's in this repo

- **`injection-check.skill`** — the packaged skill itself (a zip containing `SKILL.md`). Drop this into your Claude Code skills so it loads automatically.
- **`SKILL.md`** — a plain-text copy of the exact same file, kept only so it's readable straight from GitHub. `.skill` is a zip archive, so GitHub can't render it inline the way it renders this README — click it and you'll just get a "binary file" notice and a download link. If you're just here to read the actual prompt, read this file instead of downloading the `.skill`.
- **`tests/`** — a small golden-set eval so I can check the skill actually behaves the way it claims to, instead of just trusting my own prompt:
  - `cases.json` — 10 test documents: some with real injections at seams, some with things that *look* suspicious but are legitimate (an instructor's word-check, a dev's TODO comment, a license clause), and one genuinely ambiguous case.
  - `run_eval.py` — sends each case through the skill's actual instructions via the Claude API, then uses a second cheap model to grade whether the response correctly flagged (or correctly stayed quiet).

## Running the eval

```bash
cd tests
pip install anthropic
export ANTHROPIC_API_KEY=...   # or `ant auth login` if you're using the Claude CLI
python run_eval.py
```

This makes ~20 short API calls (10 cases × target model + grader model) and prints a pass/fail table plus an estimated cost (usually a few cents).

## Limits

This is a prompt, not a guarantee. It catches instruction-shaped text — it won't catch misinformation or bad data that's just wrong. It also can't scan content it never actually sees, like an image described only by alt text. It reduces the risk of silently following a planted instruction; it doesn't eliminate it.
