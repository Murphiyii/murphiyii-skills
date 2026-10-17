# Variants & Related Techniques — when to swap

The premortem isn't always the right tool. Use this table to redirect the user when
another technique fits better.
| Technique | Core idea | Pick it when… |
|-----------|-----------|----------------|
| **Premortem (Klein)** | The inside team imagines failure as a *fait accompli* and works backward. | You want fast, cheap, team-owned risk surfacing on a near-final plan. *(This skill's default.)* |
| **Red Team** | An independent adversarial group attacks the plan. | Stakes are very high, the plan is mature, and you can afford a parallel team and time. |
| **Devil's Advocate** | One assigned dissenter challenges the consensus. | You want a quick gut-check; lower coverage than a premortem, but better than nothing. |
| **Pre-hoc analysis** | Academic name for the same idea (prospective hindsight). | Synonymous with premortem in the research literature — not a different technique. |
| **Murder Board** | A panel grills a presenter to stress-test a proposal. | You're vetting an *individual* pitch / proposal defense, not a collective plan. |
| **FMEA** | Quantitative component-level scoring (Severity × Occurrence × Detection = RPN). | Engineered/manufactured systems needing exhaustive, auditable rigor. |
| **SWOT** | Strategic position framing (Strengths/Weaknesses/Opportunities/Threats). | Use *before* the plan exists — not a substitute for risk surfacing on a concrete plan. |
| **Atlassian variant** | Half the team brainstorms success, half failure, then they cross-examine. | Team morale needs the balance and you accept some loss of focus on failure modes. |

**Rule of thumb:** if the user has a concrete, near-final plan and a team that can
change it → premortem. If they want an *outside* adversarial attack → Red Team. If
the plan doesn't exist yet → SWOT or a design-doc review first (and the premortem
pre-flight check should refuse).
