---
description: Produce a human-approval summary and record the decision.
---
Produce a human-approval summary for the current stage (see the `five-bot` skill) and stop until I respond.

Print:
- Stage
- Artifacts to review
- Summary
- Key decisions
- Open questions
- Risks

Then ask me to reply with one of: APPROVED | APPROVED WITH CHANGES | REVISE | REJECT | DEFER | NEEDS CLARIFICATION.

When I respond, record it as a Decision / Approval block in `decisions.md` (status, by, stage, any required changes) and update `project-state.md`. Do not proceed to the next role until I respond.
