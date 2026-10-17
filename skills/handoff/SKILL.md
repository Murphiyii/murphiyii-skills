---
name: handoff
description: Compacts the current conversation into a handoff document for another agent to pick up. Use when ending a session, switching to a fresh agent or context window, or when the user asks for a handoff or session summary.
argument-hint: "What will the next session be used for?"
---

Write a handoff document summarising the current conversation so a fresh agent can continue the work. Save to the current workspace in folder /handoff.

Use this default outline and adjust the sections to the work: goal, current state, decisions made, next steps, open questions, references.

Name the file with the current date and time embedded, in the form `handoff-YYYY-MM-DD-HHmm.md` (e.g. `handoff-2026-08-13-1430.md`). Run `date +"%Y-%m-%d-%H%M"` in a terminal (PowerShell: `Get-Date -Format "yyyy-MM-dd-HHmm"`) to get the exact timestamp before creating the file; do not guess the time.

Do not duplicate content already captured in other artifacts (specs, plans, ADRs, issues, commits, diffs). Reference them by path or URL instead.

Redact any sensitive information, such as API keys, passwords, or personally identifiable information. Before finishing, re-read the saved file and confirm no secrets or personal data remain.

If the user passed arguments, treat them as a description of what the next agent will focus on and tailor the handoff document accordingly.