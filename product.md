# Product (v2.1.0 · shipped baseline v2.0.0)

## Product Name

5bot: A Claude Code Plugin for Disciplined Software Development Workflows

## One-Sentence Pitch

A Claude Code plugin that guides software teams through a structured Product → UX → Architect → Dev → QA workflow, with each stage as a slash command reading shared Markdown state and stopping at human approval gates.

## Problem

Software development with AI assistants often lacks structure:
- Unclear separation of concerns (Product defines scope, but then Dev changes it)
- Lost context between conversations (requirements drift, decisions get reversed)
- No clear approval points (hard to know what's actually been agreed to)
- Token waste from re-pasting state on every turn
- Difficult to hand off work between roles (PM → designer → architect → dev)

Existing approaches either over-automate (letting AI wander into all roles) or under-coordinate (treating it like a chat assistant).

## Target Users

**Primary:** Individual developers and small teams (1-10 people) building software using Claude Code
- Indie hackers, startup founders, solo consultants
- Small engineering teams wanting structure without ceremony
- Anyone who has struggled with "AI made decisions I didn't approve of"

**Secondary:** Larger teams using Claude Code as a supplemental development tool (pair programming, code review)

## User Goals

1. **Define work clearly** before building it (Product + UX gates)
2. **Make decisions once** and not have them silently reversed
3. **Hand off work cleanly** between roles without re-explaining context
4. **Keep token usage low** by sharing Markdown state instead of re-pasting
5. **Maintain a record** of what was approved and why (for reference, compliance, disputes)
6. **Run the workflow in any Claude interface** (Code, Projects, chat, CLI)

## Value Proposition

- **Discipline without ceremony:** Five clear roles with explicit gates; no ambiguity about who decides what
- **Markdown-native:** Works in any Claude interface, portable, version-controllable, human-readable
- **Token-efficient:** Bots read state files instead of re-pasting; each bot focuses on its stage only
- **Decision record:** Every approval and scope decision lives in git history
- **Proven patterns:** Framework has been tested by the author on real projects; comes with rules and templates

## MVP Features

The plugin currently ships with all these and they work:

1. **Five slash commands** (each a role, each reads state, produces artifact, stops at gate):
   - `/product` → Product Bot
   - `/ux` → UX Bot
   - `/architect` → Architect Bot
   - `/dev` → Developer Bot
   - `/qa` → Reviewer/QA Bot

2. **Setup and handoff commands:**
   - `/5bot-init` → scaffold the seven Markdown files into a project
   - `/handoff` → record what just changed and what's next
   - `/gate` → request human approval (produces a summary for the human)

3. **Embedded framework:**
   - Full `README.md` explaining the workflow
   - `rules.md` with anti-drift rules
   - Templated stage files (`project-state.md`, `decisions.md`, `product.md`, `ux.md`, `architecture.md`, `dev-qa.md`, `handoff.md`)

4. **Claude Code integration:**
   - Commands auto-load the framework rules into context
   - Skills (`five-bot:product`, `five-bot:ux`, etc.) for backward compatibility

## Out of Scope (Future / Recommended Later)

- **Visual design tools:** Currently the UX Bot works in Markdown only; integrating with Claude Design or Figma is a later enhancement
- **External integrations:** GitHub issue creation, Jira sync, Slack notifications
- **Multi-project orchestration:** Each project runs independently
- **Metrics/analytics:** No dashboard or reporting on project health
- **Team collaboration UI:** All state is Markdown on disk
- **Custom role variants:** Locked to five roles; six-role or three-role pipelines are future
- **LLM model selection:** Users choose via Claude Code settings

## Success Criteria

✓ **Released as a public Claude Code plugin** (shipped; https://github.com/AngryMunky/5bot-plugin)

✓ **All commands work and users report the workflow feels disciplined**

✓ **Markdown state is human-readable and survives version control**

✓ **Users can run the workflow in Code, Projects, or chat** (portable)

✓ **Humans actually respect approval gates** (not silently bypassed)

Future metrics:
- Users adopt it for multi-month projects
- Feedback confirms scope creep and decision reversals are prevented
- Community contributions to personas or frameworks

## Risks

1. **Over-scoping:** Temptation to add features (visual mockups, integrations) that bloat the core. **Mitigation:** Keep five roles + Markdown; integrations are plugins-on-plugins.

2. **Context exhaustion:** Large projects could make state files too large to auto-load. **Mitigation:** Archive old decisions; keep project-state.md short.

3. **Adoption friction:** Users expect "just build it," not approval gates. **Mitigation:** Clear messaging that this is for structured teams.

4. **Bot drift:** Commands may not follow anti-drift rules (Dev changing scope, QA approving incomplete work). **Mitigation:** Personas include explicit rules; humans catch violations at gates.

5. **Version lock-in:** Projects created with v1.1.0 could break on v2.0.0. **Mitigation:** Semantic versioning and clear migration guides.

6. **Unclear handoff:** Users confuse transient handoff.md with permanent state files. **Mitigation:** Better docs and enforcement.

## Open Questions

1. **Should the plugin ship with personas for non-software domains?** (grants, curriculum, legal). **ASSUMPTION:** Start software-only; others are extensions.

2. **Do users want a visual dashboard?** **ASSUMPTION:** No; Markdown is portable and sufficient.

3. **Auto-lint the Markdown files via CI?** **ASSUMPTION:** No; users own their process.

4. **Multi-project namespacing in one workspace?** **ASSUMPTION:** One workspace per project.

5. **Should Dev/QA be a single looping command?** **ASSUMPTION:** Separate is right; humans choose when to loop.

---

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
