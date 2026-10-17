# Facilitation Checklist: `facilitate` mode

For running a **live premortem workshop** with people in the room (4–10 participants,
~60–90 min). Pair this with the assets: the Block A canvas,
the kickoff script (workshop step 2 words), the Block B capture grid,
and the Block C mitigation table.

---

## Before the session

- [ ] Plan is documented and shared **at least 24 h in advance**: no one reasons
      about a phantom.
- [ ] Right people in the room: **planning team + 1–2 outside skeptics** if possible.
      Sponsor present but speaks **last**.
- [ ] Time horizon for "failure" is set explicitly (3 months? 1 year? at launch?).
- [ ] Capture surface ready: physical wall + sticky notes, OR Miro/Mural/FigJam with
      pre-built category columns.
- [ ] Optional pre-work: send the failure prompt 1 day ahead so introverts arrive
      with ideas.

## Agenda (60–90 min)

| # | Workshop step | Time | Notes |
|---|------|------|-------|
| 1 | Set the scene | 5–10 min | Restate the plan as if it's about to launch; confirm shared understanding; open the Block A canvas. |
| 2 | Declare the failure | 1–2 min | Read the kickoff script aloud. Go big: complete disaster. |
| 3 | Silent writing | 5–10 min | Everyone writes reasons in **past tense**, silently, independently. Quantity over quality. |
| 4 | Round-robin share | 15–20 min | One reason per person per turn until exhausted. Capture into the Block B capture grid. **No debate yet.** |
| 5 | Consolidate & prioritize | 10–15 min | Cluster duplicates, group by category, dot-vote top 3 (likely × damaging). |
| 6 | Strengthen the plan | 15–20 min | Top failure modes → Block C mitigation table (mitigation · owner · early-warning · trigger · review date). |

Allow 60 min for ≤6 people / small plan; 90 min for larger groups or plans.

## During: Do

- Frame strictly in **past tense**: "what caused the failure", not "what could".
- **Enforce silent writing** in workshop step 3: even 60 s of chat anchors everyone.
- **Round-robin**, one item per person per turn, until lists are exhausted.
- Capture **all** reasons; defer judgment to consolidation.
- Let the **most junior person speak first**; sponsor last.

## During: Don't

- Don't allow "what if X happens?" framing: convert it to "X happened, why?".
- Don't let discussion bleed into brainstorming: keep generation and evaluation
  phases separate.
- Don't skip the assignment step: unowned failure modes die in the doc.
- Don't run it as a one-off ritual and never revisit: schedule the review.

## After the session

- [ ] Within 24 h: circulate the consolidated failure-mode list + assigned mitigations
      (use the follow-up email template below).
- [ ] Add the top 5 failure modes to the project's standing risk register.
- [ ] Set a **calendar trigger** to re-review at the next milestone.
- [ ] Track which predicted failure modes actually materialized. This calibrates future
      premortems with this team.

---

## Follow-up email template

```
Subject: Premortem follow-up, [PROJECT], top risks & owners

Thanks all. We imagined [PROJECT] failed by [HORIZON] and worked backward.
Top risks and the mitigations we assigned:

1. [Risk], owner @[name], mitigation: [action], review by [date]
2. …
3. …
4. …
5. …

Full capture grid: [link]. We'll re-review these at [next milestone / date].
Flag anything I mis-captured by [date].
```
