# Decision Log

<!-- Decisions and human-gate approvals. Newest on top. -->

## Approval: QA Gate — v2.2.0 T2 git-awareness line

- Date / By / Stage: 2026-06-29 / Human / QA Gate
- Status: APPROVED
- Decision: v2.2.0 T2 (read-only git-awareness line in `/5bot-status`) ACCEPTED. Cleared for release as v2.2.0.
- Notes carried: ahead/behind = "as of last fetch" (labeled, accepted); runtime cross-surface smoke test recommended at staging before main.
- Release steps: bump `plugin.json` + `marketplace.json` 2.1.0→2.2.0; update READMEs / root `CLAUDE.md` / MEMORY; sync working copy → publish clone; **dual-push to both repos** (private `5bot-plugin` + public `5bot`); tag `v2.2.0` (per `PUBLISH.md`). **Push to public + `main` requires explicit human authorization** (not implied by this gate).

## Approval: UX Gate — v2.2.0 (re-scoped) git-awareness line

- Date / By / Stage: 2026-06-29 / Human / UX Gate
- Status: APPROVED
- Decision: UX design for the read-only git line in `/5bot-status` approved — placed in the **freshness footer**, format `git: <branch> · <sync, "last fetch"> · <clean|uncommitted changes>`, informational only, silent no-op when not a git repo.
- Detail: **always-show when in a git repo** (consistent trust indicator; avoids the "silence = fine-or-unchecked?" ambiguity), rendered tersely when all-good (`git: <branch> · in sync · clean`) and fuller when noteworthy. Confirmed by the human after asking for a recommendation.
- Architect gate: per the 2-gate default, **skipped** for this small feature (human chose go-straight-to-Dev). OQ-3 (exact no-network commands + edge handling) resolved inline by the Architect.
- Consequences: Architect specs the git commands → Dev implements in `commands/5bot-status.md` → QA → QA gate → release v2.2.0.

## Approval: Product Gate — v2.2.0 (re-scoped) Sync-Awareness via read-only git status

- Date / By / Stage: 2026-06-29 / Human / Product Gate
- Status: APPROVED
- Decision: v2.2.0 re-scoped feature APPROVED — a **read-only git-awareness line** in `/5bot-status` (branch · ahead/behind vs upstream "as of last fetch" · uncommitted-changes flag); silent no-op when not a git repo / no upstream / git absent.
- OQ-1 (scope ruling) RESOLVED: read-only local git-awareness is **acceptable local tooling, NOT** the out-of-scope "External integrations (GitHub/Jira/Slack)." Decider: it never fetches/networks/uses credentials/mutates — it reads local refs + `git status` only (same category as reading on-disk Markdown). The API-usage approach is fully retired.
- Carried: OQ-2 (line content + placement → UX); OQ-3 (cross-platform no-network commands + detached/no-upstream behavior → Architect); OQ-4 (`/5bot-init` nudge → deferred).
- Consequences: proceed to `/ux`. Architect gate optional (2-gate default) — small, well-scoped feature.

## Decision: QA Gate — v2.2.0 T1 → REVISE (pivot off the API-usage signal)

- Date / By / Stage: 2026-06-29 / Human / QA Gate
- Status: REVISE
- Context: T1 implemented the API hourly-usage line in `/5bot-status` and passed all acceptance criteria, but QA escalated a feasibility flaw — the model has no reliable access to the user's API usage %, so the line would almost always silently omit. The feature's premise ("Claude knows its own usage") is incorrect.
- Decision: **REVISE — pivot the v2.2.0 feature off API-usage % to a *knowable* signal.** The API-usage approach is rejected; the implemented Usage line in `commands/5bot-status.md` is **reverted**. The *intent* (a read-only "am I in sync / can I keep working?" awareness line in `/5bot-status`) stands.
- Recommended pivot: **git-awareness** (read-only `git status` / behind-origin line) — genuinely knowable (the command can run git), and it solves the real stale-copy divergence pain hit this session. Context-window-pressure was considered and rejected: same unknowability flaw + overlaps the existing F1 heuristic.
- Open scope question for Product: does a read-only local git line count as the approved out-of-scope "External integrations (GitHub/Jira/Slack)", or is it acceptable local-tooling? (Already flagged in project-state Open Follow-ups.)
- Consequences: re-scope via `/product` (signal choice + the integrations scope question) → UX → Architect (gate optional under the 2-gate default) → Dev → QA → gate → release v2.2.0. The Architect rationale "Claude knows its own usage" is marked inaccurate in the record.

## Approval: Architect Gate — v2.2.0 Usage-Aware Status Reporting

- Date / By / Stage: 2026-06-26 / Human / Architect Gate
- Status: APPROVED
- Decision: v2.2.0 architecture approved. Resolved all three blocking OQs from UX gate: (1) API feasibility → query via Claude context + graceful fallback if unavailable, (2) Reset time format → UTC (HH:MM UTC), (3) Placement in `/5bot-status` → after Active ticket, before Last decision. Ticket T1 defined in dev-qa.md with clear acceptance criteria (query mechanism, format, placement, silent fallback behavior, no scope expansion).
- Architecture quality: Minimal, lean, no new dependencies or state files. Fits existing plugin design cleanly. Low-risk Markdown edit.
- Consequences: Proceed to `/dev T1` to implement the API usage query + display in `/5bot-status`. Then `/qa`, gate, release as v2.2.0. Post-Dev verification questions (surface reliability, timeout behavior, reset time accuracy) are QA scope.

## Approval: UX Gate — v2.2.0 Usage-Aware Status Reporting

- Date / By / Stage: 2026-06-26 / Human / UX Gate
- Status: APPROVED
- Decision: v2.2.0 UX design approved. Read-only API usage line for `/5bot-status` showing hourly % + reset time. Silent fallback if usage data unavailable (no error, no placeholder). Three blocking OQs for Architect: (1) API feasibility (can Claude Code expose usage?), (2) Reset time format (UTC vs relative?), (3) Placement in S1 (after Active ticket or at end?).
- Design quality: Lean, scope-appropriate (one line, zero workflow disruption), graceful fallback, neutral tone. Directly addresses soft-preference request without scope creep.
- Consequences: Proceed to `/architect` to confirm API feasibility and resolve format/placement decisions. Then `/dev` (light ticket for implementation), `/qa`, gate, release as v2.2.0.

## Approval: Product Gate — v2.2.0 Usage-Aware Status Reporting

- Date / By / Stage: 2026-06-26 / Human / Product Gate
- Status: APPROVED (revised scope)
- Decision: v2.2.0 feature **approved with scope clarification**. Original proposal (pause/resume routines that block workflow when usage > 75%) was operationally risky and disruptive. **Revised scope:** lightweight, read-only usage status line in `/5bot-status` showing current hourly API usage (e.g., "API hourly usage: 76% (resets at 15:00 UTC)"). Zero impact on workflow; informational only; users decide what to do with the info. No pause logic, no blocking, no automation, graceful fallback if usage query unavailable.
- Rationale: Soft personal preference (not hard org constraint). Read-only status gives awareness without disruption. Aligns with 5bot's "know your state" philosophy and the prior git-awareness proposal.
- Version: v2.2.0 (MINOR; informational feature, additive).
- Consequences: Proceed to `/ux` to design status line format and output text. Then `/architect` (light) for feasibility, `/dev`, `/qa`, gate, release.

## Framework Decision: Gate strategy (default two gates, not three)

- Date / By: 2026-06-23 / Human (developer feedback on workflow velocity)
- Decision: Changed the default gate pattern from **three mandatory gates** (Product+UX → Architect → QA) to **two required gates** (Product+UX, QA) with **Architect gate optional**. Architect gate is now skipped by default for features under ~5 tickets or in established codebases (where the QA gate still catches gaps); included for greenfield work or when explicit Architect sign-off is desired.
- Rationale: gates are checkpoints for *human decisions* (scope/direction or quality). The Architect gate creates overhead for smaller work without changing the decision points that matter (what we build, whether we accept it). QA still reviews the architectural choices, so skipping the explicit gate doesn't lose verification — it trades a pause for velocity.
- Impact: affects _framework/5bot/rules.md (v1.2.0) and the README guidance. All future projects created via `/5bot-init` inherit this default. Existing projects (including the 5bot plugin itself) can optionally adopt it for future work.
- Backward compatible: teams that prefer three gates can easily reinstate the Architect gate by running `/architect` → `/gate` — it's just not the default nudge anymore.

## Approval: QA Gate — T14 (v2.1.0 release)

- Date / By / Stage: 2026-06-23 / Human / QA Gate (release authorization)
- Status: APPROVED
- Decision: v2.1.0 release approved and the dual-push authorized. Per the gate's three decisions, a plain APPROVED resolves them as: (1) **delete** the orphaned root `.claude-plugin/plugin.json` (stale @ v1.2.1, off the install path); (2) **publish as-is**, with the machine-specific Temp clone path sanitized out of `PUBLISH.md` before pushing; (3) **re-stamp** the `product.md`/`architecture.md`/`ux.md` headers to v2.1.0 for consistency.
- Release contents: `plugin.json` 2.0.0 → 2.1.0 + matching `marketplace.json` `5bot` entry; READMEs (plugin + root landing) + stage headers re-stamped; v2.1.0 feature = optional Claude Design design-reference. Pushed from the canonical clone as **Angry Munky** (noreply), `git push origin main` (dual-push → private `5bot-plugin` + public `5bot`), tag `v2.1.0` to both — per `PUBLISH.md`.
- Consequences: v2.1.0 ships. All v2.1.0 tickets (T10–T14) complete. Open follow-up remains: the recommended `/ux` re-sync of `ux.md` D2–D5 design-era copy. T15 (connector import) stays deferred.

## Approval: QA Gate — T13 (docs / templates / `_framework` sync)

- Date / By / Stage: 2026-06-23 / Human / QA Gate
- Status: APPROVED
- Decision: T13 approved (QA verdict was APPROVED WITH NOTES; human accepted as-is). All four ACs met — README "Optional: a Claude Design reference" section (zip workflow + `design/` convention + "ux.md canon; native Claude Design only, Figma out"); template comment (v0.1.1); `_framework` personas synced (ux.md/dev.md → v1.2.0); roster unchanged (9 commands, no phantom). The single recorded deferred fix (project `ux.md` D4 import-method → `fallback-zip`) landed. Reviewed via an independent 3-lens adversarial pass + a scope-adjudication agent (confirmed: approve with notes, do not bounce).
- Notes carried as follow-ups (non-blocking): (1) **recommended `/ux` re-sync** of project `ux.md` D2–D5 design-era copy to the as-built SKILL.md schema (D4 missing `Original zip:` + stale flat `Local artifact:` path; D2/D3/D5 still frame the connector as a live branch vs deferred-to-T15) — the shipped contract (SKILL.md) is correct, only the design doc lags; (2) optional polish to README shared-files glossary (~line 227) + `commands/ux.md` line 12 double-story. Tracked in `project-state.md` → Open Follow-ups.
- Also this session (human-approved bookkeeping): pruned `project-state.md` (removed a stale duplicate Active-Ticket block; collapsed resolved Open Questions to a pointer) to cut always-loaded canon size.
- Consequences: All v2.1.0 build tickets except release are now APPROVED (T10/T11/T12/T13). Proceed to **`/dev T14`** — the release (version bump 2.0.0→2.1.0, mirror to repo-root marketplace.json, root docs, **dual-push to both repos + tag — requires explicit human authorization** for the public + `main` push).

## Approval: QA Gate — T12 (`/dev` consumes the Design Reference)

- Date / By / Stage: 2026-06-23 / Human / QA Gate
- Status: APPROVED
- Decision: T12 (`/dev` consume trigger) approved. Reviewed via adversarial 4-lens QA workflow (acceptance-criteria, cross-file consistency, guardrails/anti-drift, edge-cases; each finding verified by an independent refuter). Deliverable met all 4 ACs on first pass; review caught one real cross-file inconsistency (T12 AC text wildcard `*.dc.html` vs spec/impl `<Name>.dc.html`) + one clarity nit (implicit no-op), both fixed via a single auto-bounce; one finding refuted; one nit deferred to T13.
- Consequences: All three v2.1.0 code tickets (T10/T11/T12) APPROVED. Proceed to `/dev T13` (docs / templates / `_framework` sync). T13 carries the deferred item: tighten `ux.md` D4 import-method example to `fallback-zip`.

## Approval: QA Gate — T11 (`/ux` design-reference step)

- Date / By / Stage: 2026-06-23 / Human / QA Gate
- Status: APPROVED
- Decision: T11 (`/ux` design-reference step) approved. QA found zero bugs, all 7 acceptance criteria met. Optional trigger added to `commands/ux.md`; three input paths (URL, "Send to local coding agent" prompt, zip path); references SKILL.md procedure (no duplication); silent no-op when no design (zero friction); URL-only fallback documented.
- Consequences: Proceed to `/dev T12` (`/dev` consumes the Design Reference). T12 depends only on T10.

## Approval: QA Gate — T10 (Design Reference procedure)

- Date / By / Stage: 2026-06-23 / Human / QA Gate
- Status: APPROVED
- Decision: T10 (Skill: Design Reference procedure) approved. QA found zero bugs, all acceptance criteria met, no changes needed. Procedure is canonical, self-contained, ready for T11/T12 to reference. DRY placement confirmed.
- Consequences: Proceed to `/dev T11` (`/ux` design-reference step). T12 parallel-ok after T11.

## Approval: Architect Gate — v2.1.0 Claude Design Integration

- Date / By / Stage: 2026-06-23 / Human / Architect Gate
- Status: APPROVED
- Decision: v2.1.0 Architect build plan approved. **Storage (OQ-7):** `design/<slug>/` for extracted contents + `design/<slug>.zip` for pristine source; primary artifact = `.dc.html` linked from `ux.md`. **On-first-use (OQ-8):** create `design/` when `/ux` first imports a zip (not at `/5bot-init`). **Cross-platform extract (OQ-9):** skill provides both PowerShell + unix commands; agent picks per OS. Connector deferred (T15, post-MVP, OQ-6 non-blocking).
- Build sequence: **T10 → { T11, T12 parallel } → T13 → T14**. T15 stays deferred.
- Consequences: Proceed to Dev on T10 (skill: Design Reference procedure).

## Approval: UX Gate — v2.1.0 Claude Design design-reference step

- Date / By / Stage: 2026-06-22 / Human / UX Gate
- Status: APPROVED WITH CHANGES
- Decision: UX design (Flow D, screens D1–D5) approved. **Change/direction:** the **`.zip` download from Claude Design is the first-class MVP path** (*"I can download the design as a `.zip` — go with that"*). The `claude_design` connector import is a **progressive enhancement deferred past MVP**, so the MVP has **no dependency on the connector**.
- Resolved OQs: OQ-2 = zip-first (fallback-first); OQ-3 = committed `.zip` (+ extracted `*.dc.html`) is the artifact, linked from a `ux.md` Design Reference block (location ~`design/`, Architect finalizes); OQ-5 = UX-only capture.
- To Architect: OQ-6 (connector connect/auth path) now **NON-blocking for MVP**; OQ-7 (artifact location/format).
- Consequences: proceed to `/architect` to scope v2.1.0 tickets around the zip path (export → drop `.zip` in repo → `/ux` records a Design Reference block → Dev builds against it); document the connector-import path as a later enhancement.

## Approval: Product Gate — v2.1.0 Claude Design Integration (optional)

- Date / By / Stage: 2026-06-22 / Human / Product Gate
- Status: APPROVED
- Decision: Bring **Claude Design integration in-scope as v2.1.0** (optional, opt-in). Claude Design (first-party / native, via the `claude_design` MCP connector) is distinct from third-party **Figma** — this **corrects a prior mis-bundling** on the Out-of-Scope list, not a reversal of the no-third-party-tools stance. Figma and other third-party design tools **stay out of scope**.
- Scope: opt-in "design-reference" step at the UX stage; import/reference a Claude Design artifact when the `claude_design` connector is available, with a connector-free fallback (zip / exported `*.dc.html` / design URL) so the pipeline never hard-blocks. `ux.md` stays source of truth. New additive feature → MINOR bump, **v2.1.0**.
- OQ-1 (blocking): RESOLVED — APPROVED.
- Carried forward: OQ-2 (connector-first vs fallback-first), OQ-3 (canonical stored form), OQ-5 (Dev-stage too vs UX-only) → resolve at UX; OQ-4 (connector availability / is `/design-login` real?) → Architect feasibility.
- Consequences: move Claude Design out of the Out-of-Scope list (now in-scope, v2.1.0); proceed to `/ux` to design the optional design-reference step.

## Decision: Rebrand install id to `5bot` → release as v2.0.0

- Date / By / Stage: 2026-06-21 / Human / Release (T9)
- Status: APPROVED
- Context: Static validation found the published install command was inconsistent — docs said `5bot@lawson-design` but the marketplace entry was named `five-bot`, so install would fail.
- Decision 1 (install name): **Rebrand the marketplace entry `five-bot` → `5bot`** so the install id is `5bot@lawson-design`. This is BREAKING (existing `five-bot@` installs orphaned → must reinstall). Chose the breaking rebrand over the non-breaking "keep five-bot" option.
- Decision 2 (version): Per the CLAUDE.md semver rule (breaking → MAJOR), the release is renumbered **v1.3.0 → v2.0.0**. Feature work (F1/F2/F3) is unchanged; dev artifacts authored as "v1.3.0" refer to this release.
- Context (repo): `AngryMunky/5bot-plugin` was flipped **public → PRIVATE** today so it lists in claude.ai's "Sync from GitHub" dialog. CLI public-install path no longer applies while private; verification likely happens after merge via re-sync of `main`.
- Follow-ups: `task_2dbc68f5` (qa.md "Test Plan" section mismatch — pre-existing, out of v2.0.0 scope).
- Consequences: re-stamp all version markers to 2.0.0; rename marketplace entry to `5bot`; merge staging → `main`, tag v2.0.0 (pending explicit authorization).

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
