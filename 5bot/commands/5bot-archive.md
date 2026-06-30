---
description: One-time retroactive sweep that relocates stale history to archive.md (Lean Context).
---
Apply the Lean Context archive rollover retroactively across this project's working files (see the `five-bot` skill → "Lean Context"). Use this once on an existing / already-bloated project; `/handoff` handles rollover incrementally thereafter. **Relocate only — never delete or summarize. History is recoverable via git.**

Steps:
1. Read `project-state.md` to determine the **current version**.
2. Apply the rollover rules across the working files:
   - `product.md` / `ux.md` / `architecture.md`: move version sections **older than the current version** → `archive.md` § Stage history (keep labels).
   - `decisions.md`: move all but the **newest 8** blocks → § Decisions.
   - `dev-qa.md`: move the full record (card + Developer Notes + QA Review) of every ticket that is **DONE and whose version has shipped** → § Dev-QA. Keep active/open tickets.
3. Create `archive.md` if absent (use the schema in the skill); append moved content under the right section, preserving headings/labels.
4. Ensure each trimmed working file carries its one-line `→ archive.md` pointer.
5. Report what moved — counts per file (e.g., *"Archived 3 version sections, 11 decisions, 9 tickets → archive.md."*). If nothing qualifies, print **"Nothing to archive — already lean."** and write nothing.

**Idempotent:** safe to re-run; only ever moves content that newly qualifies. Writes nothing to `project-state.md` and reverses no decisions — it only relocates. No arguments.
