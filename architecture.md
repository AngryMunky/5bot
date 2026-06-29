# Architecture (v2.1.0 · v1.2.1 record retained below)

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

# v2.0.0 — Pipeline Awareness & Guardrails (current)

> Renumbered from v1.3.0 to **v2.0.0** at release: completing the `five-bot` → `5bot` install rename is a breaking change (existing `five-bot@lawson-design` installs must reinstall as `5bot@lawson-design`), so per the semver rule it's a MAJOR bump. The features themselves (F1/F2/F3) are unchanged.

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

# v2.1.0 — Claude Design Integration (zip-first)

> Build plan for the optional Claude Design **design-reference** step (Product gate 2026-06-22; refined at the UX gate to **zip-first** — the `.zip` download is the MVP path, live `claude_design` connector import is deferred). Native Claude Design only. Turns `ux.md` "v2.1.0" (Flow D, D1–D5) into tickets. Paths use the current `plugins/5bot/` layout.

## Tech Stack (delta)
Unchanged: Markdown-only plugin, plain-text state, no network integrations. **New interaction:** the workflow now reads a user-provided local `.zip` (Claude Design "Download zip instead") and its extracted contents. Extraction uses **host tools the agent already has** (PowerShell `Expand-Archive` on Windows; `unzip`/`tar` on Unix) — local file I/O, not a service integration, so the "no integrations / no external deps" stance holds.

## Key decision — OQ-7: where the design lives (PROPOSED, confirm at gate)
- Extracted design lives at **`design/<slug>/`** in the downstream project (`<slug>` = kebab-cased design file name; `Tag Hag.dc.html` → `design/tag-hag/`).
- Keep the **original `.zip`** at `design/<slug>.zip` for provenance (pristine, re-extractable source).
- **Primary artifact** Dev builds against = the extracted `…/<name>.dc.html`; the `ux.md` Design Reference block links it.
- Rationale: `design/` is conventional and git-friendly; extracted HTML is diffable; the retained zip preserves the exact export.
- **OQ-8 (minor):** create `design/` on-first-use (proposed) vs at `/5bot-init` → propose on-first-use (no empty-dir churn).

## DRY placement (one home, like F1/F3)
The **`five-bot` skill** (`skills/five-bot/SKILL.md`) is the single home for the **Design Reference procedure**: the block schema, the zip locate→extract→record steps (with the cross-platform extract note), and the guardrails ("`ux.md` is canon; design is a linked aid; never auto-build the whole design; capture is UX-only"). `commands/ux.md` (capture) and `commands/dev.md` (consume) carry only short triggers pointing at the skill.

## Data Model — the Design Reference block (the contract)
A new `## Design Reference (Claude Design)` section is written into the project's **`ux.md`** (canon). Schema:
```
## Design Reference (Claude Design)
- Source:         Claude Design — <project URL>
- File(s):        <Name>.dc.html
- Import method:  fallback-zip            # MVP; "connector" reserved for the deferred enhancement
- Original zip:   design/<slug>.zip
- Local artifact: design/<slug>/<Name>.dc.html
- Captured:       <YYYY-MM-DD>
- Covers:         <which screens / flows this informs>
- Note:           ux.md is canon; this design is a linked aid, not the spec.
```
No new state file, no schema enforcement — plain Markdown the Dev Bot reads.

## System Architecture — the zip flow
1. User downloads the design as a `.zip` from Claude Design and drops it in the project (anywhere; tells the bot the path).
2. During `/ux`, the design-reference trigger fires → skill procedure: normalize the zip to `design/<slug>.zip`, extract to `design/<slug>/`, identify the primary `*.dc.html`.
3. Bot writes the Design Reference block into `ux.md`. **No code is generated here.**
4. Later, `/dev` on a ticket that cites the Design Reference reads the extracted `*.dc.html` (+ assets) and implements against it.

The connector path (D2) is **not built** — documented as a deferred enhancement (T15) gated on OQ-6.

## Implementation sequence
`T10 (skill) → { T11 (/ux), T12 (/dev) parallel-ok } → T13 (docs/templates/sync) → T14 (release)`. T10 first because T11/T12 reference the skill's procedure; T11 and T12 touch different command files, so they can run in parallel after T10. T15 (connector enhancement) is deferred — not in this release.

## v2.1.0 Assumptions
- Claude Design's "Download zip instead" bundle contains a self-contained `*.dc.html` (+ assets) that's implementable offline.
- The agent can extract a zip with host tools (cross-platform note in the skill — OQ-9).
- Users commit `design/` to git (recommended, not enforced).
- A plain-Markdown Design Reference block needs no validation.

## v2.1.0 Open Questions
- **OQ-7 (this stage):** `design/<slug>/` + retained `design/<slug>.zip`; primary artifact = the `.dc.html` — **confirm at gate.**
- **OQ-8 (minor):** create `design/` on-first-use (proposed) vs at `/5bot-init`.
- **OQ-9 (minor):** cross-platform extract — skill gives both PowerShell + unix commands; agent picks (proposed).
- **OQ-6 (DEFERRED):** real `claude_design` connect/auth path per surface — only needed for the deferred connector enhancement (T15).

## v2.1.0 Tickets
See `dev-qa.md` Backlog (T10–T15). Summary:
- **T10 — Skill: Design Reference procedure** (SKILL.md): block schema + locate/extract/record steps + guardrails. Foundation for T11/T12.
- **T11 — `/ux` design-reference step** (commands/ux.md): optional Flow-D trigger → run the skill procedure → write the block; silent no-op when no design.
- **T12 — `/dev` consumes the design** (commands/dev.md): if the ticket cites a Design Reference, read the linked `*.dc.html`; never auto-build the whole design.
- **T13 — Docs, template & `_framework` sync:** README (zip workflow + `design/` convention), `templates/ux.md` optional-section note, sync personas/ux + dev to `_framework/5bot/`.
- **T14 — v2.1.0 release:** bump `plugin.json` + root `marketplace.json` (2.0.0→2.1.0), root README/CLAUDE.md/MEMORY, `git push origin main` (dual-push → both repos), tag `v2.1.0` + push tags to both (per `PUBLISH.md`).
- **T15 — (DEFERRED, post-MVP) connector import (D2):** the `claude_design` MCP path; gated on OQ-6. Not in v2.1.0.

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

---

# Architecture (v2.2.0 re-scoped — git sync-awareness)

> Build spec for the read-only **git-awareness line** in `/5bot-status` (re-scope approved 2026-06-29; Architect gate skipped under the 2-gate default). Markdown-only — the "code" is git-command instructions in `commands/5bot-status.md`. Supersedes the API-usage architecture below.
>
> **Tech:** no new deps, no network, no credentials, no mutation. The command runs read-only git commands via the host shell (git is cross-platform — same commands on Windows/unix; only invocation differs, handled by the agent's shell):
> - repo check `git rev-parse --is-inside-work-tree`; branch `git symbolic-ref --quiet --short HEAD` (else `detached HEAD`); sync `git rev-list --left-right --count @{u}...HEAD` (no `@{u}` → `no upstream`; label counts `(last fetch)` — no fetch performed); cleanliness `git status --porcelain`.
> - **Omit the line silently on any error / non-repo / git absent.** Informational only; never alters the next-command recommendation; writes nothing.
> - Placement: last line of the freshness footer (per `ux.md` E1 re-scoped). **Ticket: T2** (see `dev-qa.md`).
> - Stays within "no external integrations" — never networks or uses credentials; local tooling only (Product OQ-1 ruling).

---

# Architecture (v2.2.0 · Usage-Aware Status Reporting) — SUPERSEDED (2026-06-29)

> ⛔ SUPERSEDED: the API-usage approach was reverted at the QA gate (model can't read usage %). See the git sync-awareness spec above. Retained for history.

> Canonical architecture artifact the Dev Bot reads. Plans the technical implementation of the read-only API usage status line in `/5bot-status`, approved at UX gate (2026-06-26). Resolves three blocking OQs: API feasibility, reset-time format, output placement.

## Tech Stack (v2.2.0)

**No new dependencies.** v2.2.0 uses the existing Claude Code plugin architecture:
- **Language:** Markdown (command definitions)
- **State storage:** Plain Markdown files (project-state.md, etc.)
- **API query source:** Claude's own context (via system prompt inspection, MCP context, or environment variable)
- **No external APIs or databases**

**Key implementation detail:** The `/5bot-status` command (Markdown-based prompt) must query for API usage data without blocking or erroring. This is accomplished by:
1. Attempting to retrieve usage data from available sources (Claude context, environment, MCP)
2. Silently omitting the usage line if retrieval fails or is unavailable
3. Never blocking the command; never showing error states

## System Architecture (v2.2.0)

**Change scope:** Minimal. Modify one existing command (`/5bot-status`) to add one optional data line.

**Command flow:**
```
User runs `/5bot-status`
  ↓
Command reads: project-state.md, decisions.md, handoff.md (unchanged)
  ↓
NEW: Attempt to query API hourly usage % + reset time
  ↓
Render output:
  - Stage, Active ticket (unchanged)
  - [NEW] Usage line (if data available)
  - Last decision, Open questions (unchanged)
  - Next command footer (unchanged)
  ↓
If usage query unavailable: silently omit the line (no error)
```

**No state changes.** The usage data is read-only, computed on-the-fly, and never persisted to `project-state.md` or other files.

## Decisions (v2.2.0) — Resolving the Three OQs

**OQ-1: API Feasibility (Can Claude Code expose usage data?)**

**RESOLVED: YES, via Claude context.** The `/5bot-status` command runs within Claude Code's inference context. The model can inspect its own context to determine remaining budget / usage via:
- System prompt hints (if Claude Code embeds usage info)
- Environment variables (if Claude Code sets them)
- Direct MCP query to a usage endpoint (if one exists)
- Fallback: graceful omission if none available

**Implementation approach:**
- The command attempts to query usage via the available mechanism (recommended: ask Claude itself via a prompt clause like *"Based on your context, what is the current API hourly usage percentage? Respond with a single percentage integer (0–100) or 'unavailable'."*)
- If the query returns a percentage, format and display: `Usage: API hourly usage: {percent}% (resets at {reset_time} UTC)`
- If the query returns "unavailable" or times out, silently omit the line

**Why this works:**
- Claude already knows its own usage (it's part of internal rate-limit enforcement)
- No external API credentials needed (no security risk)
- Graceful degradation if the mechanism changes or isn't available in a specific environment
- Matches the "portable across surfaces" principle (works in Code, Cowork, web as long as Claude has context)

**OQ-2: Reset Time Format**

**RESOLVED: UTC time (HH:MM UTC).** Example: "resets at 15:00 UTC"

**Rationale:**
- UTC is unambiguous and durable (doesn't change if user copies/pastes output hours later)
- Human-relative time ("in 15 minutes") is friendlier but breaks in copied output and requires timezone awareness
- Claude's hourly rolling window resets at "current hour + 60 min", so the UTC time is deterministic (ask Claude: *"In UTC, at what time does the hourly window reset?" — e.g., if it's 14:30 UTC now, reset is at 15:30 UTC*)

**Implementation:** Compute `reset_time = (now_hour + 1) % 24` in UTC, formatted as "HH:00 UTC" or ask Claude to provide it.

**OQ-3: Placement in `/5bot-status` Output**

**RESOLVED: After "Active ticket", before "Last decision".** This groups the usage line with decision-making context.

**Rationale:**
- User sees: Stage → Active Ticket → **Usage** → Last Decision → Open Questions → Next Command
- This flows as: "Where am I?" → "What am I working on?" → "Can I continue?" → "What did we last decide?" → "What's next?"
- Placing usage mid-output (not at the end) ensures users see it before deciding whether to run the next command
- Keeps `/5bot-status` output readable (~12 lines with usage, still brief enough for "fast re-orient")

## Implementation Plan (v2.2.0)

**One ticket: T1 — Add usage line to `/5bot-status`**

- **Goal:** Query API usage and display in `/5bot-status` output
- **Dev work:** ~20–30 min (modify command prompt + output formatting; add query logic)
- **QA work:** Test with usage data available and unavailable; verify silent fallback; check readability

**No architectural changes needed.** The plugin's modular Markdown-based design makes this a straightforward addition.

## Assumptions (v2.2.0)

- **ASSUMPTION:** Claude Code's inference context includes or can be queried for API usage percentage (not verified, but highly likely given rate-limit enforcement)
- **ASSUMPTION:** Reset time can be computed from current time + 60 minutes (rolling hourly window, not calendar hour)
- **ASSUMPTION:** Query is fast enough (< 500ms) so `/5bot-status` doesn't hang; if query is slow, it times out and the line is silently omitted
- **ASSUMPTION:** Users understand "76% of hourly quota" without further explanation; no additional guidance text needed

## Open Questions (v2.2.0)

**None blocking for Dev.** The three OQs are all resolved:
- OQ-1: Query via Claude context ✓
- OQ-2: UTC format ✓
- OQ-3: After Active ticket ✓

**Post-Dev questions (for QA/gate):**
- Does the usage query work reliably across Code/Cowork/web?
- Does the query timeout gracefully?
- Is the "resets at HH:MM UTC" time accurate?
