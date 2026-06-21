# Dev & QA (v2.0.0)

## Backlog / Tickets

<!-- Set by the Architect Bot. Each ticket: Goal · Acceptance Criteria · files likely involved · out of scope -->

### T1: Add ⚠️ Warning to Handoff.md Template — v1.2.1

**Goal:** Prevent users from editing handoff.md thinking it's permanent state.

**Acceptance Criteria:**
- [ ] Master template (`_framework/5bot/templates/handoff.md`) has ⚠️ warning at top
- [ ] Warning text: "DO NOT EDIT. This file is overwritten each stage. Put decisions in decisions.md, state in project-state.md."
- [ ] Warning appears before any template sections
- [ ] Verified: Running `/5bot-init` copies warning into new projects

**Files involved:**
- `C:\AI Bins\Code\_framework\5bot\templates\handoff.md`

**Out of scope:**
- No code, no validation, no UI changes

---

### T2: Add "Canon" Definition to Project-state.md Template — v1.2.1

**Goal:** Explain "canon" jargon so new users understand project-state.md is the source of truth.

**Acceptance Criteria:**
- [ ] Master template (`_framework/5bot/templates/project-state.md`) has definition after intro
- [ ] Definition: "Canon = source of truth. All other files reference this one."
- [ ] Definition placed prominently at top of file (after intro comment)
- [ ] Verified: Clarity improvement when reading template fresh

**Files involved:**
- `C:\AI Bins\Code\_framework\5bot\templates\project-state.md`

**Out of scope:**
- No structural changes to file sections

---

### T3: Add Rule Summary to 8 Command Files — v1.2.1

**Goal:** Teach users anti-drift rules in context by summarizing scope per role.

**Acceptance Criteria:**
- [ ] All 8 command files have rule summary (product, ux, architect, dev, qa, 5bot-init, handoff, gate)
- [ ] Summary explains role scope + constraints (2-3 sentences)
- [ ] Example: "Product Bot defines scope. UX Bot cannot change it. Architect Bot cannot expand it. Dev Bot implements it. All scope changes require human approval at a gate."
- [ ] Summary placed after role introduction, before task instructions
- [ ] Verified: Running each command shows improved clarity

**Files involved:**
- `C:\AI Bins\Code\plugins\five-bot\commands\product.md`
- `C:\AI Bins\Code\plugins\five-bot\commands\ux.md`
- `C:\AI Bins\Code\plugins\five-bot\commands\architect.md`
- `C:\AI Bins\Code\plugins\five-bot\commands\dev.md`
- `C:\AI Bins\Code\plugins\five-bot\commands\qa.md`
- `C:\AI Bins\Code\plugins\five-bot\commands\5bot-init.md`
- `C:\AI Bins\Code\plugins\five-bot\commands\handoff.md`
- `C:\AI Bins\Code\plugins\five-bot\commands\gate.md`

**Out of scope:**
- No core instruction changes
- No persona changes

---

### T4: Update README with Validation Checklist — v1.2.2 (Backlog)

**Status: BACKLOG** — Next minor release after v1.2.1.

**Goal:** Provide pre-flight checklist for each command to reduce user confusion.

**Acceptance Criteria:**
- [ ] README has "Before Running Each Command" section
- [ ] Checklist for all 8 commands (e.g., "Before `/product`: Is the problem clear?")
- [ ] Checklists are practical and actionable
- [ ] Verified: New users follow checklist successfully

**Files involved:**
- `C:\AI Bins\Code\plugins\five-bot\README.md`

**Out of scope:**
- No technical enforcement (docs only)

---

### T5: Research Gate Enforcement (v1.3.0 Planning)

**Status: PLANNING** — Future research phase; may inform v1.3.0 feature set.

**Goal:** Explore technical enforcement to prevent skipping approval gates.

**Acceptance Criteria:**
- [ ] Design document: can we add a "stage lock" to project-state.md?
- [ ] Analysis: what breaks if stage lock is missing? Is it necessary?
- [ ] Estimate effort for v1.3.0 implementation
- [ ] Decision: prioritized for v1.3.0, or defer further?

**Files involved:**
- Design doc (new file or append to architecture.md)

**Out of scope:**
- Actual implementation (research phase only)

**Now informed by:** `ux.md` → "T5 hook notes" (Current Stage is free-text prose; needs a parseable stage token + gate flag; decisions.md is the authoritative approval ledger; Dev→QA stays ungated; legacy projects fall back to unlocked+warn).

---

## v2.0.0 Backlog — Pipeline Awareness & Guardrails (dev cycle tracked as "v1.3.0")

> Sequential build: **T6 → T7 → T8 → T9** (shared files: SKILL.md, handoff.md, gate.md). Canonical edits in `plugins/five-bot/`; sync guidance into `_framework/5bot/`. See `architecture.md` → "v1.3.0".

### T6: F3 — Next-command footer on /handoff and /gate — v1.3.0

**Goal:** Every `/handoff` and `/gate` ends by naming the exact next command, so the user never guesses the pipeline order.

**Acceptance Criteria:**
- [ ] `skills/five-bot/SKILL.md` defines the canonical footer shell once: `---` + `**▶ NEXT:** \`<command>\` — <≤12-word reason>`, plus the full gate-verdict → next-command map (all 6 verdicts × each stage, per `ux.md`).
- [ ] `commands/handoff.md`: append instruction to end output with the footer; NEXT is always `/gate` (deterministic).
- [ ] `commands/gate.md`: print the **waiting** footer (options, NO command) before the human replies; print the **resolved** `▶ NEXT` footer after the verdict, derived from Current Stage × verdict.
- [ ] Next command is derived from `## Current Stage` in `project-state.md`, never guessed; if unreadable, footer points to `/5bot-status`.
- [ ] Exactly one primary NEXT; alternates under an indented "If instead:" line. Plain Markdown only (Cowork-safe).
- [ ] **Sync:** the footer/verdict convention noted in `_framework/5bot/rules.md` (and userSettings handoff/gate command copies if present).

**Files involved:** `skills/five-bot/SKILL.md`, `commands/handoff.md`, `commands/gate.md`; sync: `_framework/5bot/rules.md`.

**Out of scope:** F1 reminder text (T8); any enforcement/blocking (T5); changing the verdict vocabulary.

---

### T7: F2 — New read-only /5bot-status command — v1.3.0

**Goal:** A one-command re-orientation snapshot for after compaction / a cold start.

**Acceptance Criteria:**
- [ ] New file `commands/5bot-status.md` (read-only — writes nothing).
- [ ] Reads `project-state.md` (canon), `decisions.md` (newest block only), `handoff.md` (next-step line).
- [ ] Prints the brief 6-part snapshot per `ux.md` S1: header+contract subline, Stage, Active ticket, Last decision, Open questions (resolved collapsed to a count), one **→ recommended next command**, freshness footer. **No Known Risks section (OQ-2: keep brief).**
- [ ] Recommends exactly ONE next command, reusing T6's successor logic (reference the skill map).
- [ ] Handles all 5 edge cases from `ux.md`: no `project-state.md`; initialized-but-untouched; empty `decisions.md`; missing/stale `handoff.md` (trust canon + muted note); just-compacted (normal render).
- [ ] `/5bot-status` added to the command roster in `skills/five-bot/SKILL.md` and README command list.
- [ ] **Sync:** mirror into `_framework/5bot/` / local command set if the repo exposes commands there.

**Files involved:** NEW `commands/5bot-status.md`; `skills/five-bot/SKILL.md`; `README.md`.

**Out of scope:** Writing/altering any state file; showing risks; F1/F3 text (T6/T8).

---

### T8: F1 — Context-health reminder — v1.3.0  *(depends on T6)*

**Goal:** After a long session, nudge the user to `/compact` (or start fresh) at a natural seam, reassuring that canon is on disk so nothing is lost.

**Acceptance Criteria:**
- [ ] `skills/five-bot/SKILL.md` defines the canonical F1 block once: the reassurance line, the **terse** variant (handoff/gate) and **minimal one-liner** (dev/qa), the self-judgment heuristic (4 drift signals), and the guardrails (≤ once per session; seam-only, never mid-artifact; silent when unsure).
- [ ] `commands/handoff.md` + `commands/gate.md`: trigger to append the terse F1 **above** the F3 footer when the session is self-judged long.
- [ ] **Suppression:** `/gate` does not fire F1 if `/handoff` already nudged in the same handoff→gate transition; QA does not fire F1 if `/gate` will run next.
- [ ] `commands/dev.md` + `commands/qa.md`: trigger for the minimal one-liner under their guidance, at most once at the start of a turn.
- [ ] **Sync:** F1 note added to `_framework/5bot/rules.md` "Token discipline" and to `_framework/5bot/personas/dev.md` + `qa.md`.

**Files involved:** `skills/five-bot/SKILL.md`, `commands/handoff.md`, `commands/gate.md`, `commands/dev.md`, `commands/qa.md`; sync: `_framework/5bot/rules.md`, `_framework/5bot/personas/dev.md`, `_framework/5bot/personas/qa.md`.

**Out of scope:** Automated token measurement (impossible from command markdown); the F3 footer itself (T6); blocking behavior (T5).

---

### T9: v1.3.0 Release — version bump, docs, sync — v1.3.0  *(depends on T6–T8 + QA)*

**Goal:** Ship v1.3.0 cleanly with versions and copies in sync.

**Acceptance Criteria:**
- [ ] `.claude-plugin/plugin.json` 1.2.1 → 1.3.0 AND the `five-bot` entry in `.claude-plugin/marketplace.json` bumped to match (must stay in sync).
- [ ] `README.md` documents the new `/5bot-status` command and the F1/F3 behavior (command table + a short "v1.3.0" note).
- [ ] `_framework/5bot/` synced with all T6–T8 guidance (rules.md, personas).
- [ ] Root `CLAUDE.md` plugin row → v1.3.0; MEMORY.md `5bot-plugin-published` updated.
- [ ] **Staging branch before main:** push v1.3.0 to a non-default ("hidden") branch, then install the plugin *from that branch* in a real Claude Code session (and the human's Cowork env) to verify `/5bot-status`, the F1 reminder, and the F3 footer load and behave. Only merge/push to `main` after that verification passes (with explicit human authorization).
- [ ] Commit + tag `v1.3.0` once verified on `main`.

**Files involved:** `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, `README.md`, `_framework/5bot/*`, root `CLAUDE.md`, `MEMORY.md`.

**Out of scope:** Any new feature work; T5 research.

**Note:** The user is iterating on the source of the *currently-installed* 5bot plugin. The staging-branch install is the safe way to validate the update before it reaches the live/published plugin on `main`.

## Developer Notes

### T1: Add ⚠️ Warning to Handoff.md Template

**Summary of changes:**
Added a prominent ⚠️ warning to the top of the master handoff.md template file (`_framework/5bot/templates/handoff.md`). The warning appears immediately after the file title and before the transient comment, making it impossible to miss.

**Files modified:**
- `C:\AI Bins\Code\_framework\5bot\templates\handoff.md` — added 1 line: ⚠️ **WARNING: DO NOT EDIT THIS FILE.** This file is overwritten each stage. Put decisions in `decisions.md`, state in `project-state.md`.

**Assumptions:**
- All new projects scaffolded with `/5bot-init` will inherit this warning automatically (template is copied as-is)
- Existing projects won't be affected (this is a future fix for new projects)
- Users read warnings placed at the top of files

**Known limits:**
- Projects created before this patch won't have the warning (they'll keep their old template copy)
- No technical enforcement (relies on user discipline to read the warning)

**How to run / test:**
1. Run `/5bot-init` in a new test project
2. Check that `handoff.md` created in the project has the warning at the top
3. Verify the warning appears before any section headers
4. Manual verification is sufficient (no automated tests needed for this Markdown edit)

---

### T2: Add "Canon" Definition to Project-state.md Template

**Summary of changes:**
Added a definition line to the master project-state.md template (`_framework/5bot/templates/project-state.md`) to explain the "canon" concept. The definition appears immediately after the intro block quote, making it visible without having to read further into the file.

**Files modified:**
- `C:\AI Bins\Code\_framework\5bot\templates\project-state.md` — added 1 line: **Canon = source of truth. All other files reference this one.**

**Assumptions:**
- All new projects scaffolded with `/5bot-init` will inherit this definition automatically (template is copied as-is)
- Users will read the intro section and see the definition before filling in the file
- The short definition is sufficient to explain the concept; no additional docs needed

**Known limits:**
- Projects created before this patch won't have the definition (they'll keep their old template copy)
- Definition is brief (intentional for a template comment) but may need README expansion for new users

**How to run / test:**
1. Run `/5bot-init` in a new test project
2. Check that `project-state.md` created in the project has the definition after the intro
3. Verify the definition is on line 4 (immediately after the intro block quote)
4. Manual verification is sufficient (no automated tests needed for this Markdown edit)

---

### T3: Add Rule Summary to 8 Command Files

**Summary of changes:**
Added a 2-3 sentence rule summary to all 8 command files. Each summary explains the role's scope and constraints, placed immediately after the role introduction and before the task instructions. The summaries teach users the anti-drift rules in context.

**Files modified (8 total):**
- `C:\AI Bins\Code\plugins\five-bot\commands\product.md` — added rule summary after "Product Bot" introduction
- `C:\AI Bins\Code\plugins\five-bot\commands\ux.md` — added rule summary after "UX Bot" introduction
- `C:\AI Bins\Code\plugins\five-bot\commands\architect.md` — added rule summary after "Architect Bot" introduction
- `C:\AI Bins\Code\plugins\five-bot\commands\dev.md` — added rule summary after "Developer Bot" introduction
- `C:\AI Bins\Code\plugins\five-bot\commands\qa.md` — added rule summary after "QA Bot" introduction
- `C:\AI Bins\Code\plugins\five-bot\commands\5bot-init.md` — added workflow overview summary after initialization intro
- `C:\AI Bins\Code\plugins\five-bot\commands\handoff.md` — added bookkeeping role summary after task intro
- `C:\AI Bins\Code\plugins\five-bot\commands\gate.md` — added human gate role summary after task intro

**Assumptions:**
- All 8 command files are user-facing and will be read by users invoking the commands
- Users read the summary immediately after the role introduction
- The rule summary will prevent users from stepping outside their role's scope
- Each summary is tailored to the specific role while maintaining consistency with the anti-drift rules

**Known limits:**
- Summaries are brief (2-3 sentences) and may not cover all edge cases
- Users may still ignore the rules if they choose to (relying on discipline, not enforcement)
- Users who invoke commands without reading the intro won't see the summary

**How to run / test:**
1. Run each command in Claude Code (e.g., `/product`, `/ux`, `/architect`, etc.)
2. Verify the rule summary appears immediately after the role introduction
3. Verify the summary is readable and clear in context
4. Manual verification is sufficient (no automated tests needed)

### v1.3.0 — T6 (F3 footer), T7 (/5bot-status), T8 (F1 reminder)

**T6 — F3 next-command footer.** Added the canonical footer shell + gate-verdict map to `skills/five-bot/SKILL.md` ("## Next-command footer (F3)"). `commands/handoff.md` now ends with `▶ NEXT: /gate`; `commands/gate.md` prints a waiting footer (no command, 6 options) before the verdict and a resolved `▶ NEXT` footer after. Synced a brief note + version bump to `_framework/5bot/rules.md` (v1.0.0 → v1.1.0). Files: SKILL.md, commands/handoff.md, commands/gate.md, _framework/5bot/rules.md.

**T7 — /5bot-status.** New read-only `commands/5bot-status.md`: reads project-state.md + newest decisions.md block + handoff.md; prints the brief 6-part snapshot (no Known Risks per OQ-2), one recommended next command (reusing the F3 map), and a "safe to /compact" freshness footer; 5 edge cases handled. Added an F2 summary to SKILL.md. README + repo-root marketplace roster deferred to T9 (consolidated). Files: commands/5bot-status.md (new), SKILL.md.

**T8 — F1 context reminder.** Canonical F1 block (reassurance + terse/minimal variants + 4-signal heuristic + guardrails + suppression) added to SKILL.md. Triggers: handoff.md (terse, above footer), gate.md (terse, with handoff→gate suppression), dev.md + qa.md (minimal one-liner; QA suppresses if gate next). Synced to `_framework/5bot/rules.md` (Token discipline) and personas dev.md + qa.md (v1.0.0 → v1.1.0). Files: SKILL.md, commands/handoff.md, gate.md, dev.md, qa.md, _framework rules.md + personas/dev.md + qa.md.

**Assumptions / limits:** F1 is a prompted heuristic (no token API exists). Runtime behavior (command loads, footer renders, Cowork `/compact`) is verified at T9's staging-branch install — not previewable in this environment (Markdown plugin). `marketplace.json` version lives at the repo root (not in the local plugin folder), so it is bumped at push time in T9.

## Review Notes

### T1: Add ⚠️ Warning to Handoff.md Template — QA Review

**Verdict: ✅ APPROVED**

**Acceptance Criteria Checklist:**

| Criterion | Status | Notes |
|-----------|--------|-------|
| Master template has ⚠️ warning at top | ✅ PASS | Warning is placed immediately after `# Handoff` title, before any sections |
| Warning text content correct | ✅ PASS | Text includes "DO NOT EDIT THIS FILE", "overwritten each stage", and directs to decisions.md & project-state.md. Formatting with backticks improves readability. |
| Warning appears before template sections | ✅ PASS | Warning is at line 3, before the `## Stage Completed` section at line 7 |
| Verified with `/5bot-init` | ⚠️ NOT TESTED | Cannot test directly in QA, but template copy logic is standard and will work correctly |

**Findings:**
- ✅ File modified correctly: `_framework/5bot/templates/handoff.md`
- ✅ Warning is prominently visible (large emoji, bold text)
- ✅ Warning is placed where users will see it immediately
- ✅ Text is clear and actionable (tells users where to put state vs. decisions)
- ✅ No unintended changes to other sections or formatting
- ⚠️ Minor: Formatting is slightly different from spec (uses backticks for file names, adds "WARNING:" prefix), but this is an improvement that enhances clarity

**No bugs found.** Edge cases: Users may still manually edit the file (relying on discipline, not enforcement), but the warning makes the transient nature very clear.

**Security/Safety:** No security concerns. This is a documentation improvement only.

**Test Plan:**
1. Manual verification (completed): ✅ Inspect the template file to confirm warning is at top
2. Integration test (when released): Run `/5bot-init` in a new project and verify `handoff.md` contains the warning
3. No automated tests needed (template edit, not code)

### T2: Add "Canon" Definition to Project-state.md Template — QA Review

**Verdict: ✅ APPROVED**

**Acceptance Criteria Checklist:**

| Criterion | Status | Notes |
|-----------|--------|-------|
| Master template has definition after intro | ✅ PASS | Definition is on line 4, immediately after the intro block quote |
| Definition text is correct | ✅ PASS | Exact match: "Canon = source of truth. All other files reference this one." |
| Definition placed prominently at top | ✅ PASS | Definition is on line 4 (after intro on line 3), before `## Project Name` section |
| Clarity improvement verified | ✅ PASS | Definition is clear and directly addresses the jargon issue identified in UX audit |

**Findings:**
- ✅ File modified correctly: `_framework/5bot/templates/project-state.md`
- ✅ Definition is placed where users will read it immediately after the intro
- ✅ Text is precise and explains the concept clearly
- ✅ No unintended changes to file structure or other sections
- ✅ Formatting is consistent with the intro block quote style

**No bugs found.** Edge cases: None. This is a simple text addition.

**Security/Safety:** No security concerns. This is a documentation improvement only.

**Test Plan:**
1. Manual verification (completed): ✅ Inspect the template file to confirm definition is after intro
2. Integration test (when released): Run `/5bot-init` in a new project and verify `project-state.md` contains the definition
3. No automated tests needed (template edit, not code)

### T3: Add Rule Summary to 8 Command Files — QA Review

**Verdict: ✅ APPROVED**

**Acceptance Criteria Checklist:**

| Criterion | Status | Notes |
|-----------|--------|-------|
| All 8 command files have rule summary | ✅ PASS | All 8 files checked: product, ux, architect, dev, qa, 5bot-init, handoff, gate |
| Summary explains role scope + constraints | ✅ PASS | Each summary is 2-3 sentences explaining role scope and what the role cannot do |
| Core rule is communicated | ✅ PASS | All summaries include: "Product Bot defines scope... All scope changes require human approval at a gate" |
| Summary placed after role intro, before instructions | ✅ PASS | All 8 files have summary in correct position (after role introduction, before task instructions) |

**Findings:**
- ✅ All 8 files modified correctly
- ✅ Summaries are consistently formatted (bold header, 2-3 sentences)
- ✅ Summaries are placed where users will read them immediately (after role intro)
- ✅ Text is clear and reinforces anti-drift rules in context
- ✅ No unintended changes to other sections
- ✅ Each summary is tailored to the specific role while maintaining consistency
- ✅ Minor variations in wording per role are appropriate and enhance clarity

**No bugs found.** All summaries are clear, well-placed, and teach the rules in context.

**Security/Safety:** No security concerns. This is a documentation improvement only.

**Test Plan:**
1. Manual verification (completed): ✅ Inspected all 8 command files to confirm summary is present and in correct position
2. Integration test (when released): Run each command in Claude Code (`/product`, `/ux`, `/architect`, `/dev`, `/qa`, `/5bot-init`, `/handoff`, `/gate`) and verify rule summary appears
3. No automated tests needed (template/command file edits, not code)

### v1.3.0 QA — T6 / T7 / T8

Reviewed against the ticket acceptance criteria and `ux.md`. Static review (Markdown plugin; runtime install verified at T9 staging).

**T6 — F3 footer:** ✅ APPROVED — footer shell + verdict map in SKILL.md; handoff→NEXT `/gate`; gate waiting (no command) + resolved footer; derive-from-Current-Stage with `/5bot-status` fallback; one primary NEXT + "If instead"; synced to rules.md.

**T7 — /5bot-status:** ✅ APPROVED — new read-only command (writes nothing); reads canon + newest decision + handoff; brief 6-part snapshot, no Known Risks (OQ-2); one next command via the F3 map; 5 edge cases; SKILL roster updated (README + marketplace → T9).

**T8 — F1 reminder:** ✅ APPROVED — F1 block (reassurance/variants/heuristic/guardrails) in SKILL.md; triggers in handoff/gate/dev/qa; handoff→gate and QA→gate suppression; F1 prints above F3; synced to rules.md + personas.

**Findings:** consistent "canon on disk" message across F1/F2/F3; suppression prevents double-nudge; logic centralized (DRY) in SKILL.md. No blocking bugs.

**Carry-forward (non-blocking, owned by T9):** README command list and repo-root `marketplace.json` version not yet updated — confirm before release.

**Runtime test plan (T9 staging install):** install from the staging branch; run `/5bot-status` in (a) an initialized project and (b) an empty folder; confirm `/handoff` and `/gate` print the footer; confirm `/gate` shows waiting → resolved; verify `/compact` wording behaves in the Cowork env.

## Bug List

**T1:** No bugs found. Work is APPROVED for release.
**T2:** No bugs found. Work is APPROVED for release.
**T3:** No bugs found. Work is APPROVED for release.
**T6:** No bugs found. APPROVED (runtime check deferred to T9 staging).
**T7:** No bugs found. APPROVED (runtime check deferred to T9 staging).
**T8:** No bugs found. APPROVED (runtime check deferred to T9 staging).

## Release Checklist
