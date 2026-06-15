# The Five-Bot Framework (v1.1.0)

A disciplined way to take a software idea from rough concept to tested implementation using a small team of single-purpose AI roles, with a human as the final decision-maker. The whole system runs on plain Markdown files, so it is portable across tools and easy to read, edit, and version-control.

This guide is self-contained. Hand it to anyone; they can stand up the same workflow from it.

---

## The idea in one paragraph

Five "bots" run in sequence, each doing exactly one job: a **Product Bot** defines what's being built, a **UX Bot** designs how users move through it, a **Technical Architect Bot** plans the build, a **Developer Bot** writes the code one ticket at a time, and a **Reviewer / QA Bot** checks the work. The bots share state through a handful of Markdown files instead of long conversations, which keeps each one focused and keeps token use low. A human approves at three gates. No bot is allowed to wander outside its role or quietly change a settled decision.

---

## The pipeline

```text
Product → UX → Architect → Dev → QA → (human accepts) → release
```

Each stage produces a concrete artifact — a file, a spec, a set of tickets, a review — never a vague chat.

---

## The five roles

**Product Bot** — Defines the problem, target users, requirements, and MVP scope. Separates must-haves from later features. Does not design screens, choose technology, or write code.

**UX Bot** — Turns approved requirements into user flows, a screen list, navigation, forms, and empty/error/success states. Does not change scope or pick a stack. `ux.md` is its canonical output; a visual tool (e.g. Claude Design) may optionally be used at the UX gate to mock up screens as an aid for the human's review, but the mock-up is never a pipeline input — if kept, it's exported into the repo and linked from `ux.md`.

**Technical Architect Bot** — Recommends the stack, designs the data model and architecture, plans the API and auth, and breaks the work into buildable tickets with acceptance criteria. Does not expand the MVP or write production code.

**Developer Bot** — Implements one ticket at a time against the approved plan, reading only the files that ticket needs. Documents what changed. Does not change architecture or scope.

**Reviewer / QA Bot** — Checks the work against the ticket's acceptance criteria, finds bugs and edge cases, and returns a verdict: APPROVED, APPROVED WITH NOTES, NEEDS CHANGES, or BLOCKED. Does not add features.

---

## The shared files

State lives in seven Markdown files per project. Three are shared across all stages; four hold the detailed work.

**Shared (read by every bot):**

- `project-state.md` — the canon. Short. Every bot reads it first. Holds the current truth: concept, target users, current stage, approved MVP, active ticket, major decisions, open questions, risks.
- `decisions.md` — the log of settled decisions and human-gate approvals, so nothing gets silently reversed.
- `handoff.md` — transient. What just changed and what the next bot should do. Overwritten each stage. Never duplicates state.

**Stage files (detailed work):**

- `product.md` — product brief, requirements, MVP, roadmap.
- `ux.md` — flows, screens, interactions, states.
- `architecture.md` — stack, data model, API, plan, tickets.
- `dev-qa.md` — backlog, developer notes, review notes, bug list, release checklist.

The golden rule: a fact lives in one place. Detail belongs in the stage files; the canon stays short.

---

## The human gates

The human is the decision-maker, not a micromanager. Approval is required at three points:

1. **After Product + UX** — what we're building and how the user moves through it. (Split into two gates for large projects.)
2. **After Architecture** — stack, data model, build plan.
3. **After QA** — accept the work, request changes, or send it back.

The Developer → QA loop runs without a human gate; QA may bounce obvious fixes back to the Developer once on its own. The human is pulled in only on a second failure, on ambiguity, or when something is BLOCKED. At each gate the human responds with one of: APPROVED, APPROVED WITH CHANGES, REVISE, REJECT, DEFER, NEEDS CLARIFICATION.

---

## The anti-drift rules

These keep the bots honest. Every bot follows all of them.

1. Stay in your lane. If a task belongs to another bot, leave a note — don't take it over.
2. No new MVP features, scope changes, or reversed decisions without recorded human approval.
3. Always read `project-state.md` before acting. Never ignore prior risks or open questions.
4. Mark assumptions and open questions explicitly. Never treat an assumption as fact.
5. Produce concrete artifacts, not vague conversation.
6. Never invent integrations, APIs, services, or credentials.
7. Never mark work done without stating what changed and updating the handoff.
8. Never approve incomplete or untested work.
9. Never proceed past a human gate without recorded approval (Dev → QA is the only exception).
10. Keep the shared files current automatically — bookkeeping is not optional.

---

## Setting it up in Claude Code (recommended)

This is the lowest-friction way to run the framework, because the bots read and write the shared files directly on disk — no copying anything between chats.

**One-time setup (per machine):**

1. Create a shared framework folder, e.g. `_framework/5bot/`, containing `rules.md`, a `personas/` folder (one file per bot), and a `templates/` folder (blank copies of the seven files).
2. Add slash commands in `.claude/commands/`: `5bot-init`, `product`, `ux`, `architect`, `dev`, `qa`, `handoff`, `gate`. Each command tells Claude Code to adopt the matching persona, read the canon and latest handoff, do the work, and stop at the gate.

**Per project:**

1. Run `/5bot-init` inside the project. It copies the seven templates in and adds two import lines to the project's `CLAUDE.md`:
   ```text
   @../_framework/5bot/rules.md
   @project-state.md
   ```
   These make the rules and the canon load automatically on every run.
2. Run `/product` and describe the idea. Approve at each gate. Continue with `/ux`, `/architect`, `/dev`, `/qa`.

Because the rules and current state auto-load, you never paste them. You only ever provide the new task.

---

## Running it without Claude Code (chat or Projects)

The framework is just Markdown, so it works anywhere — it's only more manual.

- **Claude Projects:** put the seven shared files in the Project's knowledge base and save each persona as the Project's custom instructions (one Project per bot, or switch the instructions per stage). The files load automatically; you paste only the task.
- **Plain chat (any assistant):** keep the files yourself and paste, per turn, only `project-state.md`, the latest `handoff.md`, and the files the current role needs. This is the most token-hungry option — avoid passing the whole project to any bot.

---

## A note on "office agents"

This framework is shaped for software development, but the skeleton — **Define → Approve → Design → Approve → Build → Review → Human Decision** — generalizes to any structured knowledge work: grant proposals, curriculum packages, multi-stage document production. To repurpose it, keep the shared files and the gates, and swap the five personas for stage-appropriate roles (e.g. Brief → Draft → Review → Format). The discipline is the product; the specific roles are interchangeable.

---

*Five-Bot Framework v1.1.0. Built on Markdown so it outlives any single tool.*
