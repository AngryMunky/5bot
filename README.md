# 5bot - Disciplined AI Software Development Workflow

**Version 2.0.0** | [Full documentation](5bot/README.md)

A Claude Code plugin that brings structure and discipline to software development using five single-purpose AI roles with human approval gates. The whole system runs on plain Markdown, so it's portable, version-control friendly, and easy to audit.

## What is 5bot?

5bot automates a proven workflow: **Product → UX → Architect → Dev → QA → Human Decision → Release**

Each role (Product Bot, UX Bot, Architect Bot, Developer Bot, QA Bot) is a slash command that:
- Reads shared Markdown project state
- Does one specific job (nothing more)
- Produces a concrete artifact (spec, design, plan, code, review)
- Stops at a human approval gate

**Key benefits:**
- ✅ Clear scope boundaries (no silent feature creep)
- ✅ Auditable decisions (all stored in `decisions.md`)
- ✅ Token efficient (bots read only what they need)
- ✅ Portable (runs in any Claude interface)
- ✅ Git-friendly (all state is plain Markdown)

## Quick start

**1. Install the plugin:**
```
/plugin marketplace add AngryMunky/5bot
/plugin install 5bot@lawson-design
```

**2. In any project folder, run:**
```
/5bot-init
/product "describe your idea"
```

**3. Then follow the workflow:**
- `/ux` — design user flows (human approves at gate)
- `/architect` — plan the build (human approves at gate)  
- `/dev T1` — implement ticket T1
- `/qa` — review the work
- Repeat `/dev` / `/qa` for each ticket
- `/gate` — human final approval and release

## The nine commands

| Command | Does |
|---------|------|
| `/5bot-init` | Scaffold the project with templates |
| `/product` | Define scope, users, MVP, roadmap |
| `/ux` | Design flows, screens, forms, interactions |
| `/architect` | Plan stack, data model, break into tickets |
| `/dev` | Implement one ticket (write code, docs) |
| `/qa` | Review work, find bugs, return verdict |
| `/handoff` | Update state files after each stage |
| `/gate` | Human review and approval |
| `/5bot-status` | Read-only snapshot to re-orient after `/compact` or a fresh session |

## Workflow diagram

```
Product Bot → Gate ✓ → UX Bot → Gate ✓ → Architect Bot → Gate ✓
                                                             ↓
Release ← Gate ✓ ← [Dev → QA loop repeats for each ticket] ← Start coding
```

## The shared state files

All project state lives in seven Markdown files (git-friendly, versionable):

- **`project-state.md`** — The canon (single source of truth)
- **`decisions.md`** — Log of all approvals and major decisions
- **`handoff.md`** — Transient summary (what just changed, what's next)
- **`product.md`** — Product brief and scope
- **`ux.md`** — UX design and user flows
- **`architecture.md`** — Build plan and ticket breakdown
- **`dev-qa.md`** — Work tracker (tickets, code, reviews, bugs)

## Target audience

- **Indie developers** wanting structure without ceremony
- **Small teams** (1–10 people) needing clear roles and decision audit trails
- **Anyone working across multiple AI chats** and needing persistent state
- **Teams fighting scope creep** and wanting to prevent silent decisions

## Key features

✨ **v2.0.0 — Pipeline Awareness & Guardrails:**
- 🧭 `/5bot-status`: read-only snapshot to re-orient after a compaction, fresh session, or time away
- 💾 Context-health reminder: after a long session, nudges you to `/compact` — reassuring that all canon is on disk, so nothing is lost
- ➡️ Next-command footer: `/handoff` and `/gate` always name the exact next command

**v1.2.1:**
- ⚠️ Handoff warning: Templates warn users `handoff.md` is transient
- 📝 "Canon" definition: Templates clarify `project-state.md` is the source of truth
- 📋 Rule summaries: All commands explain their scope and anti-drift rules upfront

## Full documentation

See [`5bot/README.md`](5bot/README.md) for:
- Detailed role descriptions
- The ten anti-drift rules (that keep bots honest)
- How to run it without Claude Code (Projects, plain chat)
- Customization for non-software workflows
- And much more

## Repository

- **GitHub:** https://github.com/AngryMunky/5bot
- **License:** MIT
- **Author:** Chad Lawson

---

*Built on Markdown so it outlives any single tool. v2.0.0*
