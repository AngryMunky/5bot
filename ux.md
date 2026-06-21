# UX (v1.3.0 — Pipeline Awareness & Guardrails)

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
