# Failure-Mode Catalog — Software / Product

Seed archetypes for `analyze` step 2. Each entry is a short **past-tense story**
with a *because* clause — a concrete template to riff on, **not** a label to copy
verbatim. During a premortem, adapt each one to the specifics of the user's plan,
and aim to cover every category (don't let Technical eat the page).

~50 archetypes across Klein's 7 categories.

---

## Technical

1. Scope crept silently — "must-have" features were added every sprint, so the team shipped three months late and missed the launch window.
2. Hidden tech debt surfaced under deadline — the legacy module everyone avoided turned out to be load-bearing, and refactoring it ate the entire buffer.
3. The integration broke under real load — it passed in staging but the third-party API rate-limited us at production traffic, and checkout failed during peak.
4. The data migration corrupted records — an untested edge case in the backfill silently dropped a column, and we discovered it only after customers reported missing history.
5. Infra cost blew up — autoscaling was misconfigured, a runaway job span up hundreds of instances overnight, and finance escalated the bill.
6. A latency regression crept in — a new N+1 query added 800ms to the hot path, conversion dropped, and nobody noticed until a customer complained.
7. A core dependency hit end-of-life — the framework we built on was deprecated mid-project, security patches stopped, and we were forced into an unplanned migration.
8. A security flaw shipped — an auth check was missing on an internal endpoint, it was found in a pen test the week of launch, and the release was frozen.
9. The observability gap hid the failure — there were no alerts on the new service, so a partial outage ran for six hours before anyone saw it.
10. The rollback didn't work — the deploy was one-way because the schema change wasn't backward-compatible, so the bad release couldn't be reverted cleanly.

## People / Team

11. The key engineer left — the one person who understood the payments service resigned in month two, and delivery stalled while others ramped.
12. The team was under-staffed at crunch — two people were out sick during the final push, and the remaining engineers cut corners on testing to hit the date.
13. On-call burnout set in — a noisy pager during the stabilization phase exhausted the team, quality dropped, and a second incident followed the first.
14. Knowledge stayed in one head — the original architect never wrote it down, and when they went on leave the team couldn't safely change the system.
15. Hiring lagged the ramp — the planned headcount arrived a quarter late, so the roadmap slipped because the work was scoped for a bigger team.
16. Too much context-switching — the team was split across three priorities, none got full focus, and the most important one shipped half-finished.
17. The new hires never fully onboarded — documentation was stale, so ramp took twice as long as planned and senior engineers were pulled into hand-holding.

## Market / Customer

18. We built for the wrong audience — the feature solved a problem power users had, but the actual buyers were first-timers who never reached it.
19. Demand softened — the market shifted between planning and launch, and the segment we bet on tightened budgets, so adoption came in far below forecast.
20. A competitor shipped first — they launched a near-identical feature a month earlier, captured the press cycle, and our launch landed as "me too".
21. Pricing was rejected — customers liked the product but balked at the tier we set, and the conversion rate never justified the build cost.
22. Migration drove churn — forcing existing customers onto the new flow broke their workflows, and a wave of them cancelled rather than relearn.
23. Onboarding was too complex — users hit a six-step setup before any value, most dropped off at step three, and activation cratered.
24. The differentiator didn't land — what we thought was a unique selling point read as a minor tweak to customers, and nobody switched for it.

## External / Vendor

25. The vendor missed the SLA — their primary maintainer left in month two, integration deadlines slipped repeatedly, and our timeline was hostage to theirs.
26. An API breaking change blindsided us — the provider deprecated the endpoint we depended on with two weeks' notice, forcing an emergency rewrite.
27. A third-party outage hit during launch — the payment processor went down on launch day, transactions failed, and our reliability story was set by their incident.
28. The contract was renegotiated — the vendor raised prices at renewal once we were locked in, and the unit economics no longer worked.
29. A licence dispute surfaced — a dependency's licence was incompatible with our commercial use, legal flagged it late, and we had to rip it out.
30. The vendor was acquired — the acquirer sunset the product we relied on, and we were given a 6-month migration window we hadn't planned for.

## Governance / Process

31. Approval bottlenecked — the one sign-off authority was on leave for the decision window, and the launch slipped two weeks waiting for a signature.
32. Legal/compliance flagged it late — a privacy review that should have happened in design surfaced a blocker the week before GA, and the feature was held.
33. The design partner pulled out — the flagship customer we co-designed with deprioritized the project, and we lost both the validation and the reference.
34. Scope expanded without budget — stakeholders kept adding requirements but the timeline and headcount were never adjusted, so something had to give.
35. A change freeze collided with launch — the company-wide end-of-quarter code freeze landed on our release date, and we couldn't ship the fix we needed.
36. The security review gated the release — it was scheduled too late, found issues that needed real rework, and the go-live date was missed.

## Financial

37. Runway compressed — a slower-than-planned revenue ramp shortened the cash window, and the project was cut before it could prove itself.
38. An FX shift hurt margins — costs were in one currency and revenue in another, the rate moved against us, and the deal economics inverted.
39. The cloud bill spiked — usage-based costs scaled faster than revenue, the gross margin went underwater, and finance forced an architecture change.
40. Unit economics never worked — customer acquisition cost stayed above lifetime value, so every new customer lost money and growth made it worse.
41. Discounting eroded the margin — sales closed deals on heavy discounts to hit targets, and the realized price fell below the level the model assumed.
42. The fundraise slipped — the next round took longer to close than planned, spending was frozen mid-project, and momentum was lost.

## Communication

43. Stakeholders were unaligned on the success metric — engineering optimized for latency while the business expected revenue, and "success" was disputed at review.
44. Launch comms confused users — the announcement over-promised, expectations outran the actual feature, and the support queue filled with disappointment.
45. Internal docs decayed — the runbook was out of date during the incident, the on-call engineer followed stale steps, and recovery took far longer.
46. The support team was unprepared — they learned about the launch the day of, had no answers for the influx, and CSAT dropped during the rollout.
47. Executive expectations diverged — leadership thought they were getting a finished product while the team had scoped an MVP, and the demo landed badly.
48. Status reporting hid the slip — green dashboards masked a growing delay, the problem was escalated only when it was too late to recover the date.
49. A cross-team handoff dropped — the dependency on the platform team was assumed but never confirmed, their work wasn't prioritized, and we blocked on it.
50. The rollout plan wasn't communicated — regional teams didn't know the launch sequence, two markets went live early, and messaging contradicted itself.
