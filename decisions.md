# Decision Log

<!-- Decisions and human-gate approvals. Newest on top. -->

## Approval: Final Human Gate — v1.2.1 Release

- Date / By / Stage: 2026-06-21 / Human / Final QA Gate
- Status: APPROVED
- Decision: All three v1.2.1 tickets (T1-T3) are approved by QA with zero bugs. Release v1.2.1.
- QA verdicts:
  - T1 (handoff.md warning): ✅ APPROVED
  - T2 (canon definition): ✅ APPROVED
  - T3 (rule summaries): ✅ APPROVED
- Release actions:
  - Bump plugin.json: 1.2.0 → 1.2.1
  - Update MEMORY.md and root CLAUDE.md with new version
  - Commit and tag v1.2.1
  - Mark T4 (README validation checklist) as backlog for v1.2.2
  - Mark T5 (gate enforcement research) as planning for v1.3.0
- Consequences: v1.2.1 is now released. Three clarity improvements are live. T4 and T5 move to future roadmap.

## Approval: Architecture Plan — v1.2.1 Improvements + Risk Mitigations

- Date / By / Stage: 2026-06-21 / Human / Architect Gate
- Status: APPROVED
- Decision: 5 dev tickets created for clarity improvements and risk mitigation.
  - **v1.2.1 (ship immediately):** T1-T3 (three clarity improvements: handoff warning, canon definition, rule summaries)
  - **v1.2.2 (next patch):** T4 (README validation checklist)
  - **v1.3.0 (planning):** T5 (research gate enforcement)
- Tech stack: Markdown-only edits. No code, no new files, no dependencies. Same plugin architecture.
- Implementation: All tickets are Markdown edits (templates and command files). No breaking changes.
- Assumptions:
  - Users have git for version control (not enforced, documented in README)
  - Markdown editing is acceptable (no visual UI needed for v1.2.1)
  - Anti-drift rules + human gates sufficient for now (stage lock deferred to v1.3.0)
- Open questions:
  - Should stage lock be prioritized for v1.3.0, or defer further?
  - Are there other clarity improvements not captured?
- Human guidance:
  - ✓ Execute T1-T3 **sequentially** (not parallel) to avoid merge conflicts on templates/commands
  - ✓ Proceed to Dev stage immediately
  - ✓ Proceed with T4 (v1.2.2) and T5 (v1.3.0) as planned (no changes)
  - ✓ No additional tickets at this time
- Consequences: Dev Bot will implement T1-T3 sequentially. After each ticket is approved by QA, move to next. Release v1.2.1 after all three approved. T4 and T5 are future work.

## Approval: UX Audit & Three Improvements

- Date / By / Stage: 2026-06-21 / Human / UX Gate
- Status: APPROVED
- Decision: UX audit is complete and workflow is sound. Approve three clarity improvements for v1.2.1:
  1. ✓ Fix handoff.md transient nature confusion (add ⚠️ warning)
  2. ✓ Update "canon" jargon (add definition to project-state.md)
  3. ✓ Add 2-3 sentence rule summary to 8 command files
- Human guidance on risks:
  - **Field validation:** YES, update instructions to guide users (not technical enforcement, but better docs)
  - **Technical enforcement of gates:** Explore options in Architect stage (e.g., "stage lock" in project-state.md that prevents running commands out of order). Scope: future version if feasible.
  - **Version control for plain Markdown:** Document best practices in README (users should use git; 5bot state files are git-friendly). No special tooling needed.
- Consequences: Architect Bot will create dev tickets for the three improvements (v1.2.1). Will also scope research on gate enforcement and field validation docs as future work.

## Approval: Product Audit — Current State Documentation

- Date / By / Stage: 2026-06-21 / Human / Product Gate
- Status: APPROVED
- Decision: Product audit is accurate. Five-bot is shipped, working, MVP-complete. Proceed to UX stage.
- Human guidance on open questions:
  - Q1 (Non-software domains): **DEFER** — not a priority now
  - Q2 (Visual dashboard): **DEFERRED** — only if natively in Claude Code/Cowork, no separate external tools
  - Q3 (Auto-lint Markdown via CI): **APPROVED** — yes, implement this
  - Q4 (Multi-project namespacing): **RESOLVED** — current folder-per-project model is correct
  - Q5 (Dev/QA separation): **APPROVED** — keep commands separate, no looping variant
- Consequences: UX Bot will audit command prompts, workflow order, template clarity, and user interaction experience. Improvements scoped in Architect stage.
