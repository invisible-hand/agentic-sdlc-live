---
name: harvest
description: Turn what the review found into something permanent, so the next run starts better.
disable-model-invocation: true
---

Take the confirmed findings and the human's decisions from this session, then route each one:

| What it is | Where it goes |
|---|---|
| CI or the hook already catches it | Nowhere, that layer worked |
| A machine can check it | A test in `tests/` |
| A human decided it | A one-line rule in `AGENTS.md` |
| The spec should have asked | A sharper question in the `spec` skill |

Propose at most three changes, smallest first. Every change goes to `main`. If a test would fail
on `main` today, do not add it: name it under "Add with the fix" in one line.

Keep the answer short enough to read aloud. For each change: one line saying what and where, then
the diff. For a new test file, list the test names, not the code. No root-cause analysis. "Left
out" is at most three one-line items.

Apply the changes only after the human says yes.
