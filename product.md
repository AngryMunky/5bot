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

# PROPOSED v2.3.0 — Lean Context (lazy loading + archive rollover)

> Human-requested (2026-06-29) consolidation of 5bot's own context usage. Combines two levers: **#3 lazy/section loading** (load only the live slice) + **#1 archive rollover** (relocate history off the read path). Idea **#2** (per-file budgets + summarize) was **rejected** — lossy + high upkeep. Kept deliberately short (dogfooding the goal).

## Problem
As a project runs, the shared Markdown accumulates history — every version's product/ux/architecture sections, all decisions, all shipped tickets + dev notes + QA reviews — and bots reload the whole mass each turn though only the current slice matters. **Exhibit A: this very project** (`product.md`/`dev-qa.md`/`decisions.md` carry v1.3.0→v2.2.0). Effect: rising token cost, noisier context, faster context exhaustion (already a top known risk).

## Target Users
All 5bot users; especially long-running / multi-version projects and teams carrying a project across many sessions (where history compounds).

## Goals
1. Bots load only what the current stage/ticket needs.
2. History preserved + auditable, but off the default read path.
3. Near-zero added user interaction; **nothing destroyed** (relocate, don't summarize).
4. Context cost stays ~flat as a project ages (pre-emptive).

## MVP Scope (combined #3 + #1)
- **Lazy loading (#3):** bots read only the live slice — the current version section of product/ux/architecture, the active ticket in dev-qa, the newest few decisions — via file conventions (current vs archived) + read-by-section discipline in personas/commands.
- **Archive rollover (#1):** on `/handoff`, stale content auto-relocates to append-only archives never loaded unless explicitly asked — superseded version sections → `history.md` (or per-stage `*-archive.md`), decisions beyond the last N → `decisions-archive.md`, shipped tickets → `dev-qa-archive.md`. Working files keep only live content.
- **Non-destructive + discoverable:** archives are relocations, never deletes/summaries; each working file carries a one-line pointer to its archive; `/5bot-status` notes when archives exist.

## Out of Scope (this round)
- Lossy summarization / hard per-file trim budgets (idea #2) — revisit later.
- Automatic token measurement / live usage display (the reverted v2.2.0 API path — model can't read usage).
- Changing the canonical file set beyond adding archives (`project-state.md` stays the canon).
- Auto-migrating *existing* bloated projects (offer a one-time pass / manual guidance; full auto-migration later).

## Priority & Versioning
Additive framework improvement → **MINOR, v2.3.0**. Touches personas/commands/rules + templates, and must sync the two copies (plugin + `_framework`). No breaking change (archives are additive; legacy projects fall back to "unlocked + warn").

## Success Criteria
- On a multi-version project, a typical bot turn loads materially less; working files stay ~flat in size regardless of project age.
- No history lost — everything relocated stays retrievable.
- Rollover is automatic on `/handoff`; users don't hand-manage archives.
- `project-state.md` remains the single canon.

## Risks
1. **Mis-archiving live content.** Mitigate: conservative rules (keep current version section + last-N decisions + active tickets); archives are one read away.
2. **Two-copies + `_framework` sync** (templates/personas change). Mitigate: release ticket carries the sync criterion (as v2.1.0/v2.2.0 did).
3. **Existing-project migration** (files already bloated). Mitigate: one-time archive pass + legacy "unlocked + warn".
4. **Discoverability** (where did history go?). Mitigate: pointer line per file + `/5bot-status` archive note.

## Open Questions — resolved at Product gate (2026-06-29) except where noted
- ✓ **OQ-1:** APPROVED — Lean Context = #3 lazy-load + #1 archive rollover; **#2 excluded**.
- ✓ **#3 lazy loading:** approved.
- ✓ **OQ-2 (Product rec):** a **single `archive.md`**, off the read path, sectioned (Stage history / Decisions / Dev-QA). (Alt: per-kind archives.)
- ✓ **OQ-4 (Product rec):** conservative thresholds — stage files keep the *current* version section only; `decisions.md` keeps the last ~8 blocks; `dev-qa.md` archives a ticket's record once it is DONE + its version shipped. Rollover at `/handoff` + release sweep.
- ✓ **OQ-5 (Product rec):** include a one-time **`/5bot-archive`** command in MVP (retroactive, idempotent, non-destructive).
- **OQ-3 (→ Architect):** lazy-read mechanism — file-splitting vs anchored/offset reads vs both.
- *Recommendations carried to UX/Architect; human invited to tweak any.*

## Recommendation
**PROPOSED.** Recommend APPROVE as v2.3.0: directly cuts the top known risk (context exhaustion), is non-destructive, and is mostly automatic (hands-off). Route: Product gate → `/ux` → `/architect` → `/dev` → `/qa` → gate → release v2.3.0.

---


_Superseded version sections → archive.md § Stage history._
