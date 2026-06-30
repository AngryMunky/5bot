# Project State (v2.3.0)

> Canon. Every bot reads this first. Keep it short — detail lives in the stage files.

## Project Name

5bot

## One-Sentence Concept

A Claude Code plugin providing a disciplined Product → UX → Architect → Dev → QA workflow where each role is a slash command that reads shared Markdown state, produces one artifact, and stops at a human approval gate.

## Target Users

Individual developers and small teams (1–10 people) using Claude Code. Indie hackers, startup founders, consultants, and teams wanting structured development without ceremony.

## Current Active Ticket

**Released v2.3.0; Lean Context dogfooded.** Ran `/5bot-archive` on this project — relocated history to `archive.md` (product 481→185, ux 599→72, architecture 547→119, decisions 307→80; 21 decision blocks + 12 stage sections moved; verified nothing lost). Repos updated with the lean state + `archive.md` (chore — plugin unchanged at v2.3.0). `dev-qa.md` sweep deferred (interleaved across versions). Recommended: staging smoke test of the auto-rollover on `/handoff`. Open follow-ups: T4-legacy (v1.2.2 backlog), T5-legacy (gate-enforcement research), `task_2dbc68f5` (qa.md Test Plan fix), **dev-qa.md archive sweep**.

(Prior: **Architect gate APPROVED** 2026-06-26. **UX gate APPROVED** 2026-06-26. **Product gate APPROVED (revised scope)** 2026-06-26.)

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

- T4 (v1.2.2 backlog), T5 (gate-enforcement research), `task_2dbc68f5` (qa.md Test Plan fix).
- **`/ux` re-sync** of `ux.md` D2–D5 design-era copy to the as-built SKILL.md schema (non-blocking; recommended at the v2.1.0 QA gate).
- **PROPOSED → needs `/product` (suggest v2.2.0) — git-awareness.** *Problem:* across machines/sessions a user can start on a stale local copy and diverge (cf. the T14 release reconciliation). *Scope question for Product:* does local git-awareness count as the approved out-of-scope "External integrations (GitHub/Jira/Slack)", or is it acceptable local-tooling support? *Recommended minimal shape:* a **read-only**, best-effort git line in `/5bot-status` (e.g. "git: N behind origin · uncommitted changes") + an optional one-time `/5bot-init` nudge — **never** auto-`git pull`, **silent no-op when git/remote absent** (portability), never mutate the repo. Route: `/product` → `/architect` → ticket → gate. (Dev-drafted 2026-06-23; not approved scope.)

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

2026-06-29 / Dogfooded /5bot-archive — history → archive.md; repos updated (chore, v2.3.0 unchanged).
