# Decision Log

<!-- Decisions and human-gate approvals. Newest on top. -->

## Approval: Final QA Gate — v1.3.0 Features

- Date / By / Stage: 2026-06-21 / Human / Final QA Gate
- Status: APPROVED
- Decision: v1.3.0 features T6 (F3 footer), T7 (`/5bot-status`), T8 (F1 reminder) approved with no bugs. Execute T9 release.
- T9 plan: bump `plugin.json` + `marketplace.json` → 1.3.0; update README + root CLAUDE.md + MEMORY; push to a **staging branch**; install-verify (incl. Cowork); then merge to `main` with explicit push authorization.
- Note: QA was a static review (Markdown plugin, not runnable here); runtime verification happens at the staging install.
- Consequences: local release edits applied now; GitHub push held for explicit human authorization.

## Approval: Architect Gate — v1.3.0 Build Plan

- Date / By / Stage: 2026-06-21 / Human / Architect Gate
- Status: APPROVED
- Decision: v1.3.0 build plan + tickets approved. Build **sequentially T6 → T7 → T8 → T9**, edits canonical in the plugin and synced to `_framework/5bot/`. DRY: F1 block + F3 footer/verdict map live once in the `five-bot` skill.
- Added to plan (human input): T9 includes a **staging ("hidden") branch** — install v1.3.0 from a non-default branch to verify the new command/behaviors (incl. Cowork) before merging to `main`/the live published plugin.
- Execution note: implemented inline (sequential shared-file edits; subagent limit resets 1:30pm local, irrelevant to sequential work).
- Consequences: Proceed to Dev on T6 (F3 footer); Dev→QA loops without a human gate until the final QA/release gate.

## Approval: UX Gate — v1.3.0 Design

- Date / By / Stage: 2026-06-21 / Human / UX Gate
- Status: APPROVED
- Decision: `ux.md` v1.3.0 design approved (F1 reminder, F2 `/5bot-status`, F3 footer as one consistent system). Resolved open questions:
  - **OQ-1 ("long" mechanism):** **Bot self-judgment** (4 drift signals + once-per-session/seam-only guardrails). Confirmed.
  - **OQ-2 (`/5bot-status` content):** **Keep it brief** — stage, active ticket, last decision, open questions, one next command, freshness footer. Do NOT add Known Risks.
- Also locked in design: handoff→gate F1 suppression (no double-fire); F1 prints above F3 so NEXT stays last; plain-Markdown only (Cowork-safe).
- Note: synthesize/critique workflow agents failed on a session limit; synthesis + adversarial critique were performed inline by Claude instead.
- Consequences: Proceed to Architect — turn ux.md into tickets (F1 persona/command edits, F2 new `/5bot-status` command, F3 footer edits), with T5 stage-lock research as its own track.

## Approval: Product Gate — v1.3.0 Scope

- Date / By / Stage: 2026-06-21 / Human / Product Gate
- Status: APPROVED
- Decision: v1.3.0 scope (F1 context reminder · F2 `/5bot-status` · F3 next-command footer) approved. Resolved open questions:
  - **Q1 (T5 bundling):** APPROVED Product Bot recommendation — **bundle the thinking, split the shipping.** One v1.3.0 "Pipeline Awareness & Guardrails" design initiative covers F1–F3 *and* the T5 stage-lock research, designed together. Delivery splits: **F1–F3 ship as v1.3.0**; the stage-lock ships only if research greenlights it (v1.3.1 / v1.4.0, its own gate). Shippable features are NOT held hostage to the research outcome.
  - **Q2 (Cowork):** Human has a Cowork environment. **Verification is a QA-stage task**, with Claude walking the human through the test. Not a Product blocker.
  - **Q3 ("long" threshold):** **Bot self-judgment** — long-running personas (Dev, QA) judge from session length/activity and nudge. No token API needed.
  - **Q4 (`/5bot-status`):** **New dedicated read-only command.**
- Consequences: Proceed to UX to design F1 reminder copy + placement, F2 `/5bot-status` output, F3 footer format, and note where the T5 stage-lock would hook in.

## PROPOSED: v1.3.0 — Context Discipline & Workflow Smoothness

- Date / By / Stage: 2026-06-21 / Product Bot / Product
- Status: PROPOSED (awaiting human gate before UX)
- Scope proposed:
  - **F1 — Context-health reminder:** at `/handoff` + `/gate` and in Dev/QA personas, nudge to `/compact` or start fresh, reassuring that state lives on disk (safe to compact). Prompted heuristic, NOT auto-detection.
  - **F2 — `/5bot-status`:** read-only command printing current stage, active ticket, last decision, and the next command — the re-orientation entry point after compaction/cold start.
  - **F3 — "Next command" footer:** `/handoff` and `/gate` always name the exact next command.
- Out of scope this round: automated context measurement, hard stage locks (stays T5 research), Cowork re-engineering (verify/document only).
- Open questions for human: version/bundling with T5; Cowork test access; how to judge "long"; `/5bot-status` as command vs flag.
- Consequences if approved: proceed to UX to design the reminder copy/placement and `/5bot-status` output, then Architect for tickets.

- Date / By / Stage: 2026-06-21 / Human + Claude / Release & Documentation
- Status: DONE
- What happened:
  - v1.2.1 pushed to https://github.com/AngryMunky/5bot-plugin and tagged `v1.2.1` (commit `2af8572`)
  - five-bot/README.md enhanced with plugin overview, Key Concepts glossary, bot hierarchy diagram, command reference (commit `a28638c`)
  - Root README.md enhanced as marketplace landing page (commit `1e209a7`)
- Human direction: Push directly to `main` (explicitly authorized); elaborate jargon (canon, handoff, gate, etc.) so first-time readers understand the context.
- Consequences: Published plugin and documentation are in sync at v1.2.1. No open work; T4 (v1.2.2) and T5 (v1.3.0) remain on the roadmap.

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
