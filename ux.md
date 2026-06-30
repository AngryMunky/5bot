# UX (v2.1.0 — Claude Design reference)

> Supersedes the v1.2.0 UX audit (in git history) for the v1.3.0 work. Canonical UX artifact the Architect reads. Designs the experience for the three approved features: **F1** context-health reminder, **F2** `/5bot-status` command, **F3** next-command footer. T5 (stage-lock) is research only — hook notes for the Architect are at the end.

---

## Primary User Goals

1. **Know when to compact** — get nudged at a natural seam after a long session, never mid-artifact.
2. **Compact without fear** — understand canon is on disk, so `/compact` or a fresh session loses nothing.
3. **Re-orient in one step** — after compaction, a cold start, or days away, see stage + active ticket + last decision + the one next command.
4. **Never guess the next command** — every `/handoff` and `/gate` ends by naming exactly what to run.

---

## Main User Flows

**Flow A — Long session during a stage (F1).**
Bot works → bot self-judges the session is "long" (heuristic below) → at the next seam (`/handoff`, `/gate`, or the start of a Dev/QA turn) it appends one F1 reminder → user runs `/compact` or starts fresh → user re-runs the current command → state reloads from disk.

**Flow B — Re-orientation after compaction / cold start (F2).**
User returns or has just compacted → runs `/5bot-status` → reads the read-only snapshot → runs the single recommended command → continues the pipeline.

**Flow C — Advancing the pipeline (F3).**
User finishes a stage → `/handoff` prints the bookkeeping **and** an `▶ NEXT: /gate` footer → user runs `/gate` → gate prints a **waiting** footer (options, no command) → user replies with a verdict → gate prints the **resolved** `▶ NEXT` footer → user runs that command.

**Flow A+C interaction (critique fix):** When a long session reaches a stage seam, `/handoff` runs then `/gate` runs right after. Both could self-judge "long." **Rule: F1 fires at most once across that handoff→gate pair** — if `/handoff` already showed the reminder, `/gate` suppresses it. (Mirrors the existing QA→gate suppression.)

---

## Screen List

These are text "screens" (command output). Two are reusable components appended to existing commands.

| # | Screen | New / Modified | Type |
|---|--------|----------------|------|
| S1 | `/5bot-status` snapshot | **New** | Full command output |
| S2 | F1 context-reminder block | **New** | Reusable component (appended) |
| S3 | F3 next-command footer | **New** | Reusable component (appended) |
| S4 | `/handoff` output | Modified (+S3, optional S2) | Command output |
| S5 | `/gate` output | Modified (+S3 waiting/resolved, optional S2) | Command output |
| S6 | Dev turn | Modified (optional S2 one-liner) | Persona behavior |
| S7 | QA turn | Modified (optional S2 one-liner) | Persona behavior |

---

## Navigation Model

The pipeline is **linear and footer-driven**: Product → UX → Architect → Dev ⇄ QA → release, with human gates after Product, Architect, and QA. The user never has to remember the order — the **F3 footer (S3)** always names the next step. **`/5bot-status` (S1)** is the "you are here" re-entry point: the one command that re-orients from disk after any interruption. F1 (S2) is an ambient nudge layered onto existing seams, never its own screen.

All output is **plain Markdown** (horizontal rules, bold, backticks) — no tables, color, or emoji-dependence — so it renders identically in terminal, web, and Cowork.

---

## Screen-by-Screen Detail

### S1 — `/5bot-status` (new, read-only)

**Reads (never writes):** `project-state.md` (canon — stage, active ticket, open questions, last-updated), `decisions.md` (newest block only), `handoff.md` (the "Next Bot + Instructions" line).

**Sections, top to bottom:**
1. **Header + contract subline** — `5bot status — <Project Name> (project-state v<x.y.z>)`, then a muted line: `Read-only snapshot. Nothing was modified. Re-run any command to reload from disk.`
2. **Stage:** verbatim from `## Current Stage`, including the gate-status parenthetical.
3. **Active ticket:** verbatim from `## Current Active Ticket`; show "None yet." plainly rather than hiding the line.
4. **Last decision:** the single newest `decisions.md` block — heading, STATUS, date/by, one-line gist. Never dump the whole log.
5. **Open questions:** only items still OPEN; collapse resolved into a count (`(N resolved — see project-state.md)`); `None open.` if none.
6. **→ Recommended next command:** exactly ONE, arrow-prefixed, with a short why-clause. Derived from `handoff.md`'s next-step line, cross-checked against the canon stage/gate.
7. **Freshness footer:** muted echo of `## Last Updated / By` + `Canon is on disk (…) — safe to /compact.`

**Example output (mid-pipeline):**
```
5bot status — 5bot (project-state v1.2.1)
Read-only snapshot. Nothing was modified. Re-run any command to reload from disk.

Stage:            UX (designing v1.3.0: F1 reminder · F2 /5bot-status · F3 footer) — Product gate APPROVED
Active ticket:    None yet. v1.3.0 proposal approved; no dev ticket cut. Pending: T4 (v1.2.2), T5 (research).
Last decision:    Product Gate — v1.3.0 Scope — APPROVED (2026-06-21 / Human)
                  F1/F2/F3 approved; T5 designed alongside but ships later on its own gate.
Open questions:   None open. (9 resolved questions on file — see project-state.md)

→ Run /ux   — Product gate is APPROVED; UX designs the F1 copy, /5bot-status output, and F3 footer.

State last updated 2026-06-21 / Product gate APPROVED → UX stage.
Canon is on disk (project-state.md · decisions.md · handoff.md · stage files) — safe to /compact.
```

### S2 — F1 context-reminder block (new, reusable)

A single fenced block appended at a seam **only when the bot self-judges the session is long.** Reassurance leads; the action follows.

- **Reassurance line (shared across the system):** *"All canon is on disk — project-state.md, decisions.md, and the stage files. Compacting or restarting loses nothing; re-run the current command to reload."*
- **Voice variants:**
  - **Terse (for `/handoff` + `/gate`, matches command-file voice):** *"Long session — context may be drifting. State is safe on disk (project-state.md, decisions.md, stage files), so nothing is lost. Run `/compact` or start a fresh session, then re-run the current command to reload."*
  - **Minimal one-liner (for Dev/QA persona footers, sits quietly under "Token discipline"):** *"Long session — safe to `/compact` or restart. Canon is on disk; re-run the current command to reload."*

**Self-judgment heuristic (no token API exists).** Treat the session as long when ANY holds: (1) the bot had to re-read a state file it already loaded this session (it scrolled out); (2) ≥2 full pipeline stages, or ≥3 Dev↔QA hops, in one unbroken session; (3) the bot catches itself re-deriving something already settled in `decisions.md`; (4) earlier details now feel hazy and need reconstruction. **Guardrails:** at most once per session (twice only if the user keeps going well past it), only at a seam (never mid-artifact), never on a plainly short session. **When unsure, stay silent.**

### S3 — F3 next-command footer (new, reusable)

Always the **final block** in `/handoff` and `/gate` output; nothing prints after it. If F1 (S2) also fires, **F1 prints above F3** so NEXT stays last. Shared shell:
```
---
**▶ NEXT:** `<command>` — <≤12-word reason>
<optional second line: branch/condition only when a choice exists>
```
Rules: command in backticks with leading slash (copy-pasteable); exactly one primary NEXT; alternates go under an indented "If instead:" line, never as a competing NEXT; the next command is **derived from `## Current Stage`, never guessed** — if the stage is unreadable, NEXT points to `/5bot-status` instead of inventing one.

### S4 — `/handoff` (modified)

Existing bookkeeping output, then (optional S2 if long), then **always S3**. Because `/handoff` only ever runs immediately before a gate, its footer is deterministic: `▶ NEXT: /gate`.

### S5 — `/gate` (modified)

Prints the F3 footer **twice across its lifecycle** because it can't know the verdict until the human replies:
- **(a) Gate OPEN — before reply:** a *waiting* footer naming **no** command, restating "nothing proceeds until you reply," and listing the six verdict options. (Optional S2 may appear above it if long — but **not** if S4 already nudged this transition.)
- **(b) Gate RESOLVED — after verdict:** the real `▶ NEXT`, resolved from (stage just completed) × (verdict). See the verdict map below.

### S6 / S7 — Dev & QA turns (modified)

A one-line S2 (minimal variant) appears under "Token discipline" **only** if the session is self-judged long, **at most once**, at the start of the turn — never mid-artifact. **QA suppresses it if `/gate` will run next** (the gate covers it; avoid double-nudge). Dev→QA loops have no gate, so a long Dev loop is exactly where this earns its place.

---

## Forms & Fields

None. 5bot has no input forms — the only "inputs" are slash commands and the human's one-word gate verdict. The verdict vocabulary is fixed and unchanged: `APPROVED · APPROVED WITH CHANGES · REVISE · REJECT · DEFER · NEEDS CLARIFICATION`.

---

## Buttons & Actions

Actions are commands the user types. v1.3.0 adds/changes:

| Action | Where | Effect |
|--------|-------|--------|
| `/5bot-status` | Anywhere in a 5bot project | Prints S1 snapshot; **writes nothing** |
| `/compact` (Claude built-in) | Suggested by S2 | Compacts context; canon unaffected |
| `▶ NEXT` command | Tail of `/handoff` & `/gate` | The exact next pipeline step |

---

## Empty / Error / Success States

**Success:** `/5bot-status` prints the full snapshot; the F3 footer names one unambiguous next command; F1 fires at a seam and the user compacts confidently.

**Empty / edge states for `/5bot-status`:**
- **No `project-state.md`:** don't error, don't invent. Print `5bot status — no project state found in this directory.` + `This folder isn't set up for 5bot yet.` + `→ Run /5bot-init to scaffold the workflow files.`
- **Initialized but untouched:** render the layout with honest template values — `Stage: (not started)`, `Active ticket: None yet.`, `Last decision: none recorded yet.` → `→ Run /product`.
- **`decisions.md` empty:** `Last decision: none recorded yet.` (don't fail the parse).
- **`handoff.md` missing/stale vs canon:** trust `project-state.md` (source-of-truth order), derive NEXT from the canon stage, and add a muted note: `(handoff.md looks stale — next command derived from canon stage.)`
- **Just-compacted:** no special branch — re-reads disk and prints the normal snapshot; the contract subline + freshness footer are the reassurance.

**Error / fallback for F3 footer:**
- **`## Current Stage` unreadable:** `▶ NEXT: /5bot-status — current stage is unclear; re-orient before choosing a command.`

**F1 non-states (suppression):** short session → no reminder; already nudged this session → silent (max once, twice only if the user runs well past it); mid-artifact → never. **When unsure, stay silent — a missed nudge is cheaper than a nagging one.**

### Gate verdict → NEXT map (S5b)

| Stage at gate | Verdict | `▶ NEXT` |
|---|---|---|
| Product | APPROVED / W-CHANGES | `/ux` |
| UX | APPROVED / W-CHANGES | `/architect` |
| Architect | APPROVED / W-CHANGES | `/dev` (name the first ticket, e.g. T1) |
| QA | APPROVED / W-NOTES, tickets remain | `/dev` (next ticket) |
| QA | APPROVED / W-NOTES, backlog empty | `/handoff` (record release-ready) |
| any | REVISE | re-run the **same** stage's command, then `/handoff` + `/gate` |
| any | NEEDS CLARIFICATION | none — bot answers in-thread, gate re-opens (waiting footer) |
| any | REJECT | `/product` (re-scope) — or stop; logged Rejected |
| any | DEFER | none — paused; logged Deferred; resume via the stage command, re-orient via `/5bot-status` |

---

## Usability Concerns

1. **Reminder fatigue (highest priority).** Mitigated by: self-judgment + max-once-per-session + seam-only + the handoff→gate and QA→gate suppression rules + "silent when unsure." This is the single biggest risk to F1 feeling good.
2. **Post-compaction blind spot (insight).** A freshly-compacted bot's "long" signals reset, so F1 won't immediately re-fire after a compaction — which is correct (don't nag right after the user just acted). `/5bot-status` is the deliberate re-entry instead.
3. **Footer must stay last.** If F1 + F3 both fire, F1 goes above F3 so the actionable NEXT is always the final line a skimmer sees.
4. **Snapshot brevity.** `/5bot-status` must stay ~10 lines: one decision only, resolved questions collapsed to a count. Growth here defeats the "fast re-orient" purpose.
5. **Consistency as a system.** F1/F2/F3 deliberately reuse one idea — "canon is on disk, safe to compact" — and one shell (`---` + bold lead). They should read as one feature, not three.

---

## Assumptions

- `/compact` is the relevant Claude Code command; phrasing also says "or start a fresh session" so it stays meaningful if `/compact` behaves differently in Cowork (to be verified at QA).
- Bots can self-assess drift well enough for the heuristic; if they over- or under-fire in practice, only the heuristic text needs tuning (no structural change).
- Plain-Markdown rendering is safe across terminal / web / Cowork.

## Open Questions — RESOLVED at UX gate (2026-06-21)

- **OQ-1:** ✓ **RESOLVED — bot self-judgment.** The "long" heuristic (4 drift signals + guardrails) is the shipped mechanism; it lives in the Dev/QA personas and the F1 block spec.
- **OQ-2:** ✓ **RESOLVED — keep `/5bot-status` brief.** No Known Risks section; it stays at stage / active ticket / last decision / open questions / one next command / freshness footer.

## T5 hook notes (research only — for the Architect, NOT designed here)

F2/F3 are **read-only** features over the **same state** a future hard lock would **enforce** — making them a low-risk precursor. Key hooks the Architect should scope under T5:
- **`## Current Stage`** is the cursor but is **free-text prose** today. A hard lock needs a machine-parseable token (e.g., enum `PRODUCT|UX|ARCHITECT|DEV|QA|RELEASE` + a `gate: APPROVED|PENDING` flag) alongside the human-readable text. **F2/F3 should read whatever structured form is chosen, so the format is settled once.**
- **`decisions.md`** is the authoritative approval ledger; a lock must verify the gate approval *exists there*, not trust the cursor alone (a stale/hand-edited stage could wrongly unlock).
- **Enforcement point:** a pre-flight check at the top of each command (read stage + decisions, refuse/warn if out of order). **Dev→QA is intentionally ungated** — the lock table is not uniform.
- **Honesty:** with no runtime hook, a "hard" lock is still bot-honored instruction — it raises the floor, it is not cryptographic. T5 should say so.
- **Migration:** legacy projects lack the token — fall back to "unlocked + warn," never hard-block. Adding the token to the template is ≥ MINOR and must sync the `_framework\5bot\` copy and the standalone plugin copy.

---

# UX (v2.1.0 — Claude Design Integration, optional)

> Canonical UX artifact the Architect reads for v2.1.0. Designs the experience of the **optional Claude Design "design-reference" step** approved at the 2026-06-22 Product gate. **Native Claude Design only** (Figma stays out). In every downstream project, that project's `ux.md` stays source of truth; a design is a **linked aid, never an auto-build input.**

## Primary User Goals (v2.1.0)

1. **Attach a Claude Design artifact** to a 5bot project so the Dev Bot builds against the real design.
2. **Never hit a dead end** — when the `claude_design` connector isn't available in this surface, a fallback still records the design.
3. **Always know which path is active** (connector import vs fallback) and what to do next.
4. **Keep `ux.md` canon** — the design is referenced/linked, not treated as the spec.

## Design stance on the open questions — RESOLVED at UX gate (2026-06-22)

- **OQ-2 → RESOLVED: the `.zip` download is the first-class MVP path.** Human direction: *"I can download the design from Claude Design as a `.zip` — go with that."* MVP = the user exports the design as a `.zip` from Claude Design, drops it in the repo, and the bot records it. The `claude_design` connector import (D2) is a **progressive enhancement, deferred past MVP** — so the **MVP carries no dependency on the connector**, which sidesteps the surface-availability problem entirely.
- **OQ-3 → RESOLVED: the committed `.zip` (with its extracted `*.dc.html`) is the stored artifact; `ux.md` holds the Design Reference block that links it.** Saved in the repo (proposed `design/`); exact path/format = Architect (OQ-7).
- **OQ-5 → ACCEPTED: UX-only capture for MVP** (no objection raised). `/ux` captures; the Dev Bot consumes. Ad-hoc Dev-stage attach is post-MVP.

## Main User Flow — Flow D: attach a Claude Design reference at `/ux`

1. During `/ux`, the user signals they have a Claude Design design — by pasting the **project URL**, pasting Claude Design's **"Send to local coding agent" prompt**, or naming an **exported file**.
2. The UX Bot checks whether the `claude_design` connector is connected/authorized in this surface.
   - **Connected → D2 (connector import):** use the claude_design tools to fetch the project/file(s); record the reference + imported file list; optionally export a local copy into the repo.
   - **Not connected → D3 (fallback):** state plainly that the connector isn't available *here* (it's surface-dependent), then take the fallback — record the URL as a reference and/or accept an exported file path (the user uses Claude Design's **"Download zip instead"** or saves the `*.dc.html` and drops it in the repo).
3. Either path → **D4**: write the **Design Reference** block into `ux.md`. Continue normal UX (flows/screens can now cite the design). The design never auto-generates code — Dev implements via tickets.

**Branch default:** no design → plain text-only `/ux`, exactly as today. The step is opt-in and adds zero friction when unused.

## Screen List (text "screens" = command behavior)

| # | Screen | New / Mod | Type |
|---|--------|-----------|------|
| D1 | `/ux` design-reference step (offer + capture) | Modified `/ux` | Command behavior |
| D2 | Connector-available import sub-flow | New | Sub-flow |
| D3 | Connector-unavailable fallback sub-flow | New | Sub-flow |
| D4 | "Design Reference" block written to `ux.md` | New | Artifact section |
| D5 | Connector-status / error / guidance states | New | States |

## Navigation Model

The design-reference step is an **optional branch inside `/ux`**, never a gate or blocker. Plain Markdown only (terminal / web / Cowork parity). The connector is an *enhancement*; the fallback is the floor.

## Screen-by-Screen Detail

### D1 — `/ux` design-reference step
At the top of `/ux`, if the user supplied (or pastes) a Claude Design URL / prompt / file, enter Flow D; otherwise proceed with normal UX silently. The bot parses a pasted "Send to local coding agent" prompt to seed `{project URL, file name}` automatically.

### D2 — Connector available (import)
> *Claude Design connector detected. Importing **Tag Hag.dc.html** from the project… imported. Recorded in `ux.md` → Design Reference. (Saved a local copy to `design/tag-hag.dc.html`.)*

### D3 — Connector NOT available (fallback) — the key state
```
The claude_design connector isn't connected in this environment, so I can't pull the
design from the link directly. (Connector availability depends on your Claude surface —
local Claude Code vs Cowork vs web — not on 5bot.) Two ways forward, both fine:

  • Quick: I'll record the design URL as a reference now (note: a URL can change or
    expire and isn't version-controlled).
  • Durable (recommended): in Claude Design choose "Download zip instead" (or save the
    *.dc.html), drop the file into this repo, and tell me the path — I'll link it.

Either way, ux.md stays the source of truth. Want me to (a) record the URL,
(b) wait for an exported file, or (c) skip the design for now?
```
*The copy deliberately does NOT tell the user to run a specific connect command, because whether one exists is surface-dependent and unverified (see OQ-6). When Architect confirms the real connect/auth step per surface, D3 gains a third "connect it" option.*

### D4 — "Design Reference" block (written to `ux.md`)
```
## Design Reference (Claude Design)
- Source:         Claude Design — <project URL>
- File(s):        Tag Hag.dc.html
- Import method:  fallback-zip            # MVP; "connector" reserved for deferred enhancement (T15)
- Local artifact: design/tag-hag.dc.html (committed) | none (URL only)
- Captured:       2026-06-22
- Covers:         <which screens / flows this design informs>
- Note:           ux.md is canon; this design is a linked aid, not the spec.
```
*Exact file location / format is Architect's call (OQ-7); the block schema above is the UX contract.*

### D5 — States
- **Connected + authorized:** import succeeds → D4. Success.
- **Connected + unauthorized:** offer to authorize (the real step, per OQ-6) OR use fallback. Never block.
- **Not connected (reported case):** D3 fallback. Never block.
- **URL-only (no export):** record, but warn it's not version-controlled; recommend a committed export.
- **Bad / expired / inaccessible link:** say so plainly; offer a URL-only note or ask for an export.
- **No design:** skip; normal `/ux`.

## Forms & Fields

No GUI forms. Inputs are: a pasted Claude Design URL or "send to agent" prompt, an optional exported file path, and a one-word choice (record URL / wait for file / skip).

## Buttons & Actions

| Action | Where | Effect |
|--------|-------|--------|
| Paste Claude Design URL / "Send to local coding agent" prompt | during `/ux` | Seeds + starts the design-reference capture |
| Provide exported file path (`*.dc.html` / zip) | `/ux` fallback | Records + links a committed local artifact |
| Connector import | when connector present | Pulls the design via `claude_design` tools |

## Empty / Error / Success States

**Success:** a Design Reference block lands in `ux.md`, Dev can implement against it, and the user is never blocked by a missing connector. See **D5** for the remaining states.

## Usability Concerns

1. **Don't over-promise the connector (highest).** The user's actual pain was being told to use a connector/command that wasn't there. Copy stays **conditional** and always pairs with the working fallback — no instruction to run an unverified command (gated on OQ-6).
2. **Source-of-truth clarity.** The block states `ux.md` is canon and the design is an aid — prevents "the mock-up is the spec" drift.
3. **Durability.** URL-only references can rot; nudge toward a committed export.
4. **Stay invisible when unused.** Projects that don't use Claude Design see no change to `/ux`.
5. **No auto-build.** Capture only; Dev implements via tickets — matches the human's "I want the option, not this specific design built" intent.

## Visual aid for this gate (optional)

Per the `/ux` protocol, a Claude Design mock-up of this screen list could serve as a human-review aid — but the connector isn't available in this environment (the very gap this feature addresses), so this gate uses `ux.md` as the canonical and only artifact. If a mock-up is later produced, export it into the repo and link it here.

## Assumptions

- When present, the `claude_design` connector exposes import/fetch tools callable by the agent (feasibility → Architect, OQ-6).
- "Send to local coding agent" prompts embed a parseable `{project URL, file name}`.
- A plain-Markdown reference block needs no schema enforcement at MVP.

## Open Questions

- ✓ **OQ-2 / OQ-3 / OQ-5 — RESOLVED at UX gate (2026-06-22):** `.zip` download is the first-class MVP path; the committed `.zip` (+ extracted `*.dc.html`) is the artifact, linked from a `ux.md` Design Reference block; UX-only capture. Connector import deferred to a progressive enhancement.
- **OQ-6 (→ Architect, now NON-blocking for MVP):** real connect/authorize path for `claude_design` per surface (is `/design-login` a real command in Code/Cowork/web?) — only needed when the connector-import enhancement is built; the MVP zip path doesn't depend on it.
- **OQ-7 (→ Architect, minor):** default repo location + format for committed design artifacts (proposed `design/`).

---

# UX (v2.2.0 — Usage-Aware Status Reporting)

> ⛔ **SUPERSEDED (2026-06-29).** This API-usage design was built then reverted at the QA gate (model can't read usage %). v2.2.0 was re-scoped to **git-based sync-awareness**; see "UX (v2.2.0 re-scoped — Sync-Awareness via read-only git status)" below. Retained for history.

> Canonical UX artifact the Architect reads for v2.2.0. Designs the experience of **read-only API usage status** integrated into the `/5bot-status` command. Approved at Product gate (2026-06-26) as a soft personal preference — informational, non-blocking usage visibility.

## Primary User Goal (v2.2.0)

**Know current API hourly usage** without leaving the workflow, so you can decide whether to continue or pause before the next bot command.

## Main User Flow (v2.2.0) — Flow E

User at any point in the pipeline:
1. Runs `/5bot-status` (existing re-orientation command).
2. Sees a **read-only API usage line** showing current hourly quota consumption (e.g., "API hourly usage: 76% (resets at 15:00 UTC)").
3. Uses that info to decide: continue normally, or pause and resume later.

**No workflow change.** The line is informational; it never blocks execution. User decides the action.

## Screen List (v2.2.0)

| # | Screen | New / Mod | Type |
|---|--------|-----------|------|
| E1 | `/5bot-status` usage line | Modified S1 | Command output section |

## Navigation Model (v2.2.0)

The usage line is a **read-only, optional data field** appended to `/5bot-status`. It adds one line between "Active ticket" and "Last decision" (or alternative position per Architect). No new command, no gate, no fork. Silent omission if data unavailable (no error, no placeholder).

## Screen-by-Screen Detail (v2.2.0)

### E1 — `/5bot-status` usage line (modification to S1)

**Location in output:** After "Active ticket" and before "Last decision" (or after all core sections, just before the final "→ Run" footer — Architect decides based on readability).

**Format:**
```
Usage:  API hourly usage: 76% (resets at 15:00 UTC)
```

**Data displayed:**
- **Percentage:** Current usage as a simple integer percentage (0–100%) of the hourly rolling window.
- **Reset time:** UTC time (HH:MM format) when the hourly quota resets, so users know when it's safe to resume. Example: "15:00 UTC" (not relative like "in 23 minutes" — UTC is unambiguous).
- **Tone:** Neutral, factual. No warning emoji (⚠️), no color, no urgency. Just the numbers.

**Fallback / unavailable:**
- If the host environment doesn't expose API usage data (no MCP context, API unavailable, surface doesn't support it): **silently omit the line.** Print nothing, no placeholder, no error. The command succeeds either way.

**Example output (with usage):**
```
5bot status — 5bot (project-state v2.2.0)
Read-only snapshot. Nothing was modified. Re-run any command to reload from disk.

Stage:            Architect (approving v2.2.0: usage-aware status line in /5bot-status)
Active ticket:    None yet.
Usage:            API hourly usage: 76% (resets at 15:00 UTC)
Last decision:    Product Gate — v2.2.0 Usage-Aware Status — APPROVED (2026-06-26 / Human)
Open questions:   3 blocking (see product.md) — resolved by Architect.

→ Run /architect  — Confirm API feasibility and decide reset-time format.

State last updated 2026-06-26 / Product gate APPROVED → UX stage.
Canon is on disk (project-state.md · decisions.md · handoff.md · stage files) — safe to /compact.
```

**Example output (usage unavailable):**
```
5bot status — 5bot (project-state v2.2.0)
Read-only snapshot. Nothing was modified. Re-run any command to reload from disk.

Stage:            Architect (approving v2.2.0: usage-aware status line in /5bot-status)
Active ticket:    None yet.
Last decision:    Product Gate — v2.2.0 Usage-Aware Status — APPROVED (2026-06-26 / Human)
Open questions:   3 blocking (see product.md) — resolved by Architect.

→ Run /architect  — Confirm API feasibility and decide reset-time format.

State last updated 2026-06-26 / Product gate APPROVED → UX stage.
Canon is on disk (project-state.md · decisions.md · handoff.md · stage files) — safe to /compact.
```
(No "Usage" line if data unavailable.)

## Usability Concerns (v2.2.0)

1. **Silent fallback is critical.** If the API query fails or the host doesn't expose usage, omit the line completely. Never show "Usage: unavailable" or error. The command must always succeed.
2. **Placement matters.** The line should sit near the "active ticket" / "decision" context (mid-output), not at the end, so users see it before deciding what to run next.
3. **No alarm.** Keep tone neutral; numbers only. This is informational for planning, not a warning.
4. **Reset time clarity.** UTC is unambiguous; a relative format ("in 15 minutes") is tempting but fragile if the user copies/pastes the output later.

## Assumptions (v2.2.0)

- **ASSUMPTION:** API usage data is available via Claude Code MCP context, host environment variable, or Claude.ai dashboard API (never via explicit user-managed credentials in project files).
- **ASSUMPTION:** The "hourly" window is Claude API's rolling 1-hour limit. Reset time = current hour + 60 minutes.
- **ASSUMPTION:** Users can interpret "76% of hourly quota" without further explanation.
- **ASSUMPTION:** Silent omission when unavailable is acceptable (no user-facing fallback needed).

## Open Questions (v2.2.0)

**OQ-1 (Data source / feasibility):** Can Claude Code expose API usage via MCP, environment variable, or host context? Is there a quota / usage endpoint available, or is the data dashboard-only? → **Architect to confirm.**

**OQ-2 (Reset time format):** Show "15:00 UTC" (unambiguous) or "~15 minutes from now" (human-friendly but fragile)? → **Architect to decide.** Recommend UTC for durability.

**OQ-3 (Exact placement):** After "Active ticket" (context-first) or at the end of core sections (least intrusive)? → **Architect to decide** based on the final S1 layout. Recommend after Active ticket.

---

# UX (v2.2.0 re-scoped — Sync-Awareness via read-only git status)

> Supersedes the API-usage UX above (reverted at the QA gate). Designs the read-only **git-awareness line** in `/5bot-status`, approved at the re-scope Product gate (2026-06-29). Read-only, no network, no mutation, neutral tone, plain Markdown.

## Primary User Goal (v2.2.0 re-scoped)

See at a glance, in `/5bot-status`, whether the local copy is in sync with its remote and whether there are uncommitted changes — so you catch a stale-copy / unsaved-work situation **before** acting on it.

## Main User Flow — Flow E (re-scoped)

1. User runs `/5bot-status`.
2. If the project is a git repo, the snapshot's freshness footer includes one read-only **git line** (sync + cleanliness).
3. User reads it and decides what to do — purely informational; 5bot suggests no action and never touches the repo.
4. If not a git repo (or git unavailable), the line is silently omitted; the rest of the snapshot is unchanged.

## Screen List (v2.2.0 re-scoped)

| # | Screen | New / Mod | Type |
|---|--------|-----------|------|
| E1 | `/5bot-status` git line | Modified S1 (freshness footer) | Command output line |

## Navigation Model

A single read-only line appended to the **freshness footer** of `/5bot-status`. No new command, no gate, no fork. The line is **informational only** — it does **not** change the recommended next command (which stays above the footer). Silent omission when not a git repo.

## Placement decision (OQ-2, resolved here)

**In the freshness footer, as its last line** — after "State last updated…" and "Canon is on disk… safe to /compact." Rationale: git/sync status answers *"how current/trustworthy is my local copy?"* — the same question the freshness footer already addresses (last-updated + canon-on-disk). Grouping them keeps all "state freshness/trust" signals together and keeps the actionable **→ next command** prominent (footer stays last). (The old API-usage line sat mid-snapshot to inform the next-command decision; for git-awareness the "trust/freshness" framing fits the footer better.)

## Screen-by-Screen Detail

### E1 — `/5bot-status` git line

Appended as the final line of the freshness footer. Format:
```
git: <branch> · <sync state> · <cleanliness>
```
- **branch:** current branch name; `detached HEAD` if detached.
- **sync state** (vs upstream, from already-fetched refs — labeled, because the command does NOT fetch): `N behind origin (last fetch)` · `M ahead` · `N behind, M ahead (last fetch)` · `even with origin (last fetch)` · `no upstream` (omit ahead/behind when none).
- **cleanliness:** `clean` or `uncommitted changes` (from `git status`).

**Example (footer with git line):**
```
State last updated 2026-06-29 / Product gate APPROVED → UX.
Canon is on disk (project-state.md · decisions.md · handoff.md · stage files) — safe to /compact.
git: main · 2 behind origin (last fetch) · uncommitted changes
```

## Empty / Error / Success States

- **Git repo, has upstream:** full line (branch · sync · cleanliness). **Success.**
- **Git repo, no upstream:** `git: <branch> · no upstream · <cleanliness>` (omit ahead/behind).
- **Detached HEAD:** `git: detached HEAD · <cleanliness>` (omit ahead/behind).
- **Even + clean:** `git: <branch> · in sync · clean` (terse all-good render; still shown — consistent sync indicator).
- **Not a git repo / git unavailable / any git error:** **omit the line entirely** — no error, no placeholder, command still succeeds (silent no-op).

## Forms & Fields

None. The git line is pure read-only output; no inputs.

## Buttons & Actions

None new. The line is informational; if the user wants to act (commit / pull), they do so in their own terminal — 5bot neither prompts nor performs it.

## Usability Concerns

1. **Informational, never advisory (highest).** The line states facts; it does NOT tell the user to pull/commit and does NOT change the recommended next command. (A soft nudge is out of scope per Product.)
2. **Honest staleness.** "(last fetch)" signals that ahead/behind reflects the last fetch, not live — the command never fetches. Prevents false confidence.
3. **Portability / no noise.** Silent omission when not a git repo (5bot doesn't require git); no errors, no "n/a".
4. **Brevity.** Exactly one line; keeps `/5bot-status` to ~11 lines. Don't expand into multi-line git detail.
5. **Neutral tone, plain Markdown, no emoji** — consistent with the rest of the snapshot; renders identically in terminal / web / Cowork.
6. **Privacy.** Only local branch / sync / dirty facts; no remote URLs, emails, or identifying info.

## Assumptions (v2.2.0 re-scoped)

- Reading local git state is available via the host shell the agent already uses (exact commands → Architect).
- `/5bot-status` is often run inside a git checkout; when not, it silently no-ops.

## Open Questions (v2.2.0 re-scoped)

- ✓ **OQ-2 (placement / format): RESOLVED here** — freshness-footer line, format above.
- **OQ-3 (→ Architect):** exact no-network commands (`git rev-list --left-right --count @{u}...HEAD`, `git status --porcelain`, branch via `git symbolic-ref`/`rev-parse`), detached / no-upstream handling, cross-platform invocation, and graceful omission on any git error.
- **OQ-4 (minor, deferred):** a one-time `/5bot-init` nudge to set up git — Product deferred; not designed here.

## Visual aid for this gate (optional)

Per the `/ux` protocol, a Claude Design mock-up could illustrate the footer line — but it's a single text line; `ux.md` is the canonical and sufficient artifact here. No mock-up needed.

---

# UX (v2.3.0 — Lean Context)

> Designs the *visible* surface of Lean Context (Product gate APPROVED 2026-06-29): automatic archive rollover + lazy loading. Lazy loading is **invisible by design** (bots just read the live slice; mechanism → Architect, OQ-3). The user only ever sees: the rollover note, the archive file, pointer lines, `/5bot-archive`, and a `/5bot-status` note. Kept lean (dogfooding the goal).

## Primary User Goal
Context stays lean automatically; history is preserved and retrievable; the user manages nothing.

## Main Flows
- **Flow F — Auto rollover (core, near-invisible):** at `/handoff`, the bot relocates stale content (superseded version sections, decisions beyond the last ~8, shipped-ticket records) into `archive.md`, then prints **one line** naming what moved. User does nothing.
- **Flow G — Retrieval (rare):** when old info is needed, the relevant `archive.md` section is read on demand; the pointer lines in working files say where to look.
- **Flow H — One-time migration:** `/5bot-archive` applies the rollover rules retroactively to an already-bloated project; idempotent.

## Screens (text surfaces)
| # | Surface | New / Mod |
|---|---------|-----------|
| L1 | `archive.md` (sectioned, append-only) | New file |
| L2 | per-file "→ archive.md" pointer line | Mod (working files) |
| L3 | `/handoff` rollover note (1 line) | Mod `/handoff` |
| L4 | `/5bot-archive` output | New command |
| L5 | `/5bot-status` archive note (1 line) | Mod `/5bot-status` |

## Detail
**L1 — `archive.md`:** append-only, three sections — `## Stage history` (superseded product/ux/architecture version sections, labeled by version), `## Decisions (archived)` (older decision blocks), `## Dev-QA (archived)` (shipped-ticket records). Header: *"Archive — relocated, not loaded by default, never summarized. Open a section only when you need history."* Never read on normal turns.

**L2 — pointer lines** (one line at the foot of each working file): `decisions.md` → *"Older decisions → `archive.md` § Decisions."*; `product`/`ux`/`architecture.md` → *"Superseded version sections → `archive.md` § Stage history."*; `dev-qa.md` → *"Shipped-ticket records → `archive.md` § Dev-QA."* Makes retrieval discoverable without loading the archive.

**L3 — `/handoff` rollover note:** when something rolls, append exactly one line: *"🗂 Archived → archive.md: &lt;e.g. 'v2.1.0 stage sections, 4 decisions, T10–T14'&gt;. Working files trimmed; nothing lost (relocated, in git)."* If nothing rolled, print nothing. **This announcement is the trust anchor** — rollover is automatic but never silent.

**L4 — `/5bot-archive`:** one-time/occasional; no args (MVP). Applies the rollover rules across all working files, then reports counts (*"Archived 3 version sections, 11 decisions, 9 tickets → archive.md."*). **Idempotent:** on an already-lean project, prints *"Nothing to archive — already lean."* Non-destructive (relocations are git-recoverable).

**L5 — `/5bot-status` archive note:** one muted line when an archive exists — *"Archive: archive.md (N entries) — history off the read path."* Omit when none. Stays 1 line (don't re-bloat the snapshot).

## Forms & Fields
None. `/5bot-archive` takes no args in MVP.

## Empty / Error / Success States
- **No archive yet** (new/lean project): no `archive.md`, no pointer lines, no rollover note, no status note. `archive.md` is created lazily on first rollover.
- **Archive exists:** pointer lines + status note present.
- **`/5bot-archive` on a lean project:** *"Nothing to archive — already lean."*
- **Retrieval:** open the named `archive.md` section.
- **Lazy load:** bots read only the live slice; if archived detail is needed they open `archive.md` explicitly (mechanism → Architect).

## Usability Concerns
1. **Automatic but never silent (highest).** Rollover happens without the user asking, but L3's one-line note + "nothing lost" is what makes it trustworthy. Silent relocation would feel like data loss.
2. **Non-destructive is visible.** The `archive.md` header + pointer lines make clear history was *moved*, not deleted.
3. **Don't re-bloat.** Pointer lines (L2) and the status note (L5) are one line each — the cure mustn't become the disease.
4. **Conservative trim.** Per Product OQ-4; the archive is always one read away, so erring toward keeping is cheap.
5. **Idempotent migration.** `/5bot-archive` is safe to re-run; clear "nothing to do" state.

## Open Questions
- ✓ **OQ-2** (single `archive.md`): adopted (Product rec).
- **OQ-3 (→ Architect):** lazy-read mechanism (file-split vs anchored/offset reads) — UX-neutral (invisible either way).
- **OQ-6 (→ Architect/UX):** should `/handoff` ever ask before archiving, or always auto + announce? **Rec: always auto + the L3 note** (matches the low-interaction goal; non-destructive) — no confirmation prompt.
- **OQ-7 (minor → Architect):** a `/5bot-archive --dry-run`? **Rec: skip for MVP** (it's git-reversible + reports after); add later if wanted.

## Visual aid for this gate (optional)
Text surfaces only; `ux.md` is the canonical, sufficient artifact. No mock-up needed.

---

## Assumptions

- `/compact` is the relevant Claude Code command; phrasing also says "or start a fresh session" so it stays meaningful if `/compact` behaves differently in Cowork (to be verified at QA).
- Bots can self-assess drift well enough for the heuristic; if they over- or under-fire in practice, only the heuristic text needs tuning (no structural change).
- Plain-Markdown rendering is safe across terminal / web / Cowork.

## Open Questions — RESOLVED at UX gate (2026-06-21)

- **OQ-1:** ✓ **RESOLVED — bot self-judgment.** The "long" heuristic (4 drift signals + guardrails) is the shipped mechanism; it lives in the Dev/QA personas and the F1 block spec.
- **OQ-2:** ✓ **RESOLVED — keep `/5bot-status` brief.** No Known Risks section; it stays at stage / active ticket / last decision / open questions / one next command / freshness footer.
