# Block C: Mitigation Action Table

Fill during workshop step 6 for the **top-ranked failure modes** (top 5 in `analyze` mode). Every
column must be filled: owner may be a role, review date may be relative, but no
empty cells.

| Failure mode      | Mitigation action | Owner | Early-warning signal | Trigger to act | Review date |
|-------------------|-------------------|-------|----------------------|----------------|-------------|
| Vendor X delays   | Negotiate fallback vendor by D+30; weekly status call | @alice | 2 consecutive missed status updates | Switch to fallback if slip > 2 weeks | YYYY-MM-DD |
|                   |                   |       |                      |                |             |
|                   |                   |       |                      |                |             |
|                   |                   |       |                      |                |             |
|                   |                   |       |                      |                |             |

- **Owner**: a name, or a role slot (`@tech-lead`, `@launch-PM`) if unknown.
- **Early-warning signal**: the observable that tells you the failure mode is materializing.
- **Trigger to act**: the threshold that flips you from watching to acting.
- **Review date**: absolute if known, else relative to the horizon
  (e.g. "30 days before launch").
