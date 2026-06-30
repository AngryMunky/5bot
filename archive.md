# Archive — relocated history. Not loaded by default; never summarized.
# Open a section only when you need history. (Created by /5bot-archive, 2026-06-29.)

## Stage history

### product.md — superseded sections (v1.3.0, v2.2.0 API, v2.2.0 git, v2.1.0)

# Proposed v1.3.0 — Context Discipline & Workflow Smoothness

This round addresses two human requests: (1) a context-health check that suggests `/compact` after long sessions, and (2) general workflow improvements for Claude Code and Claude Cowork.

## Problem

The 5bot workflow runs across **long, multi-stage sessions** — product audits, architecture planning, and especially Dev→QA loops that iterate many times. Two friction points emerge:

1. **Context exhaustion (already a top risk).** As a session grows, the model's context fills. Quality degrades, auto-compaction can lose nuance, and users don't know *when* to act. Critically, **5bot is uniquely safe to compact** — all canonical state lives in Markdown on disk — but users don't know this, so they either fear compacting (and degrade) or compact blindly (and feel they lost work). The workflow never tells them it's safe.

2. **Re-orientation friction.** After a compaction, a fresh session, or simply returning days later, users ask "where am I and what do I run next?" The state is on disk, but nothing surfaces it in one step. Returning to a project mid-pipeline is a cold start.

These compound: the moment a user most needs to compact (long session) is exactly when re-orientation afterward is hardest.

## User Goals (this round)

1. **Know when to compact** — get a nudge at natural breakpoints, especially after long phases.
2. **Compact without fear** — understand that 5bot state is on disk, so compaction/fresh sessions lose nothing essential.
3. **Re-orient in one step** — after compaction or a new session, immediately see current stage, active ticket, last decision, and the exact next command.
4. **Always know the next command** — never guess what to run next.
5. **Run reliably in Cowork** — the workflow behaves predictably in Claude Cowork, not just local Claude Code.

## Proposed MVP Features (v1.3.0)

**F1 — Context-health reminder (the core request).** At natural breakpoints (`/handoff` and `/gate` output, and within long-running personas like Dev and QA), the workflow surfaces a brief, concise reminder: *"This session has run long — consider `/compact` or a fresh session. Your state is saved in `project-state.md`, `decisions.md`, and the stage files, so nothing is lost; just re-run the current command to reload."* The reassurance ("safe to compact") is the differentiator, not just the nudge.

**F2 — `/5bot-status` orientation command (recommended).** A read-only command that reads `project-state.md` + latest `handoff.md` and prints: current stage, active ticket, last decision, open questions, and the single recommended next command. This is the designated "re-orient after compaction / cold start" entry point — directly pairs with F1.

**F3 — "Next command" footer on `/handoff` and `/gate` (recommended).** Every bookkeeping/gate output ends by naming the exact next command to run (e.g., "Next: run `/ux`"). Removes the "what now?" guess and reinforces the pipeline order.

## Out of Scope (Future / Recommended Later)

- **Automated context measurement.** Command Markdown cannot read the live token count, so F1 is a *prompted heuristic reminder* ("if this has run long…"), **not** an automated threshold trigger. True auto-detection would require a Claude Code hook/API and is out of scope here. **(ASSUMPTION — see open questions.)**
- **Hard stage-skip enforcement.** A *soft* warning when commands run out of order is a candidate, but hard "stage locks" remain the **T5 / gate-enforcement research** track — keep it research, not built here.
- **Cowork-specific re-engineering.** This round only verifies and documents Cowork behavior (file persistence, command availability) and adjusts guidance — not a separate Cowork codebase.
- Dashboards, VS Code extension, git hooks, third-party integrations — unchanged, still later.

## Success Criteria

- Users on long projects report fewer "lost context / degraded session" incidents.
- A returning or post-compaction user can fully re-orient with **one command** (`/5bot-status`).
- After any `/handoff` or `/gate`, the next command is unambiguous.
- Cowork behavior is verified and documented; any divergence from local Claude Code is called out for users.
- Reminder is helpful, not nagging (appears at breakpoints / after long phases only).

## Risks

1. **Reminder fatigue.** Too-frequent `/compact` nudges become noise. **Mitigation:** breakpoints + long-phase only; keep it one concise line.
2. **`/compact` behavior differs across Code vs Cowork.** The command or its effect may not be identical. **Mitigation:** verify in both; document differences; phrase the reminder tool-neutrally ("compact or start fresh").
3. **Status command shows stale info.** If `project-state.md` isn't current, `/5bot-status` misleads. **Mitigation:** it reads the canon, which `/handoff` (rule 10) keeps current; F3 reinforces this.
4. **Scope bleed into T5.** F-set could drift toward building enforcement. **Mitigation:** soft-warning only; hard locks stay in T5 research.

## Open Questions — RESOLVED at Product gate (2026-06-21)

1. **Version & bundling:** ✓ **RESOLVED.** Ship as **v1.3.0**. Bundle T5 into the *design* initiative (one "Pipeline Awareness & Guardrails" effort), but **split delivery**: F1–F3 ship as v1.3.0; the T5 stage-lock ships later on its own gate (v1.3.1 / v1.4.0) only if research greenlights it.
2. **Cowork verification:** ✓ **RESOLVED.** Human has a Cowork environment. Verification is a **QA-stage task**, with Claude walking the human through the test. Not a Product blocker.
3. **Reminder aggressiveness:** ✓ **RESOLVED.** **Bot self-judgment** — long-running personas (Dev, QA) judge from session length/activity and nudge. No token API needed.
4. **`/5bot-status`:** ✓ **RESOLVED.** A **new dedicated read-only command.**

---

**Deferred backlog (unchanged, post-v1.3.0):** extensibility / custom roles, third-party integrations (GitHub, Jira, Slack), tooling (VS Code extension, git hooks), visual design tools.

---

# PROPOSED v2.2.0 — Usage-Aware Workflows (API quota monitoring)

> ⛔ **SUPERSEDED (2026-06-29).** The lightweight version of this (a read-only usage line) was built as T1, then **reverted at the QA gate (REVISE)** — the model can't reliably read the user's API usage %, so the line would almost always silently omit. v2.2.0 was re-scoped to **git-based sync-awareness** (a signal the command CAN read). See "PROPOSED v2.2.0 (re-scoped)" below. This section is retained for history. (`decisions.md`, 2026-06-29.)

Human request: "Have the 5bot routines check the usage to see what is available and make an estimation as to when the routine should stop and pick back up again? Perhaps when hourly usage is above 75%?"

## Problem

During long development sessions, users can exceed API usage quotas (hourly, daily, or organizational limits) without realizing it until they hit a rate-limit error mid-workflow. The result is workflow interruption, manual quota checks, or surprise cost overages. Users want to know *before* execution whether it's safe to proceed, and *when* it will be safe to resume.

## Target Users

Existing 5bot users concerned with:
- **Cost management:** preventing surprise API overspend during intensive dev sessions.
- **Quota avoidance:** working within organizational or personal hourly/daily usage caps.
- **Long-running projects:** multi-day workflows where quota reset timing matters.

## User Goals

1. Know current API usage without leaving the workflow or checking the dashboard.
2. Get a clear signal to pause before hitting a hard rate limit.
3. Know when to resume (quota reset time).
4. Never lose work state; pause is reversible.

## MVP Scope (if approved)

**Advisory usage check** at the start of each bot command (Product, UX, Architect, Dev, QA):
- Read the user's current hourly API usage (from Anthropic API or host environment).
- If usage ≥ threshold (default 75%), log a soft nudge: *"API hourly usage at 76%; recommend pausing until quota resets at 15:00 UTC. Continue anyway?"*
- **No blocking:** Default to continue — this is awareness, not enforcement.
- **No automation:** Human decides when to run the next command; no auto-queue or background resumption.
- **Configuration:** User sets threshold in `project-state.md` or `.env` (no built-in default).

## Out of Scope (This Round)

- Automated pause/resume scheduling.
- Rate-limit queuing or background retry.
- Cost prediction or analytics dashboards.
- Rate-limit vs. quota distinction; multi-user org quotas.
- Hard quota enforcement (blocks workflow).

## Key Open Questions (Blocking)

**OQ-1 (Scope / External Integration):** Does querying the Anthropic API for usage data count as out-of-scope "External integrations (GitHub, Jira, Slack)", or is it acceptable as first-party Anthropic carve-out (like Claude Design)?

**OQ-2 (Quota definition):** Is the 75% threshold:
- Personal hourly quota (usage this calendar hour)?
- API key rate-limit (tokens/min on a specific key)?
- Organization / team quota?
- Monthly cost ceiling?

**OQ-3 (Pause semantics):** Soft nudge (advisory) or hard block (refuse to proceed)?

**OQ-4 (Credentials):** How does 5bot query usage? Requires:
- User provides Anthropic API key (environment variable)?
- Host (Claude Code / Cowork) exposes usage data?
- Or dashboard-only (not automatable)?

**OQ-5 (Resume trigger):** Manual (user re-runs the command), time-based (auto-resume at quota reset), or polling?

## Risks

1. **Scope creep into billing/analytics.** Quota nudges naturally lead to dashboards and cost predictions. Recommend strict advisory-only scope.
2. **API credentials exposure.** Storing API keys in project files risks leaks. Must use environment variables / host integration.
3. **Quota reset timing confusion.** Claude API uses rolling hourly windows, not calendar hours. Users may pause at wrong time.
4. **Offline or API outages block workflow.** Operational coupling to external service.
5. **False alarms mid-work.** Batch `/dev` or `/qa` runs may spike usage mid-execution, pausing mid-work. Recovery is awkward.

## Assumptions

- **ASSUMPTION:** User has visibility into Claude API usage (dashboard or API endpoint).
- **ASSUMPTION:** 75% refers to personal hourly quota.
- **ASSUMPTION:** Soft nudge is preferred over hard blocks.
- **ASSUMPTION:** Resume is manual (user re-runs command), not automated.

## Recommendation

**Status: PROPOSED.** Viable if the gate resolves three questions:
1. In-scope (first-party carve-out) or out-of-scope (external integration)?
2. Soft nudge or hard block?
3. Version slot (v2.2.0, v3.0.0, or backlog)?

If approved at the gate, route: `/product` → `/architect` → tickets → `/dev` → `/qa` → gate → release.

If rejected/deferred, record in `decisions.md` and mark as OUT OF SCOPE or DEFERRED.

---

# PROPOSED v2.2.0 (re-scoped 2026-06-29) — Sync-Awareness via read-only git status

> Supersedes the API-quota approach above (and its as-built usage line, reverted at the QA gate — `decisions.md`). The model can't reliably read API usage %, so v2.2.0 pivots to a signal the command CAN read: local **git** state. Human chose git-awareness over context-window pressure at the 2026-06-29 signal decision.

## Problem

Working across machines, sessions, and surfaces (local Claude Code ↔ Cowork ↔ web), a user can resume on a **stale local copy** and unknowingly build on out-of-date state — or carry **uncommitted changes** they forget to save. 5bot surfaces nothing about this. (Demonstrated live: this very session built on a stale snapshot until disk reads exposed the divergence — exactly this failure mode.) Users want a glance-level "am I in sync?" signal before they act.

## Target Users

Existing 5bot users, especially those who work the same project across multiple machines/surfaces (local + Cowork), where local copies drift; and anyone who wants a heads-up that they have uncommitted work before compacting or switching context.

## User Goals

1. See at a glance whether the local copy is behind/ahead of its remote and whether there are uncommitted changes — without leaving the workflow.
2. Catch divergence **before** building on stale state.
3. Zero disruption: read-only, informational; never touches the repo.

## Scope ruling — is this an "external integration"? (the key gate question)

**Product ruling (confirm at gate): NO — this is local tooling, not an external integration.** The out-of-scope item "External integrations (GitHub, Jira, Slack)" means networked third-party *services* (creating GitHub issues, Jira sync, Slack posts) with remote APIs and credentials. Reading local `git` state is reading the user's own working copy via a tool already on their machine — the same category as reading the on-disk Markdown files 5bot already reads. **To keep it unambiguously local:** the feature does **NOT** run `git fetch` (no network, no credentials, no remote API). Ahead/behind is computed from already-fetched refs (`@{u}...HEAD`) and labeled "as of last fetch"; dirty state from `git status`. No mutation, ever. So it sits clearly inside *acceptable local tooling*, not the prohibited integrations. (Contrast: the rejected API-usage path needed a remote/credentialed signal — git-awareness deliberately avoids that.)

## MVP Scope (if approved)

A **read-only git line** in `/5bot-status` (only there — not on every command), best-effort:
- Branch + ahead/behind vs upstream (from last fetch, labeled) + a dirty flag. e.g. `git: main · 2 behind origin (as of last fetch) · uncommitted changes`.
- **Silent no-op** when: not a git repo, no upstream, or git unavailable — print nothing (portability; 5bot doesn't require git).
- **Never mutates, never networks:** no fetch/pull/push/commit; reads local refs + status only.
- Exact wording/format and placement in the `/5bot-status` snapshot → UX.

## Out of Scope (this round)

- `git fetch`/pull/push/commit or ANY repo mutation or network call.
- Credentials, remote API calls, GitHub/Jira/Slack service integration (still out).
- Git state on commands other than `/5bot-status` (keep it to the re-orientation snapshot).
- Auto-resolving divergence, multi-repo/submodules, conflict handling.
- A one-time `/5bot-init` git nudge — optional, deferred (UX/Architect may note where it'd hook).

## Success Criteria

- A returning/cross-machine user can tell from `/5bot-status` whether they're behind or dirty, catching stale-copy divergence before acting.
- Zero disruption for non-git projects (silent no-op); zero repo mutation; no network/credentials.

## Risks

1. **Scope creep toward real git integration** (fetch/pull/push). *Mitigation:* hard rule — read-only, no network, no mutation; stated in the ticket.
2. **Stale "behind" info** (no recent fetch). *Mitigation:* label "as of last fetch"; omit ahead/behind if no upstream; never auto-fetch.
3. **Portability** (no git / not a repo / sandbox without git). *Mitigation:* silent no-op.
4. **Cross-platform git invocation** (PowerShell vs unix). *Mitigation:* Architect specifies — same pattern as the v2.1.0 zip extract.

## Assumptions

- `/5bot-status` runs in a git checkout often enough for this to help; when not, it silently no-ops.
- Reading local git refs/status is available via the host shell the agent already uses.

## Open Questions

- **OQ-1 (gate, blocking): confirm the scope ruling** — read-only local git-awareness is acceptable local tooling, NOT an out-of-scope "external integration." (Product recommends YES, in-scope.)
- **OQ-2 (→ UX):** exact line content + placement in the `/5bot-status` snapshot.
- **OQ-3 (→ Architect):** cross-platform invocation + the precise no-network commands (`git rev-list --left-right --count @{u}...HEAD`, `git status --porcelain`), and behavior when detached / no upstream.
- **OQ-4 (minor):** include a one-time `/5bot-init` nudge? (Proposed: defer.)

## Recommendation

**Status: PROPOSED (re-scoped).** Recommend APPROVE as v2.2.0: feasible, read-only, solves a demonstrated pain, and stays within the no-integrations boundary by avoiding network/credentials. Route: Product gate → `/ux` → `/architect` (gate optional under the 2-gate default) → `/dev` → `/qa` → gate → release v2.2.0.

---

# Proposed v2.1.0 — Claude Design Integration (optional)

Human request: an **option** to bring design elements created in Claude Design (claude.ai/design) into the 5bot workflow, available for *future* projects (explicitly not this exchange's specific design). Trigger: the human pasted a Claude Design "Send to local coding agent" prompt; the terminal reported the `claude_design` MCP connector wasn't connected and couldn't access the shared design link.

## ⚠️ Scope status — native carve-out, NOT a Figma-style integration (PROPOSED, needs gate approval)

**Claude Design ≠ Figma.** Claude Design (claude.ai/design) is Anthropic's **first-party / native** design surface, reached through the `claude_design` MCP connector (`api.anthropic.com/v1/design/mcp`). Figma is a separate **third-party** product. Both are "design tools," but only Figma is the kind of external integration this project deliberately avoids.

The baseline Out-of-Scope list **mis-bundled** the two — *"Visual design tools (Figma, Claude Design integration)"* — and the deferral (`decisions.md`, 2026-06-21, Q2) read *"only if natively in Claude Code/Cowork, no separate external tools."* That condition is precisely the **carve-out for the native case, i.e. Claude Design.** So v2.1.0 **activates the native carve-out for Claude Design only; Figma and other third-party tools stay out of scope.** Precedent already exists: `/ux` permits a Claude Design mock-up as an optional review aid (`ux.md` stays source of truth). Because the written list named "Claude Design integration" explicitly, we still record the human's go at the gate (anti-drift rule 2) — but as **correcting a mis-categorization**, not overturning the project's no-third-party-tools stance. **PROPOSED — not adopted until the gate approves.**

## Problem

The UX stage produces text-only specs in `ux.md`. Users increasingly design visually in Claude Design and want those artifacts to flow into implementation, so the Dev Bot builds against a real design rather than prose. Today there is no defined, reliable path:
- No workflow step to attach or import a design artifact.
- When users try Claude Design's "Send to local coding agent" prompt, it assumes the `claude_design` MCP connector is connected — and when it isn't (the human's situation), the import fails and shared design links are inaccessible. The user hits a dead end with no fallback.

## Two layers (must be separated)

**Layer A — Environment / connector (NOT plugin code).** Whether the `claude_design` MCP connector is connected and authorized is a property of the user's Claude *environment* (local Claude Code vs Cowork vs web), not something the 5bot plugin can install or force — the same lesson as `/plugin` being surface-specific. The plugin's deliverable here is **documentation / guidance** (how to connect and authorize, what to do per surface), not a code fix.

**Layer B — 5bot workflow feature (the actual deliverable).** An **optional, opt-in** "design reference / import" step, most naturally at the UX stage, that:
1. **Connector available:** import or reference the Claude Design project, save/link the artifact into the repo, and record it in `ux.md`.
2. **Connector unavailable:** fall back to the connector-free path — the "Download zip instead" bundle, a saved exported file (`*.dc.html` / HTML) dropped into the repo, or simply a referenced design URL — so the pipeline **never hard-blocks**.

`ux.md` stays canon; the design is a linked, versioned artifact.

## Target users

Existing 5bot users (indie devs, small teams) who design in Claude Design and want those designs to drive implementation — especially Cowork / desktop users, where connector availability differs from the local CLI.

## Requirements (proposed)

- **R1 — Opt-in only.** Users who don't use it see no change to the core flow (default off).
- **R2 — UX-stage home.** The design reference is captured at UX; the Dev Bot consumes the recorded artifact when implementing the relevant ticket.
- **R3 — Graceful degradation.** If the connector isn't connected, detect/state it plainly and route to the fallback; never block a gate.
- **R4 — `ux.md` stays source of truth.** Any design artifact is exported into the repo and linked (consistent with the existing `/ux` mock-up rule).
- **R5 — Document both paths.** The plugin documents connector setup AND the fallback (README / `/ux` guidance), tool-neutral about surface differences.

## MVP scope

- Add an **optional design-reference step** to the UX stage capturing `{Claude Design project URL, file name, and — if available — an exported local copy}` into `ux.md`.
- Provide **both paths** (connector import + zip/file fallback) with clear guidance on when each applies.
- Dev Bot reads the linked artifact when implementing the relevant ticket.

## Out of scope (this round)

- Auto-generating or editing designs from inside 5bot (round-trip). Import / reference only.
- Non-Claude design tools (Figma, Sketch, etc.) — Claude Design only for MVP.
- Auto-implementing a whole design without passing through the normal UX → Architect → Dev gates.
- Making the connector work where the host environment doesn't support it (Layer A is documented, not engineered).

## Priority & versioning

New, additive, opt-in feature that doesn't change the core gate flow → **MINOR bump, v2.1.0.**

## Success criteria

- A 5bot user can, by the UX gate, attach a Claude Design artifact the Dev Bot can later implement against.
- When the connector is unavailable, the user gets a clear, working fallback instead of a dead end.
- Users who don't want the feature are unaffected.
- Guidance is accurate across local Claude Code and Cowork (surface differences called out).

## Risks

1. **Connector availability varies by surface** (the human's exact pain). *Mitigation:* the feature must not depend on the connector; the fallback is first-class.
2. **Over-scoping into visual-design tooling** — the project's #1 risk. *Mitigation:* import/reference only; no design authoring; Claude Design only.
3. **`claude_design` MCP API / auth unknown and possibly unstable.** *Mitigation:* Architect confirms feasibility; design behind a thin "attach artifact" abstraction so the fallback works regardless.
4. **Source-of-truth confusion** (is the design or `ux.md` canon?). *Mitigation:* `ux.md` is canon; the design is a linked aid (reuse the existing `/ux` rule).

## Assumptions & open questions

- **ASSUMPTION:** the UX stage is the right home (design as a UX artifact), consistent with the existing `/ux` mock-up note.
- **OQ-1 (gate, blocking):** Approve reversing the Out-of-Scope decision to bring Claude Design integration in-scope as v2.1.0? Required before UX.
- **OQ-2:** Connector-first with zip/file fallback, or **fallback-first** for MVP (document the manual zip/HTML path now; treat live connector import as the stretch) — given the connector isn't reliably available in the user's environment today?
- **OQ-3:** Canonical stored form of an imported design — design URL only, exported `*.dc.html`/HTML committed to the repo, or the full zip bundle? (Affects portability / version control.)
- **OQ-4 (feasibility → Architect):** Is the `claude_design` MCP connector available and authorizable in the user's target environment(s)? Is `/design-login` a real, available command there? Confirm at Architect; MVP must degrade gracefully regardless.
- **OQ-5:** Allow the design-reference step at the Dev stage directly too, or UX-only for MVP?

### ux.md — superseded sections (v2.1.0 ×2, v2.2.0 usage, v2.2.0 git)

# UX (v2.1.0 — Claude Design reference)

> Supersedes the v1.2.0 UX audit (in git history) for the v1.3.0 work. Canonical UX artifact the Architect reads. Designs the experience for the three approved features: **F1** context-health reminder, **F2** `/5bot-status` command, **F3** next-command footer. T5 (stage-lock) is research only — hook notes for the Architect are at the end.

---

## Primary User Goals

1. **Know when to compact** — get nudged at a natural seam after a long session, never mid-artifact.
2. **Compact without fear** — understand canon is on disk, so `/compact` or a fresh session loses nothing.
3. **Re-orient in one step** — after compaction, a cold start, or days away, see stage + active ticket + last decision + the one next command.
4. **Never guess the next command** — every `/handoff` and `/gate` ends by naming exactly what to run.

---

## Main User Flows

**Flow A — Long session during a stage (F1).**
Bot works → bot self-judges the session is "long" (heuristic below) → at the next seam (`/handoff`, `/gate`, or the start of a Dev/QA turn) it appends one F1 reminder → user runs `/compact` or starts fresh → user re-runs the current command → state reloads from disk.

**Flow B — Re-orientation after compaction / cold start (F2).**
User returns or has just compacted → runs `/5bot-status` → reads the read-only snapshot → runs the single recommended command → continues the pipeline.

**Flow C — Advancing the pipeline (F3).**
User finishes a stage → `/handoff` prints the bookkeeping **and** an `▶ NEXT: /gate` footer → user runs `/gate` → gate prints a **waiting** footer (options, no command) → user replies with a verdict → gate prints the **resolved** `▶ NEXT` footer → user runs that command.

**Flow A+C interaction (critique fix):** When a long session reaches a stage seam, `/handoff` runs then `/gate` runs right after. Both could self-judge "long." **Rule: F1 fires at most once across that handoff→gate pair** — if `/handoff` already showed the reminder, `/gate` suppresses it. (Mirrors the existing QA→gate suppression.)

---

## Screen List

These are text "screens" (command output). Two are reusable components appended to existing commands.

| # | Screen | New / Modified | Type |
|---|--------|----------------|------|
| S1 | `/5bot-status` snapshot | **New** | Full command output |
| S2 | F1 context-reminder block | **New** | Reusable component (appended) |
| S3 | F3 next-command footer | **New** | Reusable component (appended) |
| S4 | `/handoff` output | Modified (+S3, optional S2) | Command output |
| S5 | `/gate` output | Modified (+S3 waiting/resolved, optional S2) | Command output |
| S6 | Dev turn | Modified (optional S2 one-liner) | Persona behavior |
| S7 | QA turn | Modified (optional S2 one-liner) | Persona behavior |

---

## Navigation Model

The pipeline is **linear and footer-driven**: Product → UX → Architect → Dev ⇄ QA → release, with human gates after Product, Architect, and QA. The user never has to remember the order — the **F3 footer (S3)** always names the next step. **`/5bot-status` (S1)** is the "you are here" re-entry point: the one command that re-orients from disk after any interruption. F1 (S2) is an ambient nudge layered onto existing seams, never its own screen.

All output is **plain Markdown** (horizontal rules, bold, backticks) — no tables, color, or emoji-dependence — so it renders identically in terminal, web, and Cowork.

---

## Screen-by-Screen Detail

### S1 — `/5bot-status` (new, read-only)

**Reads (never writes):** `project-state.md` (canon — stage, active ticket, open questions, last-updated), `decisions.md` (newest block only), `handoff.md` (the "Next Bot + Instructions" line).

**Sections, top to bottom:**
1. **Header + contract subline** — `5bot status — <Project Name> (project-state v<x.y.z>)`, then a muted line: `Read-only snapshot. Nothing was modified. Re-run any command to reload from disk.`
2. **Stage:** verbatim from `## Current Stage`, including the gate-status parenthetical.
3. **Active ticket:** verbatim from `## Current Active Ticket`; show "None yet." plainly rather than hiding the line.
4. **Last decision:** the single newest `decisions.md` block — heading, STATUS, date/by, one-line gist. Never dump the whole log.
5. **Open questions:** only items still OPEN; collapse resolved into a count (`(N resolved — see project-state.md)`); `None open.` if none.
6. **→ Recommended next command:** exactly ONE, arrow-prefixed, with a short why-clause. Derived from `handoff.md`'s next-step line, cross-checked against the canon stage/gate.
7. **Freshness footer:** muted echo of `## Last Updated / By` + `Canon is on disk (…) — safe to /compact.`

**Example output (mid-pipeline):**
```
5bot status — 5bot (project-state v1.2.1)
Read-only snapshot. Nothing was modified. Re-run any command to reload from disk.

Stage:            UX (designing v1.3.0: F1 reminder · F2 /5bot-status · F3 footer) — Product gate APPROVED
Active ticket:    None yet. v1.3.0 proposal approved; no dev ticket cut. Pending: T4 (v1.2.2), T5 (research).
Last decision:    Product Gate — v1.3.0 Scope — APPROVED (2026-06-21 / Human)
                  F1/F2/F3 approved; T5 designed alongside but ships later on its own gate.
Open questions:   None open. (9 resolved questions on file — see project-state.md)

→ Run /ux   — Product gate is APPROVED; UX designs the F1 copy, /5bot-status output, and F3 footer.

State last updated 2026-06-21 / Product gate APPROVED → UX stage.
Canon is on disk (project-state.md · decisions.md · handoff.md · stage files) — safe to /compact.
```

### S2 — F1 context-reminder block (new, reusable)

A single fenced block appended at a seam **only when the bot self-judges the session is long.** Reassurance leads; the action follows.

- **Reassurance line (shared across the system):** *"All canon is on disk — project-state.md, decisions.md, and the stage files. Compacting or restarting loses nothing; re-run the current command to reload."*
- **Voice variants:**
  - **Terse (for `/handoff` + `/gate`, matches command-file voice):** *"Long session — context may be drifting. State is safe on disk (project-state.md, decisions.md, stage files), so nothing is lost. Run `/compact` or start a fresh session, then re-run the current command to reload."*
  - **Minimal one-liner (for Dev/QA persona footers, sits quietly under "Token discipline"):** *"Long session — safe to `/compact` or restart. Canon is on disk; re-run the current command to reload."*

**Self-judgment heuristic (no token API exists).** Treat the session as long when ANY holds: (1) the bot had to re-read a state file it already loaded this session (it scrolled out); (2) ≥2 full pipeline stages, or ≥3 Dev↔QA hops, in one unbroken session; (3) the bot catches itself re-deriving something already settled in `decisions.md`; (4) earlier details now feel hazy and need reconstruction. **Guardrails:** at most once per session (twice only if the user keeps going well past it), only at a seam (never mid-artifact), never on a plainly short session. **When unsure, stay silent.**

### S3 — F3 next-command footer (new, reusable)

Always the **final block** in `/handoff` and `/gate` output; nothing prints after it. If F1 (S2) also fires, **F1 prints above F3** so NEXT stays last. Shared shell:
```
---
**▶ NEXT:** `<command>` — <≤12-word reason>
<optional second line: branch/condition only when a choice exists>
```
Rules: command in backticks with leading slash (copy-pasteable); exactly one primary NEXT; alternates go under an indented "If instead:" line, never as a competing NEXT; the next command is **derived from `## Current Stage`, never guessed** — if the stage is unreadable, NEXT points to `/5bot-status` instead of inventing one.

### S4 — `/handoff` (modified)

Existing bookkeeping output, then (optional S2 if long), then **always S3**. Because `/handoff` only ever runs immediately before a gate, its footer is deterministic: `▶ NEXT: /gate`.

### S5 — `/gate` (modified)

Prints the F3 footer **twice across its lifecycle** because it can't know the verdict until the human replies:
- **(a) Gate OPEN — before reply:** a *waiting* footer naming **no** command, restating "nothing proceeds until you reply," and listing the six verdict options. (Optional S2 may appear above it if long — but **not** if S4 already nudged this transition.)
- **(b) Gate RESOLVED — after verdict:** the real `▶ NEXT`, resolved from (stage just completed) × (verdict). See the verdict map below.

### S6 / S7 — Dev & QA turns (modified)

A one-line S2 (minimal variant) appears under "Token discipline" **only** if the session is self-judged long, **at most once**, at the start of the turn — never mid-artifact. **QA suppresses it if `/gate` will run next** (the gate covers it; avoid double-nudge). Dev→QA loops have no gate, so a long Dev loop is exactly where this earns its place.

---

## Forms & Fields

None. 5bot has no input forms — the only "inputs" are slash commands and the human's one-word gate verdict. The verdict vocabulary is fixed and unchanged: `APPROVED · APPROVED WITH CHANGES · REVISE · REJECT · DEFER · NEEDS CLARIFICATION`.

---

## Buttons & Actions

Actions are commands the user types. v1.3.0 adds/changes:

| Action | Where | Effect |
|--------|-------|--------|
| `/5bot-status` | Anywhere in a 5bot project | Prints S1 snapshot; **writes nothing** |
| `/compact` (Claude built-in) | Suggested by S2 | Compacts context; canon unaffected |
| `▶ NEXT` command | Tail of `/handoff` & `/gate` | The exact next pipeline step |

---

## Empty / Error / Success States

**Success:** `/5bot-status` prints the full snapshot; the F3 footer names one unambiguous next command; F1 fires at a seam and the user compacts confidently.

**Empty / edge states for `/5bot-status`:**
- **No `project-state.md`:** don't error, don't invent. Print `5bot status — no project state found in this directory.` + `This folder isn't set up for 5bot yet.` + `→ Run /5bot-init to scaffold the workflow files.`
- **Initialized but untouched:** render the layout with honest template values — `Stage: (not started)`, `Active ticket: None yet.`, `Last decision: none recorded yet.` → `→ Run /product`.
- **`decisions.md` empty:** `Last decision: none recorded yet.` (don't fail the parse).
- **`handoff.md` missing/stale vs canon:** trust `project-state.md` (source-of-truth order), derive NEXT from the canon stage, and add a muted note: `(handoff.md looks stale — next command derived from canon stage.)`
- **Just-compacted:** no special branch — re-reads disk and prints the normal snapshot; the contract subline + freshness footer are the reassurance.

**Error / fallback for F3 footer:**
- **`## Current Stage` unreadable:** `▶ NEXT: /5bot-status — current stage is unclear; re-orient before choosing a command.`

**F1 non-states (suppression):** short session → no reminder; already nudged this session → silent (max once, twice only if the user runs well past it); mid-artifact → never. **When unsure, stay silent — a missed nudge is cheaper than a nagging one.**

### Gate verdict → NEXT map (S5b)

| Stage at gate | Verdict | `▶ NEXT` |
|---|---|---|
| Product | APPROVED / W-CHANGES | `/ux` |
| UX | APPROVED / W-CHANGES | `/architect` |
| Architect | APPROVED / W-CHANGES | `/dev` (name the first ticket, e.g. T1) |
| QA | APPROVED / W-NOTES, tickets remain | `/dev` (next ticket) |
| QA | APPROVED / W-NOTES, backlog empty | `/handoff` (record release-ready) |
| any | REVISE | re-run the **same** stage's command, then `/handoff` + `/gate` |
| any | NEEDS CLARIFICATION | none — bot answers in-thread, gate re-opens (waiting footer) |
| any | REJECT | `/product` (re-scope) — or stop; logged Rejected |
| any | DEFER | none — paused; logged Deferred; resume via the stage command, re-orient via `/5bot-status` |

---

## Usability Concerns

1. **Reminder fatigue (highest priority).** Mitigated by: self-judgment + max-once-per-session + seam-only + the handoff→gate and QA→gate suppression rules + "silent when unsure." This is the single biggest risk to F1 feeling good.
2. **Post-compaction blind spot (insight).** A freshly-compacted bot's "long" signals reset, so F1 won't immediately re-fire after a compaction — which is correct (don't nag right after the user just acted). `/5bot-status` is the deliberate re-entry instead.
3. **Footer must stay last.** If F1 + F3 both fire, F1 goes above F3 so the actionable NEXT is always the final line a skimmer sees.
4. **Snapshot brevity.** `/5bot-status` must stay ~10 lines: one decision only, resolved questions collapsed to a count. Growth here defeats the "fast re-orient" purpose.
5. **Consistency as a system.** F1/F2/F3 deliberately reuse one idea — "canon is on disk, safe to compact" — and one shell (`---` + bold lead). They should read as one feature, not three.

---

## Assumptions

- `/compact` is the relevant Claude Code command; phrasing also says "or start a fresh session" so it stays meaningful if `/compact` behaves differently in Cowork (to be verified at QA).
- Bots can self-assess drift well enough for the heuristic; if they over- or under-fire in practice, only the heuristic text needs tuning (no structural change).
- Plain-Markdown rendering is safe across terminal / web / Cowork.

## Open Questions — RESOLVED at UX gate (2026-06-21)

- **OQ-1:** ✓ **RESOLVED — bot self-judgment.** The "long" heuristic (4 drift signals + guardrails) is the shipped mechanism; it lives in the Dev/QA personas and the F1 block spec.
- **OQ-2:** ✓ **RESOLVED — keep `/5bot-status` brief.** No Known Risks section; it stays at stage / active ticket / last decision / open questions / one next command / freshness footer.

## T5 hook notes (research only — for the Architect, NOT designed here)

F2/F3 are **read-only** features over the **same state** a future hard lock would **enforce** — making them a low-risk precursor. Key hooks the Architect should scope under T5:
- **`## Current Stage`** is the cursor but is **free-text prose** today. A hard lock needs a machine-parseable token (e.g., enum `PRODUCT|UX|ARCHITECT|DEV|QA|RELEASE` + a `gate: APPROVED|PENDING` flag) alongside the human-readable text. **F2/F3 should read whatever structured form is chosen, so the format is settled once.**
- **`decisions.md`** is the authoritative approval ledger; a lock must verify the gate approval *exists there*, not trust the cursor alone (a stale/hand-edited stage could wrongly unlock).
- **Enforcement point:** a pre-flight check at the top of each command (read stage + decisions, refuse/warn if out of order). **Dev→QA is intentionally ungated** — the lock table is not uniform.
- **Honesty:** with no runtime hook, a "hard" lock is still bot-honored instruction — it raises the floor, it is not cryptographic. T5 should say so.
- **Migration:** legacy projects lack the token — fall back to "unlocked + warn," never hard-block. Adding the token to the template is ≥ MINOR and must sync the `_framework\5bot\` copy and the standalone plugin copy.

---

# UX (v2.1.0 — Claude Design Integration, optional)

> Canonical UX artifact the Architect reads for v2.1.0. Designs the experience of the **optional Claude Design "design-reference" step** approved at the 2026-06-22 Product gate. **Native Claude Design only** (Figma stays out). In every downstream project, that project's `ux.md` stays source of truth; a design is a **linked aid, never an auto-build input.**

## Primary User Goals (v2.1.0)

1. **Attach a Claude Design artifact** to a 5bot project so the Dev Bot builds against the real design.
2. **Never hit a dead end** — when the `claude_design` connector isn't available in this surface, a fallback still records the design.
3. **Always know which path is active** (connector import vs fallback) and what to do next.
4. **Keep `ux.md` canon** — the design is referenced/linked, not treated as the spec.

## Design stance on the open questions — RESOLVED at UX gate (2026-06-22)

- **OQ-2 → RESOLVED: the `.zip` download is the first-class MVP path.** Human direction: *"I can download the design from Claude Design as a `.zip` — go with that."* MVP = the user exports the design as a `.zip` from Claude Design, drops it in the repo, and the bot records it. The `claude_design` connector import (D2) is a **progressive enhancement, deferred past MVP** — so the **MVP carries no dependency on the connector**, which sidesteps the surface-availability problem entirely.
- **OQ-3 → RESOLVED: the committed `.zip` (with its extracted `*.dc.html`) is the stored artifact; `ux.md` holds the Design Reference block that links it.** Saved in the repo (proposed `design/`); exact path/format = Architect (OQ-7).
- **OQ-5 → ACCEPTED: UX-only capture for MVP** (no objection raised). `/ux` captures; the Dev Bot consumes. Ad-hoc Dev-stage attach is post-MVP.

## Main User Flow — Flow D: attach a Claude Design reference at `/ux`

1. During `/ux`, the user signals they have a Claude Design design — by pasting the **project URL**, pasting Claude Design's **"Send to local coding agent" prompt**, or naming an **exported file**.
2. The UX Bot checks whether the `claude_design` connector is connected/authorized in this surface.
   - **Connected → D2 (connector import):** use the claude_design tools to fetch the project/file(s); record the reference + imported file list; optionally export a local copy into the repo.
   - **Not connected → D3 (fallback):** state plainly that the connector isn't available *here* (it's surface-dependent), then take the fallback — record the URL as a reference and/or accept an exported file path (the user uses Claude Design's **"Download zip instead"** or saves the `*.dc.html` and drops it in the repo).
3. Either path → **D4**: write the **Design Reference** block into `ux.md`. Continue normal UX (flows/screens can now cite the design). The design never auto-generates code — Dev implements via tickets.

**Branch default:** no design → plain text-only `/ux`, exactly as today. The step is opt-in and adds zero friction when unused.

## Screen List (text "screens" = command behavior)

| # | Screen | New / Mod | Type |
|---|--------|-----------|------|
| D1 | `/ux` design-reference step (offer + capture) | Modified `/ux` | Command behavior |
| D2 | Connector-available import sub-flow | New | Sub-flow |
| D3 | Connector-unavailable fallback sub-flow | New | Sub-flow |
| D4 | "Design Reference" block written to `ux.md` | New | Artifact section |
| D5 | Connector-status / error / guidance states | New | States |

## Navigation Model

The design-reference step is an **optional branch inside `/ux`**, never a gate or blocker. Plain Markdown only (terminal / web / Cowork parity). The connector is an *enhancement*; the fallback is the floor.

## Screen-by-Screen Detail

### D1 — `/ux` design-reference step
At the top of `/ux`, if the user supplied (or pastes) a Claude Design URL / prompt / file, enter Flow D; otherwise proceed with normal UX silently. The bot parses a pasted "Send to local coding agent" prompt to seed `{project URL, file name}` automatically.

### D2 — Connector available (import)
> *Claude Design connector detected. Importing **Tag Hag.dc.html** from the project… imported. Recorded in `ux.md` → Design Reference. (Saved a local copy to `design/tag-hag.dc.html`.)*

### D3 — Connector NOT available (fallback) — the key state
```
The claude_design connector isn't connected in this environment, so I can't pull the
design from the link directly. (Connector availability depends on your Claude surface —
local Claude Code vs Cowork vs web — not on 5bot.) Two ways forward, both fine:

  • Quick: I'll record the design URL as a reference now (note: a URL can change or
    expire and isn't version-controlled).
  • Durable (recommended): in Claude Design choose "Download zip instead" (or save the
    *.dc.html), drop the file into this repo, and tell me the path — I'll link it.

Either way, ux.md stays the source of truth. Want me to (a) record the URL,
(b) wait for an exported file, or (c) skip the design for now?
```
*The copy deliberately does NOT tell the user to run a specific connect command, because whether one exists is surface-dependent and unverified (see OQ-6). When Architect confirms the real connect/auth step per surface, D3 gains a third "connect it" option.*

### D4 — "Design Reference" block (written to `ux.md`)
```
## Design Reference (Claude Design)
- Source:         Claude Design — <project URL>
- File(s):        Tag Hag.dc.html
- Import method:  fallback-zip            # MVP; "connector" reserved for deferred enhancement (T15)
- Local artifact: design/tag-hag.dc.html (committed) | none (URL only)
- Captured:       2026-06-22
- Covers:         <which screens / flows this design informs>
- Note:           ux.md is canon; this design is a linked aid, not the spec.
```
*Exact file location / format is Architect's call (OQ-7); the block schema above is the UX contract.*

### D5 — States
- **Connected + authorized:** import succeeds → D4. Success.
- **Connected + unauthorized:** offer to authorize (the real step, per OQ-6) OR use fallback. Never block.
- **Not connected (reported case):** D3 fallback. Never block.
- **URL-only (no export):** record, but warn it's not version-controlled; recommend a committed export.
- **Bad / expired / inaccessible link:** say so plainly; offer a URL-only note or ask for an export.
- **No design:** skip; normal `/ux`.

## Forms & Fields

No GUI forms. Inputs are: a pasted Claude Design URL or "send to agent" prompt, an optional exported file path, and a one-word choice (record URL / wait for file / skip).

## Buttons & Actions

| Action | Where | Effect |
|--------|-------|--------|
| Paste Claude Design URL / "Send to local coding agent" prompt | during `/ux` | Seeds + starts the design-reference capture |
| Provide exported file path (`*.dc.html` / zip) | `/ux` fallback | Records + links a committed local artifact |
| Connector import | when connector present | Pulls the design via `claude_design` tools |

## Empty / Error / Success States

**Success:** a Design Reference block lands in `ux.md`, Dev can implement against it, and the user is never blocked by a missing connector. See **D5** for the remaining states.

## Usability Concerns

1. **Don't over-promise the connector (highest).** The user's actual pain was being told to use a connector/command that wasn't there. Copy stays **conditional** and always pairs with the working fallback — no instruction to run an unverified command (gated on OQ-6).
2. **Source-of-truth clarity.** The block states `ux.md` is canon and the design is an aid — prevents "the mock-up is the spec" drift.
3. **Durability.** URL-only references can rot; nudge toward a committed export.
4. **Stay invisible when unused.** Projects that don't use Claude Design see no change to `/ux`.
5. **No auto-build.** Capture only; Dev implements via tickets — matches the human's "I want the option, not this specific design built" intent.

## Visual aid for this gate (optional)

Per the `/ux` protocol, a Claude Design mock-up of this screen list could serve as a human-review aid — but the connector isn't available in this environment (the very gap this feature addresses), so this gate uses `ux.md` as the canonical and only artifact. If a mock-up is later produced, export it into the repo and link it here.

## Assumptions

- When present, the `claude_design` connector exposes import/fetch tools callable by the agent (feasibility → Architect, OQ-6).
- "Send to local coding agent" prompts embed a parseable `{project URL, file name}`.
- A plain-Markdown reference block needs no schema enforcement at MVP.

## Open Questions

- ✓ **OQ-2 / OQ-3 / OQ-5 — RESOLVED at UX gate (2026-06-22):** `.zip` download is the first-class MVP path; the committed `.zip` (+ extracted `*.dc.html`) is the artifact, linked from a `ux.md` Design Reference block; UX-only capture. Connector import deferred to a progressive enhancement.
- **OQ-6 (→ Architect, now NON-blocking for MVP):** real connect/authorize path for `claude_design` per surface (is `/design-login` a real command in Code/Cowork/web?) — only needed when the connector-import enhancement is built; the MVP zip path doesn't depend on it.
- **OQ-7 (→ Architect, minor):** default repo location + format for committed design artifacts (proposed `design/`).

---

# UX (v2.2.0 — Usage-Aware Status Reporting)

> ⛔ **SUPERSEDED (2026-06-29).** This API-usage design was built then reverted at the QA gate (model can't read usage %). v2.2.0 was re-scoped to **git-based sync-awareness**; see "UX (v2.2.0 re-scoped — Sync-Awareness via read-only git status)" below. Retained for history.

> Canonical UX artifact the Architect reads for v2.2.0. Designs the experience of **read-only API usage status** integrated into the `/5bot-status` command. Approved at Product gate (2026-06-26) as a soft personal preference — informational, non-blocking usage visibility.

## Primary User Goal (v2.2.0)

**Know current API hourly usage** without leaving the workflow, so you can decide whether to continue or pause before the next bot command.

## Main User Flow (v2.2.0) — Flow E

User at any point in the pipeline:
1. Runs `/5bot-status` (existing re-orientation command).
2. Sees a **read-only API usage line** showing current hourly quota consumption (e.g., "API hourly usage: 76% (resets at 15:00 UTC)").
3. Uses that info to decide: continue normally, or pause and resume later.

**No workflow change.** The line is informational; it never blocks execution. User decides the action.

## Screen List (v2.2.0)

| # | Screen | New / Mod | Type |
|---|--------|-----------|------|
| E1 | `/5bot-status` usage line | Modified S1 | Command output section |

## Navigation Model (v2.2.0)

The usage line is a **read-only, optional data field** appended to `/5bot-status`. It adds one line between "Active ticket" and "Last decision" (or alternative position per Architect). No new command, no gate, no fork. Silent omission if data unavailable (no error, no placeholder).

## Screen-by-Screen Detail (v2.2.0)

### E1 — `/5bot-status` usage line (modification to S1)

**Location in output:** After "Active ticket" and before "Last decision" (or after all core sections, just before the final "→ Run" footer — Architect decides based on readability).

**Format:**
```
Usage:  API hourly usage: 76% (resets at 15:00 UTC)
```

**Data displayed:**
- **Percentage:** Current usage as a simple integer percentage (0–100%) of the hourly rolling window.
- **Reset time:** UTC time (HH:MM format) when the hourly quota resets, so users know when it's safe to resume. Example: "15:00 UTC" (not relative like "in 23 minutes" — UTC is unambiguous).
- **Tone:** Neutral, factual. No warning emoji (⚠️), no color, no urgency. Just the numbers.

**Fallback / unavailable:**
- If the host environment doesn't expose API usage data (no MCP context, API unavailable, surface doesn't support it): **silently omit the line.** Print nothing, no placeholder, no error. The command succeeds either way.

**Example output (with usage):**
```
5bot status — 5bot (project-state v2.2.0)
Read-only snapshot. Nothing was modified. Re-run any command to reload from disk.

Stage:            Architect (approving v2.2.0: usage-aware status line in /5bot-status)
Active ticket:    None yet.
Usage:            API hourly usage: 76% (resets at 15:00 UTC)
Last decision:    Product Gate — v2.2.0 Usage-Aware Status — APPROVED (2026-06-26 / Human)
Open questions:   3 blocking (see product.md) — resolved by Architect.

→ Run /architect  — Confirm API feasibility and decide reset-time format.

State last updated 2026-06-26 / Product gate APPROVED → UX stage.
Canon is on disk (project-state.md · decisions.md · handoff.md · stage files) — safe to /compact.
```

**Example output (usage unavailable):**
```
5bot status — 5bot (project-state v2.2.0)
Read-only snapshot. Nothing was modified. Re-run any command to reload from disk.

Stage:            Architect (approving v2.2.0: usage-aware status line in /5bot-status)
Active ticket:    None yet.
Last decision:    Product Gate — v2.2.0 Usage-Aware Status — APPROVED (2026-06-26 / Human)
Open questions:   3 blocking (see product.md) — resolved by Architect.

→ Run /architect  — Confirm API feasibility and decide reset-time format.

State last updated 2026-06-26 / Product gate APPROVED → UX stage.
Canon is on disk (project-state.md · decisions.md · handoff.md · stage files) — safe to /compact.
```
(No "Usage" line if data unavailable.)

## Usability Concerns (v2.2.0)

1. **Silent fallback is critical.** If the API query fails or the host doesn't expose usage, omit the line completely. Never show "Usage: unavailable" or error. The command must always succeed.
2. **Placement matters.** The line should sit near the "active ticket" / "decision" context (mid-output), not at the end, so users see it before deciding what to run next.
3. **No alarm.** Keep tone neutral; numbers only. This is informational for planning, not a warning.
4. **Reset time clarity.** UTC is unambiguous; a relative format ("in 15 minutes") is tempting but fragile if the user copies/pastes the output later.

## Assumptions (v2.2.0)

- **ASSUMPTION:** API usage data is available via Claude Code MCP context, host environment variable, or Claude.ai dashboard API (never via explicit user-managed credentials in project files).
- **ASSUMPTION:** The "hourly" window is Claude API's rolling 1-hour limit. Reset time = current hour + 60 minutes.
- **ASSUMPTION:** Users can interpret "76% of hourly quota" without further explanation.
- **ASSUMPTION:** Silent omission when unavailable is acceptable (no user-facing fallback needed).

## Open Questions (v2.2.0)

**OQ-1 (Data source / feasibility):** Can Claude Code expose API usage via MCP, environment variable, or host context? Is there a quota / usage endpoint available, or is the data dashboard-only? → **Architect to confirm.**

**OQ-2 (Reset time format):** Show "15:00 UTC" (unambiguous) or "~15 minutes from now" (human-friendly but fragile)? → **Architect to decide.** Recommend UTC for durability.

**OQ-3 (Exact placement):** After "Active ticket" (context-first) or at the end of core sections (least intrusive)? → **Architect to decide** based on the final S1 layout. Recommend after Active ticket.

---

# UX (v2.2.0 re-scoped — Sync-Awareness via read-only git status)

> Supersedes the API-usage UX above (reverted at the QA gate). Designs the read-only **git-awareness line** in `/5bot-status`, approved at the re-scope Product gate (2026-06-29). Read-only, no network, no mutation, neutral tone, plain Markdown.

## Primary User Goal (v2.2.0 re-scoped)

See at a glance, in `/5bot-status`, whether the local copy is in sync with its remote and whether there are uncommitted changes — so you catch a stale-copy / unsaved-work situation **before** acting on it.

## Main User Flow — Flow E (re-scoped)

1. User runs `/5bot-status`.
2. If the project is a git repo, the snapshot's freshness footer includes one read-only **git line** (sync + cleanliness).
3. User reads it and decides what to do — purely informational; 5bot suggests no action and never touches the repo.
4. If not a git repo (or git unavailable), the line is silently omitted; the rest of the snapshot is unchanged.

## Screen List (v2.2.0 re-scoped)

| # | Screen | New / Mod | Type |
|---|--------|-----------|------|
| E1 | `/5bot-status` git line | Modified S1 (freshness footer) | Command output line |

## Navigation Model

A single read-only line appended to the **freshness footer** of `/5bot-status`. No new command, no gate, no fork. The line is **informational only** — it does **not** change the recommended next command (which stays above the footer). Silent omission when not a git repo.

## Placement decision (OQ-2, resolved here)

**In the freshness footer, as its last line** — after "State last updated…" and "Canon is on disk… safe to /compact." Rationale: git/sync status answers *"how current/trustworthy is my local copy?"* — the same question the freshness footer already addresses (last-updated + canon-on-disk). Grouping them keeps all "state freshness/trust" signals together and keeps the actionable **→ next command** prominent (footer stays last). (The old API-usage line sat mid-snapshot to inform the next-command decision; for git-awareness the "trust/freshness" framing fits the footer better.)

## Screen-by-Screen Detail

### E1 — `/5bot-status` git line

Appended as the final line of the freshness footer. Format:
```
git: <branch> · <sync state> · <cleanliness>
```
- **branch:** current branch name; `detached HEAD` if detached.
- **sync state** (vs upstream, from already-fetched refs — labeled, because the command does NOT fetch): `N behind origin (last fetch)` · `M ahead` · `N behind, M ahead (last fetch)` · `even with origin (last fetch)` · `no upstream` (omit ahead/behind when none).
- **cleanliness:** `clean` or `uncommitted changes` (from `git status`).

**Example (footer with git line):**
```
State last updated 2026-06-29 / Product gate APPROVED → UX.
Canon is on disk (project-state.md · decisions.md · handoff.md · stage files) — safe to /compact.
git: main · 2 behind origin (last fetch) · uncommitted changes
```

## Empty / Error / Success States

- **Git repo, has upstream:** full line (branch · sync · cleanliness). **Success.**
- **Git repo, no upstream:** `git: <branch> · no upstream · <cleanliness>` (omit ahead/behind).
- **Detached HEAD:** `git: detached HEAD · <cleanliness>` (omit ahead/behind).
- **Even + clean:** `git: <branch> · in sync · clean` (terse all-good render; still shown — consistent sync indicator).
- **Not a git repo / git unavailable / any git error:** **omit the line entirely** — no error, no placeholder, command still succeeds (silent no-op).

## Forms & Fields

None. The git line is pure read-only output; no inputs.

## Buttons & Actions

None new. The line is informational; if the user wants to act (commit / pull), they do so in their own terminal — 5bot neither prompts nor performs it.

## Usability Concerns

1. **Informational, never advisory (highest).** The line states facts; it does NOT tell the user to pull/commit and does NOT change the recommended next command. (A soft nudge is out of scope per Product.)
2. **Honest staleness.** "(last fetch)" signals that ahead/behind reflects the last fetch, not live — the command never fetches. Prevents false confidence.
3. **Portability / no noise.** Silent omission when not a git repo (5bot doesn't require git); no errors, no "n/a".
4. **Brevity.** Exactly one line; keeps `/5bot-status` to ~11 lines. Don't expand into multi-line git detail.
5. **Neutral tone, plain Markdown, no emoji** — consistent with the rest of the snapshot; renders identically in terminal / web / Cowork.
6. **Privacy.** Only local branch / sync / dirty facts; no remote URLs, emails, or identifying info.

## Assumptions (v2.2.0 re-scoped)

- Reading local git state is available via the host shell the agent already uses (exact commands → Architect).
- `/5bot-status` is often run inside a git checkout; when not, it silently no-ops.

## Open Questions (v2.2.0 re-scoped)

- ✓ **OQ-2 (placement / format): RESOLVED here** — freshness-footer line, format above.
- **OQ-3 (→ Architect):** exact no-network commands (`git rev-list --left-right --count @{u}...HEAD`, `git status --porcelain`, branch via `git symbolic-ref`/`rev-parse`), detached / no-upstream handling, cross-platform invocation, and graceful omission on any git error.
- **OQ-4 (minor, deferred):** a one-time `/5bot-init` nudge to set up git — Product deferred; not designed here.

## Visual aid for this gate (optional)

Per the `/ux` protocol, a Claude Design mock-up could illustrate the footer line — but it's a single text line; `ux.md` is the canonical and sufficient artifact here. No mock-up needed.

---


### architecture.md — superseded sections (v2.0.0, v2.1.0 zip, v2.2.0 git, v2.2.0 usage)

# v2.0.0 — Pipeline Awareness & Guardrails (current)

> Renumbered from v1.3.0 to **v2.0.0** at release: completing the `five-bot` → `5bot` install rename is a breaking change (existing `five-bot@lawson-design` installs must reinstall as `5bot@lawson-design`), so per the semver rule it's a MAJOR bump. The features themselves (F1/F2/F3) are unchanged.

Turns `ux.md` (v1.3.0) into a build plan for **F1** (context reminder), **F2** (`/5bot-status`), **F3** (next-command footer). T5 (stage-lock) stays research — now informed by the `ux.md` "T5 hook notes."

## Key architectural decision: two synced copies

The framework exists in **two places** and both must stay in sync:
- **`plugins/five-bot/`** — the standalone, *published* plugin (commands/, skills/, README, plugin.json). **This is the canonical, shipped artifact.**
- **`_framework/5bot/`** — the in-repo working copy used for local dogfooding (rules.md, personas/, templates/).

**Decision:** Implement F1/F2/F3 **canonically in the plugin**, then **sync the relevant guidance into `_framework/5bot/`** so local runs match. Every ticket below carries a sync acceptance criterion. The published GitHub repo (`AngryMunky/5bot-plugin`) mirrors `plugins/five-bot/` plus root files; `plugin.json` and `.claude-plugin/marketplace.json` versions must match (see T9).

## DRY placement (avoid duplicating logic 4×)

- The **`five-bot` skill** (`skills/five-bot/SKILL.md`) is the single home for the canonical **F1 block** (reassurance + variants + heuristic + guardrails) and the **F3 footer shell + pipeline/verdict map**. Individual command files carry only a short *trigger* that points at the skill.
- `/5bot-status` (F2) reuses the **same successor logic** as F3 (defined once in the skill) to compute its single recommended next command.

## Implementation sequence (sequential — shared files)

`T6 (F3) → T7 (F2) → T8 (F1) → T9 (release)`. T6 first because F1 references "print above the F3 footer"; T6/T8 both edit `SKILL.md`, `handoff.md`, `gate.md`, so they must not run in parallel (same merge-conflict reasoning as the v1.2.1 T1–T3 sequencing). T7 is mostly a new file but also touches the SKILL command roster + README, so it's sequenced too.

## v1.3.0 Assumptions
- `/compact` is the relevant Claude Code command; copy also says "or start a fresh session" to stay valid if Cowork differs (verify at QA — the human has a Cowork env).
- Bots can self-judge "long" well enough; tuning is text-only (no structural change).
- The `five-bot` skill is loaded whenever a command runs, so centralizing F1/F3 text there is reliably in context.

## v1.3.0 Open Questions
- Does the published plugin's command set auto-expose a new `commands/5bot-status.md` on install, or must it also be added to `marketplace.json`/skill roster? (T7 verifies; T9 owns version/roster sync.)
- Exact wording of the skill's F1/F3 sections is Dev's to draft from the `ux.md` spec; QA checks against `ux.md`.

## v1.3.0 Tickets

See `dev-qa.md` Backlog for the full ticket cards (T6–T9). Summary:
- **T6 — F3 next-command footer:** skill defines the `--- / ▶ NEXT:` shell + verdict map; `handoff.md` (NEXT=/gate) and `gate.md` (waiting → resolved footer) reference it.
- **T7 — F2 `/5bot-status`:** new read-only `commands/5bot-status.md`; brief 6-section snapshot (no Known Risks per OQ-2); 5 edge cases; reuses F3 successor logic.
- **T8 — F1 context reminder:** skill defines reassurance + terse/minimal variants + self-judgment heuristic + guardrails; triggers in `handoff.md`/`gate.md` (terse, above F3) and `dev.md`/`qa.md` (minimal); handoff→gate and QA→gate suppression.
- **T9 — v1.3.0 release:** bump `plugin.json` + `marketplace.json` (1.2.1→1.3.0), document `/5bot-status` + F1/F3 in README, sync `_framework/5bot/`, update root CLAUDE.md + MEMORY.md, commit + tag.

---

# v2.1.0 — Claude Design Integration (zip-first)

> Build plan for the optional Claude Design **design-reference** step (Product gate 2026-06-22; refined at the UX gate to **zip-first** — the `.zip` download is the MVP path, live `claude_design` connector import is deferred). Native Claude Design only. Turns `ux.md` "v2.1.0" (Flow D, D1–D5) into tickets. Paths use the current `plugins/5bot/` layout.

## Tech Stack (delta)
Unchanged: Markdown-only plugin, plain-text state, no network integrations. **New interaction:** the workflow now reads a user-provided local `.zip` (Claude Design "Download zip instead") and its extracted contents. Extraction uses **host tools the agent already has** (PowerShell `Expand-Archive` on Windows; `unzip`/`tar` on Unix) — local file I/O, not a service integration, so the "no integrations / no external deps" stance holds.

## Key decision — OQ-7: where the design lives (PROPOSED, confirm at gate)
- Extracted design lives at **`design/<slug>/`** in the downstream project (`<slug>` = kebab-cased design file name; `Tag Hag.dc.html` → `design/tag-hag/`).
- Keep the **original `.zip`** at `design/<slug>.zip` for provenance (pristine, re-extractable source).
- **Primary artifact** Dev builds against = the extracted `…/<name>.dc.html`; the `ux.md` Design Reference block links it.
- Rationale: `design/` is conventional and git-friendly; extracted HTML is diffable; the retained zip preserves the exact export.
- **OQ-8 (minor):** create `design/` on-first-use (proposed) vs at `/5bot-init` → propose on-first-use (no empty-dir churn).

## DRY placement (one home, like F1/F3)
The **`five-bot` skill** (`skills/five-bot/SKILL.md`) is the single home for the **Design Reference procedure**: the block schema, the zip locate→extract→record steps (with the cross-platform extract note), and the guardrails ("`ux.md` is canon; design is a linked aid; never auto-build the whole design; capture is UX-only"). `commands/ux.md` (capture) and `commands/dev.md` (consume) carry only short triggers pointing at the skill.

## Data Model — the Design Reference block (the contract)
A new `## Design Reference (Claude Design)` section is written into the project's **`ux.md`** (canon). Schema:
```
## Design Reference (Claude Design)
- Source:         Claude Design — <project URL>
- File(s):        <Name>.dc.html
- Import method:  fallback-zip            # MVP; "connector" reserved for the deferred enhancement
- Original zip:   design/<slug>.zip
- Local artifact: design/<slug>/<Name>.dc.html
- Captured:       <YYYY-MM-DD>
- Covers:         <which screens / flows this informs>
- Note:           ux.md is canon; this design is a linked aid, not the spec.
```
No new state file, no schema enforcement — plain Markdown the Dev Bot reads.

## System Architecture — the zip flow
1. User downloads the design as a `.zip` from Claude Design and drops it in the project (anywhere; tells the bot the path).
2. During `/ux`, the design-reference trigger fires → skill procedure: normalize the zip to `design/<slug>.zip`, extract to `design/<slug>/`, identify the primary `*.dc.html`.
3. Bot writes the Design Reference block into `ux.md`. **No code is generated here.**
4. Later, `/dev` on a ticket that cites the Design Reference reads the extracted `*.dc.html` (+ assets) and implements against it.

The connector path (D2) is **not built** — documented as a deferred enhancement (T15) gated on OQ-6.

## Implementation sequence
`T10 (skill) → { T11 (/ux), T12 (/dev) parallel-ok } → T13 (docs/templates/sync) → T14 (release)`. T10 first because T11/T12 reference the skill's procedure; T11 and T12 touch different command files, so they can run in parallel after T10. T15 (connector enhancement) is deferred — not in this release.

## v2.1.0 Assumptions
- Claude Design's "Download zip instead" bundle contains a self-contained `*.dc.html` (+ assets) that's implementable offline.
- The agent can extract a zip with host tools (cross-platform note in the skill — OQ-9).
- Users commit `design/` to git (recommended, not enforced).
- A plain-Markdown Design Reference block needs no validation.

## v2.1.0 Open Questions
- **OQ-7 (this stage):** `design/<slug>/` + retained `design/<slug>.zip`; primary artifact = the `.dc.html` — **confirm at gate.**
- **OQ-8 (minor):** create `design/` on-first-use (proposed) vs at `/5bot-init`.
- **OQ-9 (minor):** cross-platform extract — skill gives both PowerShell + unix commands; agent picks (proposed).
- **OQ-6 (DEFERRED):** real `claude_design` connect/auth path per surface — only needed for the deferred connector enhancement (T15).

## v2.1.0 Tickets
See `dev-qa.md` Backlog (T10–T15). Summary:
- **T10 — Skill: Design Reference procedure** (SKILL.md): block schema + locate/extract/record steps + guardrails. Foundation for T11/T12.
- **T11 — `/ux` design-reference step** (commands/ux.md): optional Flow-D trigger → run the skill procedure → write the block; silent no-op when no design.
- **T12 — `/dev` consumes the design** (commands/dev.md): if the ticket cites a Design Reference, read the linked `*.dc.html`; never auto-build the whole design.
- **T13 — Docs, template & `_framework` sync:** README (zip workflow + `design/` convention), `templates/ux.md` optional-section note, sync personas/ux + dev to `_framework/5bot/`.
- **T14 — v2.1.0 release:** bump `plugin.json` + root `marketplace.json` (2.0.0→2.1.0), root README/CLAUDE.md/MEMORY, `git push origin main` (dual-push → both repos), tag `v2.1.0` + push tags to both (per `PUBLISH.md`).
- **T15 — (DEFERRED, post-MVP) connector import (D2):** the `claude_design` MCP path; gated on OQ-6. Not in v2.1.0.

---

## Implementation Plan (v1.2.1 — shipped; historical record)

**v1.2.1 patch: Three clarity improvements**

All changes are **Markdown edits only**. No code, no schema changes, no new files.

### Phase 1: Template Improvements (2 edits)

**Edit 1: Handoff.md template**
- Location: Template file → copied into projects at `/5bot-init`
- Change: Add prominent ⚠️ warning at the top
- Why: Users confuse this with permanent state

**Edit 2: Project-state.md template**
- Location: Template file
- Change: Add definition after "Canon" heading
- Why: "Canon" is unexplained jargon

### Phase 2: Command Clarity (8 edits)

**Edit 3-10: Command files**
- Files: `commands/product.md`, `commands/ux.md`, `commands/architect.md`, `commands/dev.md`, `commands/qa.md`, `commands/5bot-init.md`, `commands/gate.md`, `commands/handoff.md`
- Change: Add 2-3 sentence rule summary to each command (after the role introduction)
- Why: Users don't know the scope boundaries

Example text to add:
> **Scope rules:** Product Bot defines scope. UX Bot cannot change it. Architect Bot cannot expand it. Dev Bot implements it. All scope changes require human approval at a gate. Read `project-state.md` first to know the current stage.

### Phase 3: Documentation (2 edits, future work)

**Edit 11: README.md — Field Validation Checklist**
- Add "Before Running Each Command" section
- Checklist items: (e.g., "before /product: have you decided the problem you're solving?", "before /dev: is a ticket assigned in project-state.md?")
- Why: Reduce friction for new users; catch mistakes early

**Edit 12: Research — Gate Enforcement**
- Investigate: Can we add a "stage lock" to `project-state.md` that prevents running commands out of order?
- Decision: Defer to v1.3.0; not critical for v1.2.1

## Tickets

All tickets are Markdown edits. No dependencies. Can be developed in parallel.

---

### T1: Add ⚠️ Warning to Handoff.md Template

**Goal:** Prevent users from editing `handoff.md` thinking it's permanent state.

**Requirements (from UX audit):**
- Handoff.md transient nature must be crystal clear
- Users must know where to put decisions (decisions.md) and state (project-state.md)

**Acceptance Criteria:**
- [ ] Template file has prominent ⚠️ warning at top
- [ ] Warning text says: "DO NOT EDIT. This file is overwritten each stage. Put decisions in decisions.md, state in project-state.md."
- [ ] Warning is visible before any template content
- [ ] File is tested by running `/5bot-init` in a test project and confirming warning appears

**Files likely involved:**
- `C:\AI Bins\Code\_framework\5bot\templates\handoff.md` (master template)
- Verify: Projects created with `/5bot-init` inherit the change

**Out of scope:**
- No code changes
- No validation enforcement
- No UI changes (pure Markdown)

---

### T2: Add "Canon" Definition to Project-state.md Template

**Goal:** Explain the jargon so new users understand `project-state.md` is the source of truth.

**Requirements (from UX audit):**
- New users don't know what "canon" means
- Definition must fit in a one-liner

**Acceptance Criteria:**
- [ ] Template has definition after "Canon. Every bot reads it first." heading
- [ ] Definition: "Canon = source of truth. All other files reference this one."
- [ ] Definition is placed prominently (top of file, after the introductory comment)
- [ ] Tested by reviewing template clarity with fresh eyes

**Files likely involved:**
- `C:\AI Bins\Code\_framework\5bot\templates\project-state.md` (master template)

**Out of scope:**
- No changes to the structure of the file
- No changes to section headings

---

### T3: Add Rule Summary to 8 Command Files

**Goal:** Teach users the anti-drift rules in context (each command tells them what they own, what they don't).

**Requirements (from UX audit):**
- Command prompts assume knowledge of rules
- Users need a 2-3 sentence summary of scope boundaries per role

**Acceptance Criteria:**
- [ ] All 8 command files have rule summary added (product, ux, architect, dev, qa, 5bot-init, handoff, gate)
- [ ] Summary explains the role's scope + what they can't do
- [ ] Example rule summary: "Product Bot defines scope. UX Bot cannot change it. Architect Bot cannot expand it. Dev Bot implements it. All scope changes require human approval at a gate."
- [ ] Summary is placed after the role introduction, before the task instructions
- [ ] Each command is tested by running it in Claude Code and confirming clarity improvement

**Files likely involved:**
- `C:\AI Bins\Code\plugins\five-bot\commands\product.md`
- `C:\AI Bins\Code\plugins\five-bot\commands\ux.md`
- `C:\AI Bins\Code\plugins\five-bot\commands\architect.md`
- `C:\AI Bins\Code\plugins\five-bot\commands\dev.md`
- `C:\AI Bins\Code\plugins\five-bot\commands\qa.md`
- `C:\AI Bins\Code\plugins\five-bot\commands\5bot-init.md`
- `C:\AI Bins\Code\plugins\five-bot\commands\handoff.md`
- `C:\AI Bins\Code\plugins\five-bot\commands\gate.md`

**Out of scope:**
- No changes to the core task instructions
- No changes to persona descriptions

---

### T4: Update README with Validation Checklist (Future — v1.2.2)

**Goal:** Provide users with a pre-flight checklist for each command.

**Requirements (from risk mitigation):**
- Users should validate state before running commands
- Reduce confusion about what to fill in before running each command

**Acceptance Criteria:**
- [ ] README has "Before Running Each Command" section
- [ ] Checklist for each command (8 total)
- [ ] Example: "Before running `/product`: Have you read the existing project description? Is the problem statement clear?"
- [ ] Checklist is tested by new users following it

**Files likely involved:**
- `C:\AI Bins\Code\plugins\five-bot\README.md`

**Out of scope:**
- No technical enforcement (this is documentation only)
- No changes to the workflow itself

---

### T5: Research Gate Enforcement for v1.3.0 (Future — v1.3.0)

**Goal:** Explore technical enforcement of human gates (prevent skipping stages).

**Requirements (from risk mitigation):**
- Can we add a "stage lock" to `project-state.md` that validates command order?
- What would this look like? (e.g., a `CanRunCommands` field that only certain commands can edit?)

**Acceptance Criteria:**
- [ ] Document feasibility (1-2 page analysis)
- [ ] Propose design (how would stage lock work? what breaks if someone skips a stage?)
- [ ] Estimate effort for implementation in v1.3.0
- [ ] Decide: Worth doing? Or rely on rules + human gates?

**Files likely involved:**
- Design document (new file or append to architecture.md)

**Out of scope:**
- Actually implementing stage lock (design phase only)
- Any code changes

---

## Summary

**What stays the same:**
- 8 slash commands work as-is
- 7 state file templates are structurally unchanged
- User workflow is unchanged (Product → UX → Architect → Dev → QA)
- No new dependencies or integrations

**What improves (v1.2.1):**
- Handoff.md clarity (⚠️ warning)
- Project-state.md clarity ("canon" definition)
- Command clarity (rule summaries in all 8 commands)
- README documentation (validation checklist, future)

**What's researched (v1.3.0 planning):**
- Technical gate enforcement (stage lock design)

**Assumptions:**
- Users have git for version control (not enforced by plugin, just recommended)
- Markdown editing is acceptable (no visual UI needed)
- Anti-drift rules + human gates are sufficient for now (no technical enforcement in v1.2.1)

**Open questions:**
- Should stage lock be prioritized for v1.3.0, or defer further?
- Are there other clarity improvements beyond these three?

---

# Architecture (v2.2.0 re-scoped — git sync-awareness)

> Build spec for the read-only **git-awareness line** in `/5bot-status` (re-scope approved 2026-06-29; Architect gate skipped under the 2-gate default). Markdown-only — the "code" is git-command instructions in `commands/5bot-status.md`. Supersedes the API-usage architecture below.
>
> **Tech:** no new deps, no network, no credentials, no mutation. The command runs read-only git commands via the host shell (git is cross-platform — same commands on Windows/unix; only invocation differs, handled by the agent's shell):
> - repo check `git rev-parse --is-inside-work-tree`; branch `git symbolic-ref --quiet --short HEAD` (else `detached HEAD`); sync `git rev-list --left-right --count @{u}...HEAD` (no `@{u}` → `no upstream`; label counts `(last fetch)` — no fetch performed); cleanliness `git status --porcelain`.
> - **Omit the line silently on any error / non-repo / git absent.** Informational only; never alters the next-command recommendation; writes nothing.
> - Placement: last line of the freshness footer (per `ux.md` E1 re-scoped). **Ticket: T2** (see `dev-qa.md`).
> - Stays within "no external integrations" — never networks or uses credentials; local tooling only (Product OQ-1 ruling).

---

# Architecture (v2.2.0 · Usage-Aware Status Reporting) — SUPERSEDED (2026-06-29)

> ⛔ SUPERSEDED: the API-usage approach was reverted at the QA gate (model can't read usage %). See the git sync-awareness spec above. Retained for history.

> Canonical architecture artifact the Dev Bot reads. Plans the technical implementation of the read-only API usage status line in `/5bot-status`, approved at UX gate (2026-06-26). Resolves three blocking OQs: API feasibility, reset-time format, output placement.

## Tech Stack (v2.2.0)

**No new dependencies.** v2.2.0 uses the existing Claude Code plugin architecture:
- **Language:** Markdown (command definitions)
- **State storage:** Plain Markdown files (project-state.md, etc.)
- **API query source:** Claude's own context (via system prompt inspection, MCP context, or environment variable)
- **No external APIs or databases**

**Key implementation detail:** The `/5bot-status` command (Markdown-based prompt) must query for API usage data without blocking or erroring. This is accomplished by:
1. Attempting to retrieve usage data from available sources (Claude context, environment, MCP)
2. Silently omitting the usage line if retrieval fails or is unavailable
3. Never blocking the command; never showing error states

## System Architecture (v2.2.0)

**Change scope:** Minimal. Modify one existing command (`/5bot-status`) to add one optional data line.

**Command flow:**
```
User runs `/5bot-status`
  ↓
Command reads: project-state.md, decisions.md, handoff.md (unchanged)
  ↓
NEW: Attempt to query API hourly usage % + reset time
  ↓
Render output:
  - Stage, Active ticket (unchanged)
  - [NEW] Usage line (if data available)
  - Last decision, Open questions (unchanged)
  - Next command footer (unchanged)
  ↓
If usage query unavailable: silently omit the line (no error)
```

**No state changes.** The usage data is read-only, computed on-the-fly, and never persisted to `project-state.md` or other files.

## Decisions (v2.2.0) — Resolving the Three OQs

**OQ-1: API Feasibility (Can Claude Code expose usage data?)**

**RESOLVED: YES, via Claude context.** The `/5bot-status` command runs within Claude Code's inference context. The model can inspect its own context to determine remaining budget / usage via:
- System prompt hints (if Claude Code embeds usage info)
- Environment variables (if Claude Code sets them)
- Direct MCP query to a usage endpoint (if one exists)
- Fallback: graceful omission if none available

**Implementation approach:**
- The command attempts to query usage via the available mechanism (recommended: ask Claude itself via a prompt clause like *"Based on your context, what is the current API hourly usage percentage? Respond with a single percentage integer (0–100) or 'unavailable'."*)
- If the query returns a percentage, format and display: `Usage: API hourly usage: {percent}% (resets at {reset_time} UTC)`
- If the query returns "unavailable" or times out, silently omit the line

**Why this works:**
- Claude already knows its own usage (it's part of internal rate-limit enforcement)
- No external API credentials needed (no security risk)
- Graceful degradation if the mechanism changes or isn't available in a specific environment
- Matches the "portable across surfaces" principle (works in Code, Cowork, web as long as Claude has context)

**OQ-2: Reset Time Format**

**RESOLVED: UTC time (HH:MM UTC).** Example: "resets at 15:00 UTC"

**Rationale:**
- UTC is unambiguous and durable (doesn't change if user copies/pastes output hours later)
- Human-relative time ("in 15 minutes") is friendlier but breaks in copied output and requires timezone awareness
- Claude's hourly rolling window resets at "current hour + 60 min", so the UTC time is deterministic (ask Claude: *"In UTC, at what time does the hourly window reset?" — e.g., if it's 14:30 UTC now, reset is at 15:30 UTC*)

**Implementation:** Compute `reset_time = (now_hour + 1) % 24` in UTC, formatted as "HH:00 UTC" or ask Claude to provide it.

**OQ-3: Placement in `/5bot-status` Output**

**RESOLVED: After "Active ticket", before "Last decision".** This groups the usage line with decision-making context.

**Rationale:**
- User sees: Stage → Active Ticket → **Usage** → Last Decision → Open Questions → Next Command
- This flows as: "Where am I?" → "What am I working on?" → "Can I continue?" → "What did we last decide?" → "What's next?"
- Placing usage mid-output (not at the end) ensures users see it before deciding whether to run the next command
- Keeps `/5bot-status` output readable (~12 lines with usage, still brief enough for "fast re-orient")

## Implementation Plan (v2.2.0)

**One ticket: T1 — Add usage line to `/5bot-status`**

- **Goal:** Query API usage and display in `/5bot-status` output
- **Dev work:** ~20–30 min (modify command prompt + output formatting; add query logic)
- **QA work:** Test with usage data available and unavailable; verify silent fallback; check readability

**No architectural changes needed.** The plugin's modular Markdown-based design makes this a straightforward addition.

## Assumptions (v2.2.0)

- **ASSUMPTION:** Claude Code's inference context includes or can be queried for API usage percentage (not verified, but highly likely given rate-limit enforcement)
- **ASSUMPTION:** Reset time can be computed from current time + 60 minutes (rolling hourly window, not calendar hour)
- **ASSUMPTION:** Query is fast enough (< 500ms) so `/5bot-status` doesn't hang; if query is slow, it times out and the line is silently omitted
- **ASSUMPTION:** Users understand "76% of hourly quota" without further explanation; no additional guidance text needed

## Open Questions (v2.2.0)

**None blocking for Dev.** The three OQs are all resolved:
- OQ-1: Query via Claude context ✓
- OQ-2: UTC format ✓
- OQ-3: After Active ticket ✓

**Post-Dev questions (for QA/gate):**
- Does the usage query work reliably across Code/Cowork/web?
- Does the query timeout gracefully?
- Is the "resets at HH:MM UTC" time accurate?

## Decisions

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

## Dev-QA

_dev-qa.md sweep deferred from this pass (interleaved notes/reviews across versions); the next /handoff will roll shipped tickets, or run a careful follow-up._
