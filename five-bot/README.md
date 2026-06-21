# The 5bot Framework (v1.3.0)

A disciplined way to take a software idea from rough concept to tested implementation using a small team of single-purpose AI roles, with a human as the final decision-maker. The whole system runs on plain Markdown files, so it is portable across tools and easy to read, edit, and version-control.

This guide is self-contained. Hand it to anyone; they can stand up the same workflow from it.

---

## What is 5bot?

**5bot** is a Claude Code plugin that automates the 5-bot software development workflow. Instead of freestyle conversations, each role (Product, UX, Architect, Dev, QA) is a slash command that:
- Reads shared Markdown project state
- Performs its specific task
- Produces a concrete artifact (spec, design, architecture, code, review)
- Stops at a human approval gate

**Perfect for:**
- Indie developers and small teams (1–10 people) who want structured development without ceremony
- Projects that need clear scope boundaries and decision audit trails
- Teams working across multiple AI chat sessions (all state lives in git-friendly Markdown)
- Reducing decision debt and scope creep

**Install in Claude Code:**
```
/plugin marketplace add AngryMunky/5bot-plugin
/plugin install 5bot@lawson-design
```

Then in any project folder, run:
```
/5bot-init
/product
```

to start the workflow.

---

## Key concepts (jargon explained)

**Canon** — The single source of truth. In 5bot, `project-state.md` is the canon. All other files reference it, never the reverse. This prevents conflicting versions of the truth floating around.

**Artifact** — A concrete output from a bot. Not a vague chat. Examples: a written spec (product.md), a design doc (ux.md), a build plan (architecture.md), working code, a review report. Each bot must produce one.

**Handoff** — A transient summary file that says "what just happened, what's next, who should read what." It's overwritten every stage; it's never permanent state. Think of it as sticky notes, not the filing cabinet.

**Gate** — A human decision point. After Product+UX, after Architect, and after all QA work, the human reviews and approves (or rejects) before the next stage runs. This prevents silent scope creep.

**Bot** — An AI role (Product, UX, Architect, Dev, QA). Each bot is a slash command. Each reads shared state, does its job, and stops at a gate. No wandering off scope.

**Persona** — The instructions that tell a bot how to think and what to do. Each bot has a persona file that explains its job, constraints, and how to read/update the shared state.

**Scope** — What's in the MVP (must build now) vs. out of scope (later, or never). The Product Bot defines scope; no other bot can expand it without human approval. This prevents feature creep.

**Anti-drift rules** — Ten rules that keep bots honest. Rule #1: stay in your lane. Rule #2: no scope changes without recorded human approval. Rule #9: never skip a human gate. These enforce discipline.

**MVP** — Minimum Viable Product. The smallest set of features that solves the core problem for the target users. Everything else is "later" or "never."

---

## The idea in one paragraph

Five "bots" run in sequence, each doing exactly one job: a **Product Bot** defines what's being built, a **UX Bot** designs how users move through it, a **Technical Architect Bot** plans the build, a **Developer Bot** writes the code one ticket at a time, and a **Reviewer / QA Bot** checks the work. The bots share state through a handful of Markdown files instead of long conversations, which keeps each one focused and keeps token use low. A human approves at three gates. No bot is allowed to wander outside its role or quietly change a settled decision.

---

## The bot hierarchy and commands

```
┌─────────────────────────────────────────────────────────┐
│                 Human starts here                        │
│              /product [describe idea]                    │
└──────────────────────┬──────────────────────────────────┘
                       ↓
           ┌───────────────────────────┐
           │  🤖 Product Bot           │
           │  /product                 │
           │  ↓ produces: product.md   │
           └───────────────┬───────────┘
                           ↓
                  ┌─────────────────┐
                  │ 🚪 GATE 1       │
                  │ /gate           │
                  │ Human approves  │
                  │ scope & MVP     │
                  └────────┬────────┘
                           ↓
           ┌───────────────────────────┐
           │  🤖 UX Bot               │
           │  /ux                      │
           │  ↓ produces: ux.md        │
           └───────────────┬───────────┘
                           ↓
                  ┌─────────────────┐
                  │ 🚪 GATE 2       │
                  │ /gate           │
                  │ Human approves  │
                  │ flows & design  │
                  └────────┬────────┘
                           ↓
           ┌───────────────────────────┐
           │  🤖 Architect Bot         │
           │  /architect               │
           │  ↓ produces:              │
           │    architecture.md        │
           │    dev-qa.md (tickets)    │
           └───────────────┬───────────┘
                           ↓
                  ┌─────────────────┐
                  │ 🚪 GATE 3       │
                  │ /gate           │
                  │ Human approves  │
                  │ stack & plan    │
                  └────────┬────────┘
                           ↓
           ┌───────────────────────────┐
           │  🤖 Developer Bot         │
           │  /dev [ticket T1]         │
           │  ↓ produces: code +       │
           │    dev notes              │
           └───────────────┬───────────┘
                           ↓
           ┌───────────────────────────┐
           │  🤖 QA Bot               │
           │  /qa [review verdict]     │
           │  ↓ produces: review notes │
           │    + bug list             │
           └───────────────┬───────────┘
                           ↓
              (repeats dev+qa for next ticket)
                           ↓
                  ┌─────────────────┐
                  │ 🚪 GATE 4       │
                  │ /gate           │
                  │ Human approves  │
                  │ all work        │
                  └────────┬────────┘
                           ↓
                    Release! 🚀
```

**Key insight:** Each bot does exactly one job and stops. No scope creep, no silent decisions.

---

## The nine slash commands

| Command | Bot | Job | Reads | Produces |
|---------|-----|-----|-------|----------|
| `/5bot-init` | Setup | Copy templates and scaffold the project | — | `project-state.md`, `decisions.md`, `handoff.md`, `product.md`, `ux.md`, `architecture.md`, `dev-qa.md` |
| `/product` | Product Bot | Define what's being built: problem, users, scope, MVP, roadmap | `project-state.md` | `product.md` |
| `/ux` | UX Bot | Design how users move through it: flows, screens, forms, states | `product.md`, `project-state.md` | `ux.md` |
| `/architect` | Architect Bot | Plan the build: stack, data model, API, and break into tickets | `product.md`, `ux.md`, `project-state.md` | `architecture.md`, `dev-qa.md` (backlog) |
| `/dev` | Developer Bot | Implement one ticket per run; document changes | `architecture.md`, `dev-qa.md`, ticket files | Code + `dev-qa.md` (dev notes) |
| `/qa` | QA Bot | Review work against acceptance criteria; find bugs; return verdict | Ticket + code changes + `dev-qa.md` | `dev-qa.md` (review notes, bug list) |
| `/handoff` | Bookkeeper | Update state files after each stage: what changed, what's next, active ticket | Current stage files | `handoff.md` (transient), `project-state.md`, `decisions.md` |
| `/gate` | Human | Review stage summary; approve, request changes, or reject. Record decision. | `project-state.md`, `handoff.md`, relevant artifacts | `decisions.md` (approval block), `project-state.md` (new stage) |
| `/5bot-status` | — | Read-only re-orientation snapshot: stage, active ticket, last decision, and the one next command. Run it after `/compact`, a fresh session, or time away | `project-state.md`, `decisions.md`, `handoff.md` | Nothing (read-only) |

**Workflow pattern:**
1. Run a bot command (e.g., `/product "build a todo app"`)
2. It reads shared state, does its job, produces an artifact
3. Run `/handoff` to update state files
4. Run `/gate` to request human approval (unless it's Dev→QA, which auto-loops)
5. If approved, continue to the next bot
6. Repeat until shipped

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

## The shared files (where state lives)

All state is stored in seven Markdown files per project. These are the "filing cabinet" for the project — your source of truth, decision log, and work tracker rolled into one, all in plain text (easy to version in git).

**Shared files (read by every bot before it acts):**

- **`project-state.md`** — **The canon.** This is THE source of truth. Every bot reads it first. It's kept short and holds only the current facts: what are we building, who is it for, what stage are we in, what's the MVP, which ticket is active now, what major decisions have we made, what are the open questions, what are the risks. When in doubt, read this file.

- **`decisions.md`** — The decision log. Every human-gate approval is recorded here (what was approved, by whom, on what date, any conditions). Also records major technical decisions. This prevents "wait, did we approve that?" arguments later.

- **`handoff.md`** — The transient sticky note. Overwritten every stage. Says "we just did X, we produced Y, Z is the next step." Never contains permanent state. Think of it like a post-it note on the filing cabinet, not a file in it.

**Stage files (the detailed work):**

- **`product.md`** — The product brief. Problem statement, target users, requirements, MVP feature list, what's explicitly out of scope, roadmap, risks.

- **`ux.md`** — The design spec. User flows, screen list, navigation model, form fields, empty/error/success states, interaction details. Can include a link to a visual mock-up, but the Markdown is the real spec.

- **`architecture.md`** — The build plan. Tech stack, data model, API design, auth plan, system architecture, and the ticket breakdown (what gets built in what order).

- **`dev-qa.md`** — The work tracker. Backlog of tickets, developer notes (what was built per ticket), QA review notes (bugs found, verdict), and release checklist.

**The golden rule:** A fact lives in exactly one place. If it's permanent state (scope, decision, design), it goes in the stage files or canon. If it's "what just changed," it goes in the handoff and then gets incorporated into the canon. This prevents conflicting versions of the truth.

---

## The human gates

The human is the decision-maker, not a micromanager. Approval is required at three points:

1. **After Product + UX** — what we're building and how the user moves through it. (Split into two gates for large projects.)
2. **After Architecture** — stack, data model, build plan.
3. **After QA** — accept the work, request changes, or send it back.

The Developer → QA loop runs without a human gate; QA may bounce obvious fixes back to the Developer once on its own. The human is pulled in only on a second failure, on ambiguity, or when something is BLOCKED. At each gate the human responds with one of: APPROVED, APPROVED WITH CHANGES, REVISE, REJECT, DEFER, NEEDS CLARIFICATION.

---

## The anti-drift rules (discipline without overhead)

These ten rules keep the bots (and humans) honest. They prevent scope creep, silent decisions, and wasted work. Every role—Product, UX, Architect, Dev, QA, and Human—follows all of them.

1. **Stay in your lane.** If a task belongs to another bot's job, flag it but don't do it. Product Bot doesn't design screens. UX Bot doesn't pick the tech stack. Dev Bot doesn't change architecture.

2. **No scope changes without recorded human approval.** Can't add a feature to the MVP. Can't remove one. Can't reverse a prior decision. Everything goes through a gate.

3. **Always read `project-state.md` before acting.** It's the canon. Never ignore what it says about current stage, approved MVP, active ticket, or known risks.

4. **Mark assumptions and open questions explicitly.** Don't treat a guess as a fact. Write "ASSUMPTION: users have fast internet" or "OPEN QUESTION: should we support mobile?" so the next person knows it's unsettled.

5. **Produce concrete artifacts, not vague conversation.** A spec file, not "sounds good." A design doc, not "I think we should." A review verdict, not "seems OK." Artifacts are reviewable and versionable.

6. **Never invent integrations, APIs, services, or credentials.** Don't assume Stripe exists or that we have AWS access. If it's needed, the human approves it at a gate; then it gets added to scope and architecture.

7. **Never mark work done without stating what changed and updating the handoff.** The next bot needs to know: what files did you touch? what's the verdict? what should they read? Write it down.

8. **Never approve incomplete or untested work.** QA doesn't say "APPROVED" for code that doesn't run. Dev doesn't say a ticket is done if it doesn't match acceptance criteria.

9. **Never proceed past a human gate without recorded approval.** The human reviews, approves or rejects, and the decision gets logged in `decisions.md`. The only exception: Dev → QA loops without a gate (QA can auto-bounce obvious fixes back to Dev once).

10. **Keep the shared files current automatically.** Don't ask. It's not optional. After each stage, `project-state.md`, `decisions.md`, and `handoff.md` must be updated to reflect the new truth.

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

---

## Recent improvements

**v1.3.0 — Pipeline Awareness & Guardrails**
- **`/5bot-status`:** a read-only snapshot to re-orient after compaction, a fresh session, or time away — current stage, active ticket, last decision, and the single next command.
- **Context-health reminder:** after a long session, the workflow nudges you to `/compact` (or start fresh) at a natural seam — reassuring you that all canon is on disk, so nothing is lost.
- **Next-command footer:** `/handoff` and `/gate` always end by naming the exact next command, so you never have to remember the pipeline order.

**v1.2.1**
- **⚠️ Handoff warning:** Template warns users that `handoff.md` is transient and overwritten each stage
- **"Canon" definition:** Template clarifies that `project-state.md` is the single source of truth
- **Command rule summaries:** All commands explain their scope and anti-drift rules upfront

---

*5bot Framework v1.3.0. Built on Markdown so it outlives any single tool.*
