# Project State (v1.2.1)

> Canon. Every bot reads this first. Keep it short — detail lives in the stage files.

## Project Name

5bot

## One-Sentence Concept

A Claude Code plugin providing a disciplined Product → UX → Architect → Dev → QA workflow where each role is a slash command that reads shared Markdown state, produces one artifact, and stops at a human approval gate.

## Target Users

Individual developers and small teams (1–10 people) using Claude Code. Indie hackers, startup founders, consultants, and teams wanting structured development without ceremony.

## Current Stage

Released (v1.2.1)

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

**All v1.2.1 tickets APPROVED:** T1 ✅ T2 ✅ T3 ✅ Ready for final human acceptance and release.

## Major Decisions (Product / UX / Technical)

**PENDING:** Recorded in `decisions.md`.

## Open Questions

**RESOLVED by human approval:**
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

2026-06-21 / Human (Final acceptance APPROVED) → v1.2.1 released
