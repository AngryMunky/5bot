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

# Architecture (v2.3.0 — Lean Context)

> Build spec for Lean Context (UX gate APPROVED 2026-06-29; Architect gate skipped). Markdown-only, no new deps. Resolves OQ-3 (lazy-read mechanism) + OQ-4 (rollover mechanics).

## OQ-3 — lazy-read mechanism: anchored/section reads (NOT file-splitting)
Keep the single working files; bots read only the live slice via offset/grep — `/dev` reads the active ticket block, stage bots read the current version section, `/5bot-status` reads the newest decision. The big win is archive rollover (removing history) + read-by-section, **not** sub-file proliferation (which adds upkeep). **Decision: anchored reads + archive split; no per-ticket/per-version sub-files.**

## OQ-4 — rollover mechanics (runs inside `/handoff`, after the state update)
Deterministic + conservative; "current version" read from `project-state.md`:
- **Stage files** (product/ux/architecture): move any version section *older* than the current version → `archive.md` § Stage history. Keep current only.
- **decisions.md:** keep the newest **8** blocks; older → `archive.md` § Decisions.
- **dev-qa.md:** a ticket that is **DONE and whose version has shipped** → move its full record (card + dev notes + QA review) → `archive.md` § Dev-QA. Keep active/open tickets.
- `archive.md` created on first rollover (append-only); ensure the one-line pointer in each trimmed file; `/handoff` prints the one-line L3 note listing what moved.

## DRY
The rollover procedure + `archive.md` schema live once in `skills/five-bot/SKILL.md`. Both `/handoff` (incremental, at a seam) and `/5bot-archive` (retroactive full sweep) call it — same rules, different trigger.

## Sequence
`T1 (archive schema + rollover in SKILL.md/handoff) → T2 (anchored/section reads) → T3 (/5bot-archive) → T4 (/5bot-status note) → T5 (templates + _framework sync + README + release)`. T1 is foundational (T3 reuses its logic).

## Assumptions / Risks
- Bots already offset/grep-read reliably (no new capability). Risk: mis-archiving live content → conservative rules + git-recoverable + archive one read away. Two-copies + `_framework` sync handled at T5.

## Tickets
See `dev-qa.md` → "v2.3.0 Backlog" (T1–T5).

---


_Superseded version sections → archive.md § Stage history._
