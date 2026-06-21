---
description: Read-only orientation snapshot — current stage, active ticket, last decision, and the next command.
---
Print a read-only 5bot status snapshot to re-orient after a compaction, a fresh session, or time away (see the `five-bot` skill). **Read only — do not modify any file.**

Read `project-state.md` (canon), the newest block of `decisions.md`, and `handoff.md` (its "Next Bot + Instructions" line). Then print, briefly:

- **Header:** `5bot status — <Project Name> (project-state v<x.y.z>)`, then a muted line: `Read-only snapshot. Nothing was modified. Re-run any command to reload from disk.`
- **Stage:** verbatim from `## Current Stage` (include the gate-status parenthetical).
- **Active ticket:** verbatim from `## Current Active Ticket` (show "None yet." plainly).
- **Last decision:** the single newest `decisions.md` block — heading, STATUS, date/by, one-line gist. One entry only.
- **Open questions:** only items still OPEN; collapse resolved into a count (`(N resolved — see project-state.md)`); `None open.` if none.
- **→ Recommended next command:** exactly ONE, arrow-prefixed, with a short reason. Derive from `handoff.md`'s next-step line cross-checked against the canon stage/gate, using the skill's verdict / next-command map. If a gate is pending, recommend `/gate`; if a gate was approved with a next role, recommend that role's command.
- **Freshness footer:** `State last updated <date> / <by>.` then `Canon is on disk (project-state.md · decisions.md · handoff.md · stage files) — safe to /compact.`

Keep it to ~10 lines. Do NOT print Known Risks. Do NOT recommend more than one command.

**Edge cases:**
- **No `project-state.md`:** print `5bot status — no project state found in this directory.`, `This folder isn't set up for 5bot yet.`, and `→ Run /5bot-init to scaffold the workflow files.` Create nothing.
- **Initialized but untouched** (template defaults): render the layout with honest values — `Stage: (not started)`, `Active ticket: None yet.`, `Last decision: none recorded yet.` → `→ Run /product`.
- **`decisions.md` empty:** `Last decision: none recorded yet.`
- **`handoff.md` missing/stale vs canon:** trust `project-state.md`, derive the next command from the canon stage, and add a muted note `(handoff.md looks stale — next command derived from canon stage.)`
- **Just-compacted:** no special handling — re-read disk and print the normal snapshot.
