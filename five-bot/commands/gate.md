---
description: Produce a human-approval summary and record the decision.
---
Produce a human-approval summary for the current stage (see the `five-bot` skill) and stop until I respond.

**Human gate role:** Stop here and wait for human decision. Show the stage summary, key decisions, and risks. Record the human's verdict (APPROVED, NEEDS CHANGES, REVISE, REJECT, DEFER) in decisions.md. Update project-state.md accordingly.

Print:
- Stage
- Artifacts to review
- Summary
- Key decisions
- Open questions
- Risks

Then end with the **F3 waiting footer** (see the `five-bot` skill) — name NO next command, just stop and list the options:

   ---
   **⏸ AWAITING YOUR CALL** — reply with one of:
   `APPROVED` · `APPROVED WITH CHANGES` · `REVISE` · `REJECT` · `DEFER` · `NEEDS CLARIFICATION`
   Nothing proceeds until you reply. State is saved in project-state.md / decisions.md.

If the session is long (F1 heuristic in the `five-bot` skill) **and** `/handoff` did not already show the reminder in this transition, print the terse context-health reminder once, just above the waiting footer.

When I respond, record it as a Decision / Approval block in `decisions.md` (status, by, stage, any required changes) and update `project-state.md`. Then end with the **F3 resolved footer** naming the next command for my verdict (see the skill's verdict map). Do not proceed to the next role until I respond.
