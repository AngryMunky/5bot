# Project State (v2.1.0)

> Canon. Every bot reads this first. Keep it short — detail lives in the stage files.

## Project Name

5bot

## One-Sentence Concept

A Claude Code plugin providing a disciplined Product → UX → Architect → Dev → QA workflow where each role is a slash command that reads shared Markdown state, produces one artifact, and stops at a human approval gate.

## Target Users

Individual developers and small teams (1–10 people) using Claude Code. Indie hackers, startup founders, consultants, and teams wanting structured development without ceremony.

## Current Active Ticket

**v2.1.0 RELEASED 2026-06-23.** All tickets T10–T14 complete. v2.1.0 ships the optional Claude Design design-reference feature. T14 (release) approved at the QA gate: `plugin.json` + `marketplace.json` `5bot` entry → 2.1.0; READMEs + stage headers re-stamped; orphaned root `.claude-plugin/plugin.json` removed; dual-pushed (private `5bot-plugin` + public `AngryMunky/5bot`) + tagged `v2.1.0` from the canonical clone as Angry Munky. Open follow-up: recommended `/ux` re-sync of `ux.md` D2–D5 design-era copy. T15 (connector import) deferred. (Prior: v2.0.0 `main` `dbb0e1c`, tagged; installs as `5bot@lawson-design`.)

## Approved MVP Features (v1.2.0)

- Five slash commands (product, ux, architect, dev, qa)
- Setup (/5bot-init) and handoff (/handoff, /gate) commands
- Embedded framework (README, rules, templates)
- Claude Code integration (auto-loading, skills)
- Markdown-native, portable across Claude interfaces

## Approved v1.2.1 Patch Improvements (from UX audit)

- Add ⚠️ warning to handoff.md template (explain transient nature)
- Add "canon" definition to project-state.md template
- Add 2-3 sentence rule summary to 8 command files (clarify scope boundaries)

## Explicitly Out of Scope

- Third-party visual design tools (Figma, Sketch, etc.) — *native Claude Design is now in-scope as v2.1.0*
- External integrations (GitHub, Jira, Slack)
- Multi-project orchestration
- Metrics/analytics dashboards
- Team collaboration UI
- Custom role variants
- LLM model selection (user responsibility)

## Current Tech Stack

Claude Code plugin architecture
- Commands: slash command files in `commands/`
- Skills: skill definitions in `skills/`
- Framework: Markdown files auto-loaded by Claude Code
- Storage: User's local project directory (Markdown files + git)

## Open Follow-ups

T4 (v1.2.2 backlog), T5 (gate-enforcement research), `task_2dbc68f5` (qa.md Test Plan fix). *Proposed (pending T13 gate):* a `/ux` re-sync of `ux.md` D2–D5 to the as-built design schema.

## Deployment

Published as **two repos**. `AngryMunky/5bot-plugin` is **PRIVATE** so it appears in claude.ai's "Sync from GitHub" dialog (Cowork sync, source of truth). `AngryMunky/5bot` is a **PUBLIC mirror** (created 2026-06-21) for CLI installs — visibility is per-repo, not per-branch, so a public branch of a private repo isn't possible. The canonical clone dual-pushes `origin` to both (`git push origin main` → both); `git push public main` is the manual fallback. Public install: `/plugin marketplace add AngryMunky/5bot` → `/plugin install 5bot@lawson-design`.

## Major Decisions (Product / UX / Technical)

**PENDING:** Recorded in `decisions.md`.

## Open Questions

**None open.** v2.1.0 design-integration questions are all resolved (OQ-1/2/3/5/7/8/9, the last three confirmed at the Architect gate) or deferred (OQ-6 → T15, the connector path); earlier-release questions (v1.2.x / v1.3.0) are all resolved. Full history in `decisions.md`; v2.1.0 specifics in `architecture.md` / `product.md` / `ux.md` "v2.1.0".

## Known Risks

1. Over-scoping with features (visual tools, integrations)
2. State file context exhaustion in large projects
3. Adoption friction (users expecting "just build it")
4. Bot drift (commands not following anti-drift rules)
5. Version lock-in between releases
6. Unclear handoff semantics for new users

See `product.md` for mitigation strategies.

## Last Updated / By

2026-06-23 / Dev Bot — canon prune (removed stale duplicate Active-Ticket block; collapsed resolved Open Questions to a pointer). T13 still at QA gate.
