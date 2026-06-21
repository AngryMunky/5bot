# Architecture (v1.3.0 · v1.2.1 record retained below)

## Tech Stack

**Platform:** Claude Code plugin system
- **Language:** Markdown (commands, templates, documentation)
- **State storage:** Plain Markdown files on user's local disk (project-state.md, decisions.md, handoff.md, and stage files)
- **Command interface:** Slash commands in Claude Code (e.g., `/product`, `/ux`, `/architect`, `/dev`, `/qa`, `/5bot-init`, `/handoff`, `/gate`)
- **Skills interface:** Reusable skill definitions (`five-bot:product`, `five-bot:ux`, etc.) for role persona + instructions
- **Version control:** Git-friendly (all state is plain text, version-controllable, human-readable)
- **No external dependencies:** No APIs, no databases, no authentication, no integrations (by design)

## System Architecture

**High-level flow:**
```
User project
  ├── commands/ → 8 Markdown files (role prompts)
  ├── skills/ → 5bot skill (shared role definitions)
  ├── templates/ → 7 Markdown templates (scaffold a new project)
  └── Project state files (managed by user's git):
      ├── project-state.md (canon)
      ├── decisions.md (approval log)
      ├── handoff.md (transient, overwritten each stage)
      ├── product.md (written by Product Bot)
      ├── ux.md (written by UX Bot)
      ├── architecture.md (written by Architect Bot)
      └── dev-qa.md (written by Dev + QA Bots)
```

**Command flow (user perspective):**
1. User runs `/5bot-init` → scaffolds 7 templates into project root
2. User runs `/product` → Product Bot reads project-state.md + handoff.md, writes product.md, updates project-state.md
3. User runs `/gate` → formats approval summary, records decision in decisions.md, updates project-state.md
4. Repeat: next command reads prior bot's output, continues the workflow

**Command implementation (technical):**
- Each command file (e.g., `commands/product.md`) is a Markdown file with YAML frontmatter
- Frontmatter contains: `description` (shown in Claude Code UI), and the command body is the bot's persona + instructions
- When user runs `/product`, Claude Code loads `commands/product.md`, adopts the persona, and auto-loads context from:
  - `project-state.md` (canon)
  - `_framework/5bot/rules.md` (anti-drift rules, auto-loaded via CLAUDE.md)
  - The bot's instructions tell it which files to read/write

**No state machine / no enforcement:**
- Commands don't validate that previous stages were completed
- Users could run commands out of order (violates rules, but technically possible)
- **Mitigation:** Anti-drift rules + human gates enforce discipline

## Data Model

**Project state (all Markdown, no database):**

| File | Owner | Purpose | Size |
|------|-------|---------|------|
| `project-state.md` | All bots (read-only for most) | **Canon**. Current truth: concept, users, stage, MVP, risks, decisions | ~1-2 KB |
| `decisions.md` | /gate command | Log of all approvals + reversals. Human decisions recorded, never lost | ~1-5 KB |
| `handoff.md` | Each bot (overwritten each stage) | **Transient**. What just changed. Next bot reads it, then it's overwritten | ~0.5 KB |
| `product.md` | Product Bot | Problem, users, MVP, success criteria, risks, roadmap | ~2-5 KB |
| `ux.md` | UX Bot | User flows, screen list, navigation, forms, error states | ~2-10 KB |
| `architecture.md` | Architect Bot | Stack, system design, data model, API, auth, tickets, implementation plan | ~3-10 KB |
| `dev-qa.md` | Dev + QA Bots | Backlog/tickets, dev notes (per ticket), review notes, bug list, release checklist | ~5-20 KB |

**No database tables / schemas / API objects.** State is purely Markdown. Humans read and edit it. Git tracks changes.

## API Spec

**No API.** This is a plugin, not a service.

**Interface:**
- **Slash commands:** `/5bot-init`, `/product`, `/ux`, `/architect`, `/dev`, `/qa`, `/handoff`, `/gate`
- **File I/O:** Bots read/write Markdown files to the user's project directory
- **Skills:** Reusable role definitions for backward compatibility (e.g., `five-bot:product`)

## Auth & Permissions

**No authentication or permissions.**
- Plugin runs locally in Claude Code
- User has full access to their own project files
- All decisions are recorded in `decisions.md` (git history serves as an audit trail)

## Integrations

**None by design.**
- No GitHub, Jira, Slack, etc. (out of scope for v1.2.0)
- Future versions could add plugins-on-plugins, but core remains standalone

---

# v1.3.0 — Pipeline Awareness & Guardrails (current)

Turns `ux.md` (v1.3.0) into a build plan for **F1** (context reminder), **F2** (`/5bot-status`), **F3** (next-command footer). T5 (stage-lock) stays research — now informed by the `ux.md` "T5 hook notes."

## Key architectural decision: two synced copies

The framework exists in **two places** and both must stay in sync:
- **`plugins/five-bot/`** — the standalone, *published* plugin (commands/, skills/, README, plugin.json). **This is the canonical, shipped artifact.**
- **`_framework/5bot/`** — the in-repo working copy used for local dogfooding (rules.md, personas/, templates/).

**Decision:** Implement F1/F2/F3 **canonically in the plugin**, then **sync the relevant guidance into `_framework/5bot/`** so local runs match. Every ticket below carries a sync acceptance criterion. The published GitHub repo (`AngryMunky/5bot-plugin`) mirrors `plugins/five-bot/` plus root files; `plugin.json` and `.claude-plugin/marketplace.json` versions must match (see T9).

## DRY placement (avoid duplicating logic 4×)

- The **`five-bot` skill** (`skills/five-bot/SKILL.md`) is the single home for the canonical **F1 block** (reassurance + variants + heuristic + guardrails) and the **F3 footer shell + pipeline/verdict map**. Individual command files carry only a short *trigger* that points at the skill.
- `/5bot-status` (F2) reuses the **same successor logic** as F3 (defined once in the skill) to compute its single recommended next command.

## Implementation sequence (sequential — shared files)

`T6 (F3) → T7 (F2) → T8 (F1) → T9 (release)`. T6 first because F1 references "print above the F3 footer"; T6/T8 both edit `SKILL.md`, `handoff.md`, `gate.md`, so they must not run in parallel (same merge-conflict reasoning as the v1.2.1 T1–T3 sequencing). T7 is mostly a new file but also touches the SKILL command roster + README, so it's sequenced too.

## v1.3.0 Assumptions
- `/compact` is the relevant Claude Code command; copy also says "or start a fresh session" to stay valid if Cowork differs (verify at QA — the human has a Cowork env).
- Bots can self-judge "long" well enough; tuning is text-only (no structural change).
- The `five-bot` skill is loaded whenever a command runs, so centralizing F1/F3 text there is reliably in context.

## v1.3.0 Open Questions
- Does the published plugin's command set auto-expose a new `commands/5bot-status.md` on install, or must it also be added to `marketplace.json`/skill roster? (T7 verifies; T9 owns version/roster sync.)
- Exact wording of the skill's F1/F3 sections is Dev's to draft from the `ux.md` spec; QA checks against `ux.md`.

## v1.3.0 Tickets

See `dev-qa.md` Backlog for the full ticket cards (T6–T9). Summary:
- **T6 — F3 next-command footer:** skill defines the `--- / ▶ NEXT:` shell + verdict map; `handoff.md` (NEXT=/gate) and `gate.md` (waiting → resolved footer) reference it.
- **T7 — F2 `/5bot-status`:** new read-only `commands/5bot-status.md`; brief 6-section snapshot (no Known Risks per OQ-2); 5 edge cases; reuses F3 successor logic.
- **T8 — F1 context reminder:** skill defines reassurance + terse/minimal variants + self-judgment heuristic + guardrails; triggers in `handoff.md`/`gate.md` (terse, above F3) and `dev.md`/`qa.md` (minimal); handoff→gate and QA→gate suppression.
- **T9 — v1.3.0 release:** bump `plugin.json` + `marketplace.json` (1.2.1→1.3.0), document `/5bot-status` + F1/F3 in README, sync `_framework/5bot/`, update root CLAUDE.md + MEMORY.md, commit + tag.

---

## Implementation Plan (v1.2.1 — shipped; historical record)

**v1.2.1 patch: Three clarity improvements**

All changes are **Markdown edits only**. No code, no schema changes, no new files.

### Phase 1: Template Improvements (2 edits)

**Edit 1: Handoff.md template**
- Location: Template file → copied into projects at `/5bot-init`
- Change: Add prominent ⚠️ warning at the top
- Why: Users confuse this with permanent state

**Edit 2: Project-state.md template**
- Location: Template file
- Change: Add definition after "Canon" heading
- Why: "Canon" is unexplained jargon

### Phase 2: Command Clarity (8 edits)

**Edit 3-10: Command files**
- Files: `commands/product.md`, `commands/ux.md`, `commands/architect.md`, `commands/dev.md`, `commands/qa.md`, `commands/5bot-init.md`, `commands/gate.md`, `commands/handoff.md`
- Change: Add 2-3 sentence rule summary to each command (after the role introduction)
- Why: Users don't know the scope boundaries

Example text to add:
> **Scope rules:** Product Bot defines scope. UX Bot cannot change it. Architect Bot cannot expand it. Dev Bot implements it. All scope changes require human approval at a gate. Read `project-state.md` first to know the current stage.

### Phase 3: Documentation (2 edits, future work)

**Edit 11: README.md — Field Validation Checklist**
- Add "Before Running Each Command" section
- Checklist items: (e.g., "before /product: have you decided the problem you're solving?", "before /dev: is a ticket assigned in project-state.md?")
- Why: Reduce friction for new users; catch mistakes early

**Edit 12: Research — Gate Enforcement**
- Investigate: Can we add a "stage lock" to `project-state.md` that prevents running commands out of order?
- Decision: Defer to v1.3.0; not critical for v1.2.1

## Tickets

All tickets are Markdown edits. No dependencies. Can be developed in parallel.

---

### T1: Add ⚠️ Warning to Handoff.md Template

**Goal:** Prevent users from editing `handoff.md` thinking it's permanent state.

**Requirements (from UX audit):**
- Handoff.md transient nature must be crystal clear
- Users must know where to put decisions (decisions.md) and state (project-state.md)

**Acceptance Criteria:**
- [ ] Template file has prominent ⚠️ warning at top
- [ ] Warning text says: "DO NOT EDIT. This file is overwritten each stage. Put decisions in decisions.md, state in project-state.md."
- [ ] Warning is visible before any template content
- [ ] File is tested by running `/5bot-init` in a test project and confirming warning appears

**Files likely involved:**
- `C:\AI Bins\Code\_framework\5bot\templates\handoff.md` (master template)
- Verify: Projects created with `/5bot-init` inherit the change

**Out of scope:**
- No code changes
- No validation enforcement
- No UI changes (pure Markdown)

---

### T2: Add "Canon" Definition to Project-state.md Template

**Goal:** Explain the jargon so new users understand `project-state.md` is the source of truth.

**Requirements (from UX audit):**
- New users don't know what "canon" means
- Definition must fit in a one-liner

**Acceptance Criteria:**
- [ ] Template has definition after "Canon. Every bot reads it first." heading
- [ ] Definition: "Canon = source of truth. All other files reference this one."
- [ ] Definition is placed prominently (top of file, after the introductory comment)
- [ ] Tested by reviewing template clarity with fresh eyes

**Files likely involved:**
- `C:\AI Bins\Code\_framework\5bot\templates\project-state.md` (master template)

**Out of scope:**
- No changes to the structure of the file
- No changes to section headings

---

### T3: Add Rule Summary to 8 Command Files

**Goal:** Teach users the anti-drift rules in context (each command tells them what they own, what they don't).

**Requirements (from UX audit):**
- Command prompts assume knowledge of rules
- Users need a 2-3 sentence summary of scope boundaries per role

**Acceptance Criteria:**
- [ ] All 8 command files have rule summary added (product, ux, architect, dev, qa, 5bot-init, handoff, gate)
- [ ] Summary explains the role's scope + what they can't do
- [ ] Example rule summary: "Product Bot defines scope. UX Bot cannot change it. Architect Bot cannot expand it. Dev Bot implements it. All scope changes require human approval at a gate."
- [ ] Summary is placed after the role introduction, before the task instructions
- [ ] Each command is tested by running it in Claude Code and confirming clarity improvement

**Files likely involved:**
- `C:\AI Bins\Code\plugins\five-bot\commands\product.md`
- `C:\AI Bins\Code\plugins\five-bot\commands\ux.md`
- `C:\AI Bins\Code\plugins\five-bot\commands\architect.md`
- `C:\AI Bins\Code\plugins\five-bot\commands\dev.md`
- `C:\AI Bins\Code\plugins\five-bot\commands\qa.md`
- `C:\AI Bins\Code\plugins\five-bot\commands\5bot-init.md`
- `C:\AI Bins\Code\plugins\five-bot\commands\handoff.md`
- `C:\AI Bins\Code\plugins\five-bot\commands\gate.md`

**Out of scope:**
- No changes to the core task instructions
- No changes to persona descriptions

---

### T4: Update README with Validation Checklist (Future — v1.2.2)

**Goal:** Provide users with a pre-flight checklist for each command.

**Requirements (from risk mitigation):**
- Users should validate state before running commands
- Reduce confusion about what to fill in before running each command

**Acceptance Criteria:**
- [ ] README has "Before Running Each Command" section
- [ ] Checklist for each command (8 total)
- [ ] Example: "Before running `/product`: Have you read the existing project description? Is the problem statement clear?"
- [ ] Checklist is tested by new users following it

**Files likely involved:**
- `C:\AI Bins\Code\plugins\five-bot\README.md`

**Out of scope:**
- No technical enforcement (this is documentation only)
- No changes to the workflow itself

---

### T5: Research Gate Enforcement for v1.3.0 (Future — v1.3.0)

**Goal:** Explore technical enforcement of human gates (prevent skipping stages).

**Requirements (from risk mitigation):**
- Can we add a "stage lock" to `project-state.md` that validates command order?
- What would this look like? (e.g., a `CanRunCommands` field that only certain commands can edit?)

**Acceptance Criteria:**
- [ ] Document feasibility (1-2 page analysis)
- [ ] Propose design (how would stage lock work? what breaks if someone skips a stage?)
- [ ] Estimate effort for implementation in v1.3.0
- [ ] Decide: Worth doing? Or rely on rules + human gates?

**Files likely involved:**
- Design document (new file or append to architecture.md)

**Out of scope:**
- Actually implementing stage lock (design phase only)
- Any code changes

---

## Summary

**What stays the same:**
- 8 slash commands work as-is
- 7 state file templates are structurally unchanged
- User workflow is unchanged (Product → UX → Architect → Dev → QA)
- No new dependencies or integrations

**What improves (v1.2.1):**
- Handoff.md clarity (⚠️ warning)
- Project-state.md clarity ("canon" definition)
- Command clarity (rule summaries in all 8 commands)
- README documentation (validation checklist, future)

**What's researched (v1.3.0 planning):**
- Technical gate enforcement (stage lock design)

**Assumptions:**
- Users have git for version control (not enforced by plugin, just recommended)
- Markdown editing is acceptable (no visual UI needed)
- Anti-drift rules + human gates are sufficient for now (no technical enforcement in v1.2.1)

**Open questions:**
- Should stage lock be prioritized for v1.3.0, or defer further?
- Are there other clarity improvements beyond these three?
