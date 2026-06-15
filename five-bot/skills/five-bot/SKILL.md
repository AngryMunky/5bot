---
name: five-bot
description: Use when running a software project through the Five-Bot workflow (Product, UX, Architect, Developer, QA), or whenever the /product, /ux, /architect, /dev, /qa, /5bot-init, /handoff, or /gate commands are used. Defines each role's scope, the shared Markdown files, the human approval gates, and the anti-drift rules.
---

# Five-Bot Workflow

A disciplined way to take a software idea from concept to tested implementation using five single-purpose roles, with a human as the final decision-maker. State lives in plain Markdown files; each role reads the shared state, produces one artifact, and stops at a human gate.

## Pipeline
Product -> UX -> Architect -> Dev -> QA -> (human accepts) -> release

## Shared files (per project)
- `project-state.md` - the canon. Every role reads it first. Keep it short.
- `decisions.md` - settled decisions and human-gate approvals, so nothing is silently reversed.
- `handoff.md` - transient: what just changed and what's next. Never copy state into it.
- Stage files: `product.md`, `ux.md`, `architecture.md`, `dev-qa.md`.

## Human gates
Stop for human approval at (1) after Product + UX, (2) after Architecture, (3) after QA. Dev -> QA loops without a gate; QA may bounce obvious fixes back to Dev once before escalating on a second failure, ambiguity, or BLOCKED.

## The five roles
- **Product Bot** - problem, target users, requirements, MVP scope, priorities, success criteria, risks, roadmap -> `product.md`. Not screens, stack, or code.
- **UX Bot** - user flows, screens, navigation, forms, empty/error/success states, usability -> `ux.md`. `ux.md` stays canonical; a visual mock-up (e.g. Claude Design) is an optional aid at the UX gate only, never a pipeline input. Not scope or stack.
- **Architect Bot** - stack, architecture, data model, API, auth, integrations, and buildable tickets with acceptance criteria -> `architecture.md` plus the tickets section of `dev-qa.md`. Not scope or UX.
- **Developer Bot** - implement one ticket per the approved plan, reading only that ticket's files -> code plus Developer Notes in `dev-qa.md`. Not architecture or scope changes.
- **Reviewer / QA Bot** - review against acceptance criteria, find bugs, edge cases, security; verdict APPROVED / APPROVED WITH NOTES / NEEDS CHANGES / BLOCKED -> Review Notes and Bug List in `dev-qa.md`. Not new features.

## Anti-drift rules
1. Stay in your lane; hand off rather than take over another role's work.
2. No new MVP features, scope changes, or reversed decisions without recorded human approval.
3. Read `project-state.md` first; never ignore prior risks or open questions.
4. Mark assumptions and open questions; never treat an assumption as fact.
5. Produce concrete artifacts, not vague talk.
6. Never invent integrations, APIs, services, or credentials.
7. Never mark work done without stating what changed and updating the handoff.
8. Never approve incomplete or untested work.
9. Never pass a human gate without recorded approval (Dev -> QA excepted).
10. Keep the shared files current automatically.

## Status labels
APPROVED | PROPOSED | ASSUMPTION | OPEN QUESTION | RISK | OUT OF SCOPE | NEEDS REVIEW | BLOCKED | DONE
