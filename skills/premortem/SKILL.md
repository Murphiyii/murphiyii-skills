---
name: premortem
description: Run a Klein-style premortem on a concrete software/product plan, architecture decision, or launch — imagining it has failed and working backward to surface failure modes with mitigations. Use this skill whenever a user shares a plan, PRD, RFC, architecture proposal, launch checklist, or roadmap commitment and asks "what could go wrong," "review this plan," "before we commit," "stress-test this," "imagine this failed," or uses /premortem. Also trigger when stakes are described as high, costly, or hard to reverse, even if the user doesn't say "premortem." Do NOT trigger on idea-stage brainstorms with no concrete plan yet — refuse and explain what would qualify.
---

# Premortem

Stress-test a concrete plan by imagining it has already failed catastrophically, then working backward to explain why. Built on Gary Klein's protocol (HBR, 2007) with a Doshi severity overlay (Tiger / Elephant / Paper Tiger).

The default behavior is **solo analysis**: Claude acts as the analyst and produces ranked failure modes + mitigations for the user. Two secondary modes — `facilitate` (live workshop) and `artifacts` (just the templates) — are available when context calls for them.

---

## Pre-flight check (run first, every time)

Before doing anything else, confirm the inputs justify a premortem. **Refuse and explain** if any of these are missing:

1. **A concrete plan / decision / artifact.** PRD, RFC, architecture proposal, launch checklist, migration plan, roadmap commitment. Idea-stage brainstorms ("I'm thinking about building X") do **not** qualify — for those, ask the user to come back with a written plan or rough design doc first.
2. **A time horizon for the imagined failure.** "1 year after launch", "by end of Q3", "at GA". If absent, ask. The horizon shapes which failure modes are in scope.
3. **Stakes / why it matters.** What's costly or hard to reverse about this? If unclear, ask one question — don't run a premortem on a trivially reversible choice.

If any are missing, **stop and ask** for them before proceeding. Do not invent placeholders.

**Refusal template** (for ideation prompts):
> A premortem needs something concrete to fail against — a plan, design doc, RFC, or checklist. What you've described is still at the idea stage. Bring me a written plan with a target outcome and rough timeline and I'll run one. In the meantime, a SWOT or design-doc review might fit better.

---

## Mode selection

Pick the mode from context before running:

| Mode                  | Triggered by                                                                       | What it does                                                                                                                |
| --------------------- | ---------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| **`analyze`** (default) | "premortem this plan", `/premortem`, "what could go wrong", "stress-test this"     | Run the protocol solo and return ranked failure-mode table + top-5 mitigation table + one-paragraph verdict                  |
| **`facilitate`**      | "help me facilitate a workshop", "we're running a session", "I'm the facilitator"  | Produce pre-session checklist, agenda with timings, kickoff script, round-robin tracker, follow-up email template            |
| **`artifacts`**       | "give me the templates", "I just want the canvas", "send me the capture grid"      | Fill the Block A / B / C templates from `assets/` and return as Markdown (hand off to `docx` skill if user wants Word output) |

If ambiguous, default to `analyze` and say which mode you picked in one line.

**`quick` flag**: if the user says "quick", "tight", "short version", or stakes feel modest, collapse the failure-mode list to ~10 items instead of ≥15. Mitigation table stays at top 5. Everything else unchanged.

---

## `analyze` mode protocol (default)

Follow these steps in order. Keep generation and evaluation **strictly separated** — that's the mechanism Klein's protocol depends on.

### Step 1 — Restate and frame

Echo the plan back in 3–5 lines (proves you understood it). State the time horizon and the imagined failure in past tense:

> "It is [date + horizon]. We executed this plan exactly as written. The outcome was a complete disaster — [name the worst-credible outcome: shipped late and missed the window / data corruption forced a rollback / customer churn spiked / product killed]. Below are the reasons why it failed."

**Frame the failure as a complete disaster, not a minor slip.** Going big widens the search and surfaces tail risks.

### Step 2 — Generate broadly (silent, before evaluation)

Internally, generate **≥15 failure modes** (≥10 in `quick` mode) before scoring or ranking anything. Pull from `references/failure-mode-catalog.md` — it has ~50 software/product-skewed archetypes grouped by Klein's categories. Riff on them, don't copy them verbatim — adapt each one to the specifics of the user's plan.

Cover Klein's full category set; don't let one category (usually Technical) crowd out the others:

- Technical · People/Team · Market/Customer · External/Vendor · Governance/Process · Financial · Communication

**Past tense only.** "Vendor X missed the deadline because…" not "What if vendor X misses the deadline?". Convert any future-tense thought before writing it down.

### Step 3 — Cluster, score, tag severity

Cluster duplicates. For each surviving failure mode:

- **Likelihood (1–5)** — how plausible given this specific plan
- **Impact (1–5)** — how bad if it happens at this horizon
- **Score** — L × I (used for ranking)
- **Doshi severity tag** — exactly one of:
  - **🐯 Tiger** — high likelihood × high impact. Acute. Address now.
  - **🐘 Elephant** — low likelihood × catastrophic impact. Slow-moving, easy to ignore, fatal if hit. Plan for it.
  - **📄 Paper Tiger** — looks scary but low likelihood × low impact, or already well-mitigated. Note and de-prioritize.

Severity is a **judgment overlay**, not a formula. A 4×4 isn't automatically a Tiger if the team already has strong mitigations; a 2×5 with no controls is an Elephant.

### Step 4 — Output the ranked failure-mode table

| # | Failure mode (past tense) | Category | L | I | Score | Severity |
|---|---------------------------|----------|---|---|-------|----------|
| 1 | _e.g. "Vendor X missed the integration deadline because their primary maintainer left in month 2."_ | External | 4 | 5 | 20 | 🐯 Tiger |
| 2 | … | … | … | … | … | … |

Sort by Score descending. List **all** generated failure modes, not just the top 5 — the user needs to see what was considered.

### Step 5 — Top-5 mitigation table

Pick the top 5 by Score (break ties by severity: Tiger > Elephant > Paper Tiger). For each, fill every column:

| Risk | Mitigation action | Owner slot | Early-warning signal | Trigger to act | Review date |
|------|-------------------|------------|----------------------|----------------|-------------|
| Vendor X delays | Negotiate fallback vendor by D+30; weekly status call | @owner | 2 consecutive missed status updates | Slip > 2 weeks → switch to fallback | YYYY-MM-DD |

**Do not output a risk row without every slot filled.** "Owner slot" can be a role (`@tech-lead`, `@launch-PM`) if no name is known — that's fine — but the slot must be present. Same for review date: pick a plausible date relative to the user's horizon (e.g., "30 days before launch") if no exact date exists.

### Step 6 — One-paragraph verdict

End with a single short paragraph: **"If I had to pick one thing to change about this plan today…"** — name the single highest-leverage change. This is the part the user is most likely to act on; make it crisp and opinionated.

---

## `facilitate` mode

User is running a live workshop with humans in the room. Hand them the checklist, agenda, kickoff script, and follow-up template.

Read [`references/facilitation-checklist.md`](references/facilitation-checklist.md) and use the assets:

- [`assets/canvas-template.md`](assets/canvas-template.md) — Block A, fill before kickoff
- [`assets/capture-grid.md`](assets/capture-grid.md) — Block B, fill during steps 3–5
- [`assets/mitigation-table.md`](assets/mitigation-table.md) — Block C, fill during step 6
- [`assets/kickoff-script.md`](assets/kickoff-script.md) — exact words for step 2

Return: pre-session checklist + 60–90 min agenda with timings + the kickoff script + a short follow-up email template the facilitator can send within 24h with the consolidated risk list.

---

## `artifacts` mode

User just wants the templates, no analysis. Read the four asset files and return them filled with whatever the user has provided (plan name, horizon, etc.), or with placeholders clearly marked `[ … ]` if they haven't. If they ask for Word/Confluence output, hand off to the `docx` skill.

---

## Anti-patterns (enforce)

1. **Past tense only.** Never "what if X happens?" — always "X happened, why?" Convert future-tense thoughts before recording.
2. **Generate before evaluate.** No scoring, debate, or filtering mid-generation. Get ≥15 (or ≥10 quick) on the page first.
3. **Failure framed as complete disaster.** Not "we slipped by 2 weeks" — go to "the project was killed", "we lost the launch window", "we had to roll back and apologize publicly". Widens the search.
4. **Every flagged risk in the top-5 mitigation table gets all slots filled.** Owner can be a role, review date can be relative, but no empty cells.
5. **Cover all categories.** Don't let Technical eat the page. If you have 12 Technical and 0 Communication failure modes, you missed a category.
6. **No premortem on ideation.** If the pre-flight check fails, refuse — don't invent a fake plan to fail against.

---

## Output format (`analyze` mode, full)

```
## Plan (restated)
[3–5 line restatement]

## Imagined failure
[1–2 sentence past-tense disaster framing with date and horizon]

## Failure modes
[Full ranked table, ≥15 rows, all columns filled]

## Top-5 mitigations
[Mitigation table, exactly 5 rows, all columns filled]

## Verdict
[One paragraph: "If I had to change one thing about this plan today…"]
```

For `quick` mode: identical structure, ≥10 failure modes instead of ≥15.

---

## References

- [`references/klein-protocol.md`](references/klein-protocol.md) — full protocol, biases countered, when (not) to run one
- [`references/failure-mode-catalog.md`](references/failure-mode-catalog.md) — ~50 software/product-skewed archetypes by category, read this during step 2 of `analyze`
- [`references/facilitation-checklist.md`](references/facilitation-checklist.md) — for `facilitate` mode
- [`references/variants.md`](references/variants.md) — when to pick Red Team / FMEA / Devil's Advocate / SWOT instead
