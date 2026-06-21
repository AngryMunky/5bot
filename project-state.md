# Project State (v1.2.1)

> Canon. Every bot reads this first. Keep it short — detail lives in the stage files.

## Project Name

5bot

## One-Sentence Concept

A Claude Code plugin providing a disciplined Product → UX → Architect → Dev → QA workflow where each role is a slash command that reads shared Markdown state, produces one artifact, and stops at a human approval gate.

## Target Users

Individual developers and small teams (1–10 people) using Claude Code. Indie hackers, startup founders, consultants, and teams wanting structured development without ceremony.

## Current Stage

Releasing v1.3.0 (final gate APPROVED; local edits + version bump done; staging-branch push pending authorization)

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

- Visual design tools (Figma, Claude Design integration)
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

## Current Active Ticket

**T9 (release) in progress.** T6/T7/T8 implemented + QA-APPROVED. Local edits done (plugin.json → 1.3.0, README, root CLAUDE, MEMORY). Remaining: staging-branch push → install-verify → merge to `main`. Also pending: T4 (v1.2.2 backlog), T5 (gate-enforcement research).

**Deployment note:** Updating the source of the currently-installed 5bot plugin. Post-build, stage v1.3.0 on a non-default ("hidden") branch and install from it to verify before merging to `main` (folded into T9).

## Major Decisions (Product / UX / Technical)

**PENDING:** Recorded in `decisions.md`.

## Open Questions

**RESOLVED at v1.3.0 UX gate (2026-06-21):**
- ✓ OQ-1: "long" = bot self-judgment (confirmed)
- ✓ OQ-2: `/5bot-status` stays brief (no Known Risks section)

**RESOLVED at v1.3.0 Product gate (2026-06-21):**
- ✓ Bundling: bundle T5 in the *design* initiative, but ship F1–F3 separately as v1.3.0 (stage-lock = later, own gate)
- ✓ Cowork: verify in human's environment as a QA task (Claude walks through it)
- ✓ "Long" threshold: bot self-judgment in Dev/QA personas
- ✓ `/5bot-status`: new dedicated read-only command

**RESOLVED by prior human approval:**
- ✓ Non-software domains: DEFER (not priority now)
- ✓ Visual dashboard: DEFERRED (only if native to Claude Code/Cowork)
- ✓ Auto-lint Markdown: APPROVED (do this)
- ✓ Multi-project namespacing: RESOLVED (current folder-per-project model is correct)
- ✓ Dev/QA separation: APPROVED (keep separate, no looping variant)

## Known Risks

1. Over-scoping with features (visual tools, integrations)
2. State file context exhaustion in large projects
3. Adoption friction (users expecting "just build it")
4. Bot drift (commands not following anti-drift rules)
5. Version lock-in between releases
6. Unclear handoff semantics for new users

See `product.md` for mitigation strategies.

## Last Updated / By

2026-06-21 / Final gate APPROVED → T9 release in progress (staging push pending authorization)
