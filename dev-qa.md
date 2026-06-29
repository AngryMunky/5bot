# Dev & QA (v2.2.0)

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

## v2.1.0 Backlog — Claude Design Integration (zip-first)

> Sequence: **T10 → { T11, T12 parallel-ok } → T13 → T14.** T15 is DEFERRED (post-MVP). Canonical edits in `plugins/5bot/`; sync guidance into `_framework/5bot/`. See `architecture.md` → "v2.1.0". DRY: the design-reference procedure lives once in `SKILL.md`; commands carry only triggers.

### T10: Skill — Design Reference procedure — v2.1.0

**Goal:** One canonical home in the skill for the design-reference logic, so `/ux` and `/dev` don't duplicate it.

**Acceptance Criteria:**
- [ ] `skills/five-bot/SKILL.md` gains a "## Design reference (Claude Design import)" section.
- [ ] Defines the **Design Reference block schema** exactly as in `architecture.md` / `ux.md` D4.
- [ ] Defines the **zip procedure:** locate the user-provided `.zip`; normalize to `design/<slug>.zip`; extract to `design/<slug>/` (give BOTH PowerShell `Expand-Archive` and unix `unzip`/`tar` commands — agent picks per OS, OQ-9); identify the primary `*.dc.html`.
- [ ] States the **guardrails:** `ux.md` is canon; the design is a linked aid; never auto-generate the whole design; capture is UX-only (Dev consumes).
- [ ] Notes import-method values (`fallback-zip` now; `connector` reserved for the deferred enhancement).
- [ ] Plain Markdown only (Cowork-safe).

**Files involved:** `skills/five-bot/SKILL.md`.

**Out of scope:** connector/MCP import (T15); editing `/ux` or `/dev` (T11/T12); generating actual designs.

---

### T11: `/ux` design-reference step — v2.1.0  *(depends on T10)*

**Goal:** Let `/ux` optionally capture a Claude Design `.zip` into a Design Reference block (Flow D) without changing normal `/ux` when no design is given.

**Acceptance Criteria:**
- [ ] `commands/ux.md` gains a short **trigger**: if the user supplies a Claude Design `.zip` path (or pastes a project URL / "Send to local coding agent" prompt), follow the skill's Design Reference procedure; otherwise proceed with normal UX **silently** (zero friction when unused).
- [ ] On the zip path: extract per the skill, then write the `## Design Reference (Claude Design)` block into the project's `ux.md` with real paths + date.
- [ ] If only a URL is available (no zip yet): record a `fallback-url-only` reference and recommend exporting a zip for durability (ux.md D3/D5) — never block.
- [ ] Parses a pasted "Send to local coding agent" prompt to seed `{project URL, file name}`.
- [ ] Trigger references the skill (no duplicated procedure text). Plain Markdown.

**Files involved:** `commands/ux.md` (+ skill from T10).

**Out of scope:** the procedure itself (T10); Dev consumption (T12); connector import (T15).

---

### T12: `/dev` consumes the Design Reference — v2.1.0  *(depends on T10)*

**Goal:** Dev builds against the recorded design when a ticket cites it.

**Acceptance Criteria:**
- [ ] `commands/dev.md` gains a short **trigger**: when the active ticket references a Design Reference (or `ux.md` has one relevant to the ticket), read the linked `design/<slug>/<Name>.dc.html` (+ assets) and implement against it.
- [ ] Explicit: implement the **ticket's scope only** — do NOT auto-generate the entire design; `ux.md` / the ticket remain the spec.
- [ ] If the linked artifact is missing, say so and fall back to the `ux.md` text spec (never block).
- [ ] Trigger references the skill's guardrails. Plain Markdown.

**Files involved:** `commands/dev.md`.

**Out of scope:** capturing the design (T11); connector import (T15).

---

### T13: Docs, template & `_framework` sync — v2.1.0

**Goal:** Document the feature and keep the two synced copies aligned.

**Acceptance Criteria:**
- [ ] `README.md` documents the optional Claude Design design-reference step: the zip workflow ("Download zip instead" → drop in repo → `/ux` records it → Dev builds against it), the `design/` convention, and "ux.md stays canon; native Claude Design only (Figma out)".
- [ ] `_framework/5bot/templates/ux.md` (master) gains a brief commented note that an optional `## Design Reference (Claude Design)` section may be added (no structural change).
- [ ] `_framework/5bot/` synced: personas `ux.md` (capture behavior) + `dev.md` (consume behavior) note the design-reference step; bump touched persona versions per the semver rule.
- [ ] Command roster unchanged (no new command — modified `/ux` + `/dev`); confirm nothing else references a phantom command.

**Files involved:** `README.md`, `_framework/5bot/templates/ux.md`, `_framework/5bot/personas/ux.md`, `_framework/5bot/personas/dev.md`.

**Out of scope:** version bump / publish (T14); connector docs beyond a "later" mention.

---

### T14: v2.1.0 Release — version bump, docs, dual-push — v2.1.0  *(depends on T10–T13 + QA)*

**Goal:** Ship v2.1.0 with versions and both repos in sync.

**Acceptance Criteria:**
- [ ] `5bot/.claude-plugin/plugin.json` 2.0.0 → 2.1.0; mirror the same number into the `5bot` entry of the repo-root `.claude-plugin/marketplace.json` (must match).
- [ ] Root `README.md` version → 2.1.0; plugin `README.md` notes the v2.1.0 feature; stage files re-stamped as needed.
- [ ] `_framework/5bot/` synced (T13 changes present).
- [ ] Root `CLAUDE.md` plugin row → v2.1.0; MEMORY.md `5bot-plugin-published` updated.
- [ ] Push: `git push origin main` (**dual-push → private `5bot-plugin` + public `5bot`**), tag `v2.1.0`, push tags to both (`git push origin --tags`; `git push public --tags`) — per `PUBLISH.md`. **Explicit human authorization required** (public + `main`).
- [ ] (Optional) staging-branch install verify before `main`, as in T9.

**Files involved:** `.claude-plugin/plugin.json`, repo-root `.claude-plugin/marketplace.json`, READMEs, `_framework/5bot/*`, root `CLAUDE.md`, `MEMORY.md`.

**Out of scope:** new feature work; the connector enhancement (T15).

---

### T15: (DEFERRED, post-MVP) Connector import via `claude_design` MCP — future

**Status: DEFERRED** — not built in v2.1.0; the deferred D2 path.

**Goal:** When the connector is present, import the design directly via the `claude_design` MCP (D2) instead of a manual zip.

**Acceptance Criteria (design-time, when picked up):**
- [ ] Resolve **OQ-6:** the real connect/authorize path per surface (is `/design-login` a real command in Code/Cowork/web?); confirm the connector's import/fetch tools.
- [ ] Add a connector branch to the skill procedure: detect connector → import → same Design Reference block (`import method: connector`), with graceful fallback to the zip path when absent/unauthorized.
- [ ] Never block; never instruct an unverified connect command (the original pain point).

**Files involved:** `skills/five-bot/SKILL.md`, `commands/ux.md`.

**Out of scope:** everything in the v2.1.0 MVP; auto-build.

---

## Developer Notes

### T14: v2.1.0 Release — version bump, docs, dual-push — v2.1.0

**Summary:** Phase 1 (safe, reversible) is done — the canonical version + docs are bumped in the working source. Phase 2 (the irreversible dual-push to the **public** mirror + `main`) is **documented but NOT executed** — it requires explicit human authorization at the T14 QA gate.

**Files modified — Phase 1 (working source + root, reversible):**
- `plugins/5bot/.claude-plugin/plugin.json` — `version` 2.0.0 → **2.1.0** (the single canonical version spot).
- `plugins/5bot/README.md` — title → v2.1.0; new "**v2.1.0 — Claude Design reference (optional)**" entry in Recent improvements; footer → v2.1.0.
- `plugins/5bot/project-state.md` — title re-stamp → v2.1.0.
- `C:/AI Bins/Code/CLAUDE.md` (root index) — `plugins\5bot\` row → v2.1.0.

**`_framework/5bot/` sync:** confirmed T13's changes are present (personas `ux.md`/`dev.md` → v1.2.0, template → v0.1.1). `_framework/5bot/` is the local working copy used by the userSettings commands — it is **not** part of the published plugin repo, so there's nothing to publish there; this AC is a consistency check, satisfied.

**Clone ↔ source mapping (verified this session; canonical clone present + clean, `main…origin/main`):**
- working `commands/`, `skills/`, `README.md`, `.claude-plugin/plugin.json` → clone **`5bot/`** subfolder (the installable plugin).
- working framework stage files (`architecture.md`, `decisions.md`, `dev-qa.md`, `handoff.md`, `product.md`, `project-state.md`, `ux.md`) → clone **root**.
- clone-only files: `.claude-plugin/marketplace.json` (repo-root marketplace — bump the `5bot` entry 2.0.0 → 2.1.0 to match `plugin.json`), root `README.md` (separate GitHub-landing page — bump any version ref), plus `LICENSE`/`PUBLISH.md`/`.gitignore` (unchanged). Working `CLAUDE.md` is **not** published.

**PENDING — Phase 2 (EXPLICIT HUMAN AUTHORIZATION REQUIRED; public + `main`, irreversible), per `PUBLISH.md`:**
1. Sync working source → clone (`5bot/*` + root stage files).
2. In clone: bump `.claude-plugin/marketplace.json` `5bot` entry → 2.1.0 (must equal `plugin.json`); update root `README.md` version.
3. `git add -A`; show `git status` + `git diff --stat` for a final eyeball.
4. Commit as **Angry Munky** (AngryMunky account + noreply commit email — per the GitHub-identity rule).
5. `git push origin main` (dual-push → private `5bot-plugin` + public `5bot`); `git tag v2.1.0`; `git push origin --tags`; `git push public --tags`.
6. Post-push: update the `5bot-plugin-published` memory with the new commit hash + v2.1.0.

**How to test (post-push):** `git -C <clone> log --oneline -1` + `git -C <clone> tag` shows `v2.1.0`; optionally `gh` to confirm both repos advanced; consumers get it via `/plugin update`. (Optional staging-branch verify before `main`, as in T9.)

**Known limits:** plugin.json and marketplace.json versions MUST match (the v1.1.0 drift bug). The clone lives in a Windows Temp dir — verified present this session; if ever cleaned, re-clone per `PUBLISH.md` before pushing.

### T13: Docs, template & `_framework` sync — v2.1.0

**Summary of changes:**
Documented the optional Claude Design design-reference feature in the plugin README, added a commented hint to the master `ux.md` template, and synced the `_framework/5bot/` personas (UX captures, Dev consumes) with version bumps. Also applied the deferred D4 fix to the project's own `ux.md`. No code/command behavior changed — docs + sync only. Command roster confirmed unchanged (9 commands; T11/T12 modified existing `/ux` + `/dev`, added none).

**Files modified:**
- `README.md` — new "## Optional: a Claude Design reference (UX stage)" section (zip workflow, `design/` convention, "ux.md stays canon; native Claude Design only — Figma out"); added a one-line pointer to it from the UX-role description. *Version header left at v2.0.0 — the bump to 2.1.0 + a release note is T14's job.*
- `_framework/5bot/templates/ux.md` — brief HTML-comment note that an optional `## Design Reference (Claude Design)` section may be added; no structural change. PATCH bump v0.1.0 → v0.1.1 (touched framework file).
- `_framework/5bot/personas/ux.md` — replaced the old "Visual aid (optional)" image-mockup paragraph with a "Design reference (optional)" paragraph describing the zip-capture step (points at the `five-bot` skill procedure; canon + Figma-out guardrails). MINOR bump v1.1.0 → v1.2.0 (new behavior).
- `_framework/5bot/personas/dev.md` — added a "Design reference (optional)" paragraph: consume the linked `design/<slug>/<Name>.dc.html` when a ticket cites it, ticket-scope only, fall back to text spec, never block. MINOR bump v1.1.0 → v1.2.0 (new behavior).
- `ux.md` (project, D4 block) — **deferred item:** narrowed the import-method example from `connector | fallback-zip | fallback-url-only` to `fallback-zip` with the `# MVP; "connector" reserved (T15)` note, matching the canonical SKILL.md schema.

**Assumptions:**
- README placement: the dedicated design-reference section sits between "The five roles" and "The shared files"; the inline UX-role mock-up note is kept (still accurate) and cross-links to it.
- Semver: a comment-only template change is a PATCH; new persona behavior is a MINOR (per the global semver rule on framework files).
- T14 owns the version bump (plugin.json/marketplace/root README → 2.1.0) and the plugin README's release note — so the plugin README's top version intentionally still reads v2.0.0 after this ticket.

**Known limits / notes for QA:**
- The recorded deferred scope named only the **import-method** line of D4. While there, I noticed `ux.md` D4 still shows `- Local artifact: design/tag-hag.dc.html (committed) | none (URL only)`, which predates the approved SKILL.md schema (`Original zip:` + `Local artifact: design/<slug>/<Name>.dc.html`). I **did not** edit that line — it's outside the recorded deferred scope and `ux.md` is the UX bot's artifact. Flagging it for QA to rule: tighten now (within "docs sync" spirit) or leave for a UX pass.
- `_framework/5bot/templates/` has no `dev-qa.md`/persona structural change needed beyond the above; the plugin's own `commands/` + `SKILL.md` were already synced in T10–T12.

**How to verify:**
- `README.md`: the new section documents the three required points (zip workflow, `design/` convention, canon + Figma-out).
- `_framework` files: open the two personas and the template; confirm the design-reference notes and bumped version headers.
- Roster: `commands/` lists the same 9 commands; README's command table is unchanged.

### T12: `/dev` consumes the Design Reference — v2.1.0

**Summary of changes:**
Added an optional "Building against a Claude Design reference" trigger to `commands/dev.md`. When the active ticket cites a Design Reference (or `ux.md` has a relevant block), the Dev Bot reads the linked `design/<slug>/<Name>.dc.html` (+ assets) and implements against it — scoped to the ticket only, never auto-generating the whole design. Falls back to the `ux.md` text spec if the artifact is missing. References the skill's guardrails (no duplication).

**Files modified:**
- `commands/dev.md` — new "## Optional: Building against a Claude Design reference" section, placed after the scope paragraph and before the "When done, run `/qa`" line (mirrors T11's placement in `ux.md`).

**What was added:**
- **Consume trigger:** if the ticket cites a Design Reference (or `ux.md` has one relevant to the ticket), read the linked `design/<slug>/<Name>.dc.html` + assets and build against it.
- **Scope guard:** implement the ticket's scope only — never auto-generate the entire design. `ux.md` / the ticket remain the spec; design is a linked aid.
- **Missing-artifact fallback:** if the linked artifact is missing or stale, say so plainly and fall back to the `ux.md` text spec — never block.
- **Guardrail reference:** points to the `five-bot` skill's Design Reference guardrails (no duplicated logic — DRY).

**Assumptions:**
- The active ticket (or `ux.md`) will name the Design Reference when one is relevant; the Dev Bot does not hunt for designs unprompted.
- The extracted `*.dc.html` written by the T11 `/ux` capture step is at `design/<slug>/<Name>.dc.html` per the OQ-7 storage decision.
- A ticket may legitimately have no design reference — the trigger is a silent no-op in that case (no friction).

**Known limits:**
- Trigger is instructional text; no runtime code. Actual artifact reads happen when `/dev` runs on a design-citing ticket.
- "Stale" detection is judgment-based (e.g., a design that no longer matches `ux.md`); no automated diff. The guardrail tells Dev to prefer `ux.md` when in doubt.
- Does not parse the Design Reference block fields itself — relies on the linked path being present in `ux.md` (written by T11).

**Pairing with T11:**
- T11 (`/ux`) **captures** a design into the Design Reference block (write side).
- T12 (`/dev`) **consumes** it when a ticket cites it (read side).
- Both reference the single SKILL.md procedure/guardrails (DRY); neither duplicates the logic.

**QA-bounce fixes (round 1 — both obvious, in-scope):**
1. Tightened T12 AC #1 text (dev-qa.md): `design/<slug>/*.dc.html` → `design/<slug>/<Name>.dc.html`, so the acceptance criterion matches the spec (SKILL.md / architecture.md / ux.md) and the dev.md impl. The deliverable was already correct; the AC text was the lone outlier.
2. Added an explicit no-op sentence to `commands/dev.md` ("If the ticket cites no Design Reference and `ux.md` has no relevant block, proceed with normal implementation — no friction."), mirroring T11's explicit skip path for clarity parity.

Deferred per QA (out of T12 scope): `ux.md` D4 import-method example breadth → T13 docs-sync / T11 follow-up. SKILL.md heading finding → refuted (not a defect).

---

### T11: `/ux` design-reference step — v2.1.0

**Summary of changes:**
Added an optional "Claude Design Import (Design Reference)" trigger section to `commands/ux.md`. The trigger asks if the user has a Claude Design file/URL, explains the optional capture workflow, and points to the canonical procedure in `SKILL.md`. If no design is provided, UX proceeds silently with zero friction (normal UX flow unchanged).

**Files modified:**
- `commands/ux.md` — new "## Optional: Claude Design Import (Design Reference)" section added after scope rules and before existing instructions.

**What was added:**
- **Trigger section** (8 lines): explains the optional Claude Design capture step.
- **Three input paths:** URL, "Send to local coding agent" prompt, or zip file path.
- **Procedure reference:** points to `SKILL.md` Design Reference procedure (locate → normalize → extract → identify → write block). No duplicated logic.
- **Fallback guidance:** if URL-only (no zip), record and recommend export. Never block.
- **Silent no-op:** if user says "no design", skip and proceed with normal UX.
- **DRY principle:** trigger is a short reference; actual procedure lives once in SKILL.md.

**Assumptions:**
- Users can paste a Claude Design URL or "Send to local coding agent" prompt.
- Alternatively, users can provide a file path to a locally saved `.zip`.
- The agent can parse a pasted prompt to extract project URL + file name.
- If no design available, users will say so (or skip this step).

**Known limits:**
- Trigger is text-only; no GUI form for file selection. User provides the zip path as text (same as existing command interface).
- Design Reference block write-up is deferred to the skill procedure invocation (T11 just asks; the skill does the work).
- Runtime testing (actual zip extraction, block writing) happens when the trigger fires in a real `/ux` run.

**Placement rationale:**
- Added after scope rules (so users know this is part of UX's scope) but before the main task instructions.
- Positioned as **completely optional** to maintain zero friction for projects without a design.
- Uses the exact wording from `ux.md` "v2.1.0" design (Flow D, states D1–D5) to stay consistent.

---

## Review Notes

### T14: v2.1.0 Release — QA Review

**Verdict: ✅ APPROVED WITH NOTES** — **Phase 1** (version + doc bumps in the working source) is correct and version-consistent. **Phase 2** (the irreversible dual-push to the **public** mirror + `main`) is correctly **gated, not executed** — the human gate is the explicit authorization. Three notes below should be ruled on/confirmed *before* the push, since the public push is irreversible.

**Phase 1 — verified on disk:**
- ✅ `plugin.json` (source, canonical spot) = **2.1.0**.
- ✅ plugin `README.md`: title v2.1.0, new "v2.1.0 — Claude Design reference (optional)" Recent-improvements entry, footer v2.1.0.
- ✅ `project-state.md` title re-stamped v2.1.0.
- ✅ root `CLAUDE.md` `plugins\5bot\` row → v2.1.0.
- ✅ Version-straggler sweep: remaining `2.0.0` hits in the source are all legitimate history (README/architecture changelog entries, `decisions.md` log, `product.md` risk text, "v2.0.0 shipped" provenance) — none are erroneous.
- ✅ `_framework/5bot/` carries T13's changes (personas v1.2.0, template v0.1.1); it's the local working copy, not a publish target.

**Phase 2 — plan verified sound (clone present + clean, mapping confirmed file-by-file); NOT executed.** On authorization: sync source→clone, bump `marketplace.json` `5bot` entry → 2.1.0 (must equal `5bot/.claude-plugin/plugin.json`), bump root landing `README.md` (`Version 2.0.0`→2.1.0 line 3, footer line 116, + v2.1.0 changelog entry), commit as **Angry Munky** (noreply), `git push origin main` (dual), tag `v2.1.0`, push tags to origin + public.

**Bug List / Findings:**
1. **[finding · recommend cleanup — gate decision] Orphaned root `.claude-plugin/plugin.json` @ v1.2.1.** The clone has a *third* version file at the repo root (`.claude-plugin/plugin.json`) stuck at **1.2.1** — stale since before v2.0.0. It is **off the install path** (the marketplace `source: ./5bot` resolves the manifest to `5bot/.claude-plugin/plugin.json`), so it doesn't break installs and isn't a v2.1.0 regression. But it's confusing dead weight and a latent version-drift landmine (cf. PUBLISH.md's v1.1.0 lesson). **Recommend: delete it** (cleanest) — or bump to 2.1.0. Not done here: deleting a file / resolving its fate is a human call, not a unilateral QA edit. **Note: PUBLISH.md does not mention this file** — the documented procedure only covers `5bot/.claude-plugin/plugin.json` + `marketplace.json` (which DO stay in sync at 2.1.0).
2. **[note · PRIVACY — confirm before public push] The public mirror publishes all internal dev docs.** Git-tracked at the repo root (therefore pushed to the **public** `AngryMunky/5bot`): `dev-qa.md` (every ticket + full QA history, incl. adversarial review notes), `decisions.md` (approval log), `handoff.md` (transient), `project-state.md` — which now includes the **Deployment** note (the private/public two-repo strategy, the local **Temp clone path**, and internal follow-up task IDs like `task_2dbc68f5`). This is the **established practice** (v2.0.0 already published these), but v2.1.0 expands the internal detail. Since publishing to public is irreversible/indexable, the human should explicitly confirm — or we exclude internal docs (gitignore `dev-qa.md`/`decisions.md`/`handoff.md`/`project-state.md`, or publish a leaner tree). Out of T14's recorded scope to change unilaterally.
3. **[minor · optional consistency] Stage-file header re-stamp.** `project-state.md` → v2.1.0 (done); `product.md`/`architecture.md`/`ux.md` line-1 headers still read v2.0.0. The v2.0.0 release re-stamped all of them; the global semver rule says bump only the canonical spot (plugin.json — done). So this is optional internal-doc polish, not a version-of-record issue. Decide at the gate.

**Test Plan:** Markdown-only plugin; no runtime. Phase-1 verification = the on-disk checks above (done). Phase-2 verification (post-authorization, post-push): in the clone, `git diff --stat` before commit shows only the intended v2.1.0 changes; after push, `5bot/.claude-plugin/plugin.json` and `marketplace.json` both read **2.1.0** (must match); `git tag` shows `v2.1.0`; `git log --oneline -1` is the Angry-Munky release commit; optionally `gh repo view` both repos and `/plugin update` picks up 2.1.0. (Optional: staging-branch install verify before `main`, as in T9.)

### T13: Docs, template & `_framework` sync — QA Review

**Verdict: ✅ APPROVED WITH NOTES** — all four T13 acceptance criteria met; the one recorded deferred fix (project `ux.md` D4 import-method → `fallback-zip`) landed correctly. The notes below are follow-ups *outside* T13's recorded scope.

**Reviewed via** an independent 3-lens adversarial pass (schema-consistency vs the canonical SKILL.md block, AC-completeness, stale-reference sweep) plus a dedicated scope-adjudication agent on the one borderline finding. No blockers, no majors.

**Acceptance Criteria — all met:**
- ✅ AC1 (README): the new "## Optional: a Claude Design reference (UX stage)" section documents all three required points — the zip workflow ("Download zip instead" → drop in repo → `/ux` records it → Dev builds against it), the `design/` convention (extracted → `design/<slug>/`, pristine source → `design/<slug>.zip`, primary artifact → `design/<slug>/<Name>.dc.html`, created on first use), and "ux.md stays canon; native Claude Design only — Figma out".
- ✅ AC2 (template): `_framework/5bot/templates/ux.md` gains a brief HTML-comment note about the optional `## Design Reference (Claude Design)` section; all nine `##` headers structurally unchanged. PATCH → v0.1.1.
- ✅ AC3 (`_framework` sync): personas `ux.md` (capture) + `dev.md` (consume) note the design-reference step and point at the `five-bot` skill (no duplication); both MINOR-bumped v1.1.0 → v1.2.0 (correct for added behavior).
- ✅ AC4 (roster): exactly 9 command files, unchanged; README's command table matches; a full sweep of README + `commands/ux.md` + `commands/dev.md` + SKILL.md found **no phantom command** references (apparent matches are built-ins `/plugin` `/compact` or phrases/paths).
- ✅ Deferred fix: project `ux.md` D4 import-method narrowed to `fallback-zip` (+ "connector reserved (T15)" note), matching the canonical SKILL.md schema.

**Bug List:** none blocking — no defect fails an acceptance criterion.

**Notes (follow-ups, OUT of T13's recorded scope — for human triage at the gate):**
1. **[minor · UX-bot follow-up] Project `ux.md` Design Reference copy (D2/D3/D4/D5 + Nav/Screen-List/Buttons) lags the as-built feature.** D4 omits the canonical `Original zip:` line and its `Local artifact:` still shows the flat `design/tag-hag.dc.html` (pre-OQ-7) instead of nested `design/<slug>/<Name>.dc.html`; D2/D3/D5 still frame the connector as a live runtime branch rather than deferred-to-T15. The **shipped contract (SKILL.md) is correct**, so runtime behavior is right — only the design doc's examples lag. This is the UX bot's artifact and lane; the recorded deferred fix named ONLY the import-method line (done). A partial D4 fix would leave D3/D5 inconsistent (strictly worse); a full re-sync is a `/ux` re-pass. **Recommend a follow-up `/ux` ticket:** "Re-sync `ux.md` Design Reference block to the as-built SKILL.md schema + connector-deferred (T15) reality."
2. **[minor · polish; README is in-scope] README shared-files glossary (line ~227)** still describes `ux.md` generically as "Can include a link to a visual mock-up." Not wrong (a `.dc.html` is arguably a mock-up) and the dedicated section + UX-role line tell the correct story — but it could be tightened to reference the Design Reference convention. Non-blocking; human may fold into T13 or defer.
3. **[minor · T11/T12-owned file] `commands/ux.md` line 12** still carries the image-era "export a mock-up… link it" clause directly above the new zip Design Reference section (a double-story). NOT in T13's recorded file list (T11/T12 own the command files). Recommend folding into the same follow-up.

**Test Plan:** T13 is documentation/sync (Markdown-only plugin; no runtime here). Verification = the read-checks already performed: (a) README section contains the 3 AC1 points; (b) template comment present, headers unchanged, v0.1.1; (c) both personas note the step + v1.2.0; (d) `commands/` lists 9 files, README table matches, no phantom command. Runtime exercise of the real zip → `.dc.html` → Dev path is deferred to the T14 staging-branch install alongside T10/T11/T12.

---

### T12: `/dev` consumes the Design Reference — QA Review

**Verdict: ✅ APPROVED (round 2, after fixes)** — round 1 was NEEDS CHANGES (obvious fixes, single auto-bounce).

**Round 2 (re-review after Dev fixes):** Both round-1 fixes verified applied with no regression:
- ✅ `dev-qa.md` AC #1 now reads `design/<slug>/<Name>.dc.html` — matches spec (SKILL.md / architecture.md / ux.md) and the dev.md impl. The lone cross-file inconsistency is resolved.
- ✅ `commands/dev.md:16` now states the no-op path explicitly ("…proceed with normal implementation — no friction."), at parity with T11.
- All 4 acceptance criteria remain met; trigger text unchanged otherwise. T12 is clean.

**Round 1 detail (retained for the record):** Reviewed via adversarial multi-lens pass (4 lenses: acceptance-criteria, cross-file consistency, guardrails/anti-drift, edge-cases; each finding adversarially verified). The **deliverable** (`commands/dev.md` trigger) met all 4 T12 acceptance criteria — 3 lenses fully passed; edge cases all handled. One real cross-file inconsistency and one in-scope clarity nit were fixed via the single auto-bounce.

**Acceptance Criteria — deliverable (dev.md):**
- ✅ AC1: trigger reads linked `design/<slug>/<Name>.dc.html` + assets when a ticket cites a Design Reference (dev.md:14)
- ✅ AC2: explicit ticket-scope-only guard; never auto-generate whole design; `ux.md`/ticket remain spec
- ✅ AC3: missing/stale artifact → say so, fall back to `ux.md` text spec, never block
- ✅ AC4: references the skill's guardrails (no duplication); plain Markdown

**Bug List (obvious fixes for the Dev bounce):**
1. **[minor · in-scope] AC-text wildcard mismatch** — `dev-qa.md` T12 AC #1 (line ~232) says `design/<slug>/*.dc.html` (glob), but the spec (SKILL.md, architecture.md, ux.md) and the actual dev.md impl use the precise `design/<slug>/<Name>.dc.html`. The deliverable is correct; the AC text is the outlier and contradicts canon. **Fix:** tighten the AC text to `design/<slug>/<Name>.dc.html`. (Not covered by T13's docs-sync scope, so fix here.)
2. **[nit · in-scope] implicit no-op path** — dev.md's `if [design cited]` structure leaves the "no design → proceed normally" path implicit; T11's `/ux` makes it explicit ("If you don't have a design, just say so…"). **Fix:** add one sentence mirroring T11 for parity/clarity.

**Notes (NOT in this bounce):**
- **[nit · out of T12 scope → T13/T11 follow-up]** `ux.md` D4 (line ~276) lists `Import method: connector | fallback-zip | fallback-url-only` as a choice, but SKILL.md/architecture.md show the single MVP value `fallback-zip` (connector reserved/deferred to T15). This is a UX-doc breadth issue in the block T11 writes, not T12's consume path (which defers to the skill guardrails). Fold into T13 docs-sync or a T11 follow-up.
- **[refuted]** A heading "mismatch" between SKILL.md section title `## Design Reference (Claude Design import)` (line 83) and the code-block template `## Design Reference (Claude Design)` (line 89) is **not a defect**: the title labels the procedure; the code block is the literal template written into downstream `ux.md` (and correctly omits "import"). Different by design.

**Test Plan:** T12 is instruction text (Markdown-only plugin; no runtime here). After the two fixes, re-verify dev.md trigger text against the 4 ACs and confirm the AC text now matches the spec. Runtime behavior (Dev reading a real `design/<slug>/<Name>.dc.html` on a design-citing ticket, and the missing-artifact fallback) is exercised at the T14 staging-branch install, alongside T10/T11.

---

### T11: `/ux` design-reference step — QA Review

**Verdict: ✅ APPROVED**

**Acceptance Criteria — all met:**
- ✅ Short trigger added to commands/ux.md (new "## Optional: Claude Design Import" section, lines 14–25)
- ✅ If user supplies `.zip` / URL / prompt: trigger invokes SKILL.md procedure
- ✅ If URL-only: record `fallback-url-only` + recommend export (never block)
- ✅ Parses "Send to local coding agent" prompt for {project URL, file name}
- ✅ Trigger references skill (no duplicated procedure text)
- ✅ Proceeds with normal UX silently when no design (zero friction)
- ✅ Plain Markdown only (Cowork-safe)

**QA findings:**
- **No bugs.** Trigger is clear, comprehensive, and user-friendly.
- **No edge cases missed.** All three input paths (URL, prompt, zip) documented. Fallback (URL-only) covers the "missing zip" case.
- **No security concerns.** Local file I/O only (zip path provided by user); no external integrations.
- **Zero friction maintained.** Silent no-op when design not provided keeps normal UX unaffected.
- **DRY principle:** Procedure lives in SKILL.md; trigger just invokes it. No logic duplication.

**Test plan:** T11 is instructional text (no runtime code). Runtime testing deferred to real `/ux` runs with design reference inputs. QA will observe zip extraction, Design Reference block write, and fallback behavior in practice.

---

### T10: Skill — Design Reference procedure — v2.1.0

**Summary of changes:**
Added a canonical "## Design Reference (Claude Design import)" section to `SKILL.md` with the complete design-reference procedure. This is the DRY home for the logic; `/ux` (T11) and `/dev` (T12) will reference it with short triggers only (no duplication).

**Files modified:**
- `skills/five-bot/SKILL.md` — added Design Reference section with block schema, zip procedure (locate/normalize/extract/identify), guardrails, and import-method values.

**What was added:**
- **Block schema:** The exact Markdown structure written into `ux.md` when a design is captured (Source, File(s), Import method, Original zip, Local artifact, Captured date, Covers, Note).
- **Zip procedure:** Step-by-step flow: get user's `.zip` → move to `design/<slug>.zip` → extract to `design/<slug>/` (with both PowerShell and unix commands) → identify primary `.dc.html` → write block to `ux.md`.
- **Guardrails:** `ux.md` is canon; design is an aid; never auto-build whole design; capture is UX-only; Dev may read when a ticket cites it but implements only the ticket scope; graceful fallback to `ux.md` if artifact missing.
- **Import methods:** `fallback-zip` (MVP), `fallback-url-only` (URL without zip). `connector` reserved for deferred T15.

**Assumptions:**
- Claude Design's "Download zip instead" bundle contains a self-contained `*.dc.html` + assets, implementable offline.
- Both PowerShell (Windows) and unix tools (unzip/tar on macOS/Linux) are available in the agent's environment.
- Users commit `design/` to git (recommended, not enforced).
- Plain Markdown block needs no validation.
- Cross-platform extract is handled by the agent context picking the appropriate command.

**Known limits:**
- T10 defines the procedure in plain Markdown; T11/T12 implement the triggers that invoke it. Testing the full flow happens at T11/T12 + QA.
- Zip extraction assumes standard .zip format; unusual structures would be addressed per user feedback.
- No automated validation of `.dc.html` structure; assumes Claude Design export is always valid.

**Foundation for T11/T12:**
- T11 (`/ux` command) adds a short trigger referencing this section to invoke the procedure.
- T12 (`/dev` command) adds a trigger to read the linked artifact when a ticket cites Design Reference.

---

## Review Notes

### T10: Skill — Design Reference procedure — QA Review

**Verdict: ✅ APPROVED**

**Acceptance Criteria — all met:**
- ✅ SKILL.md gains "## Design Reference (Claude Design import)" section (lines 83–117)
- ✅ Block schema matches architecture.md spec exactly (lines 89–98 vs architecture.md 152–161)
- ✅ Zip procedure: locate → normalize to `design/<slug>.zip` → extract → identify `.dc.html` (lines 101–108)
- ✅ Both PowerShell `Expand-Archive` (line 105) + Unix `unzip`/`tar` (line 106) provided; agent picks
- ✅ Guardrails stated: canon (ux.md), no auto-build, UX-only capture, no integrations, fallback (lines 110–116)
- ✅ Import-method values noted: `fallback-zip` (MVP), `fallback-url-only`; `connector` reserved for T15
- ✅ Plain Markdown only (Cowork-safe)

**QA findings:**
- **No bugs.** Procedure is clear and implementable.
- **No edge cases missed.** Slug conversion, folder creation (`design/` on-first-use per OQ-8), and missing-artifact fallback are all addressed or deferred correctly to T11/T12.
- **No security concerns.** Local file I/O only; no external integrations per guardrails.
- **DRY principle:** Procedure lives once; T11/T12 will reference it without duplication. ✅

**Test plan:** T10 is documentation. Runtime testing deferred to T11 (capture trigger) and T12 (consume trigger); each will exercise the skill procedure in context. No further QA gates needed until T13/T14.

---

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

## v2.2.0 Backlog — Usage-Aware Status Reporting

> Single ticket: **T1**. Query API hourly usage and display in `/5bot-status` output. Canonical edits in `plugins/5bot/`; sync guidance into `_framework/5bot/`. See `architecture.md` → "v2.2.0".

### T1: Add API usage line to /5bot-status — v2.2.0

**Goal:** Display current hourly API usage percentage + reset time in the `/5bot-status` output, so users can decide whether to continue or pause before the next bot command.

**Acceptance Criteria:**
- [ ] `commands/5bot-status.md` modified: query API hourly usage percentage + reset time (UTC).
- [ ] New **Usage** line added to output, placed after "Active ticket" and before "Last decision".
- [ ] Format: `Usage: API hourly usage: {percent}% (resets at HH:MM UTC)` — neutral tone, no emoji, no urgency.
- [ ] Query mechanism: attempt to retrieve usage from Claude context (prompt-based query: "Based on your context, what is the current API hourly usage percentage? Respond with an integer 0–100 or 'unavailable'.").
- [ ] Reset time computed as: `(current_hour + 1) % 24` in UTC, or ask Claude for it directly.
- [ ] **Silent fallback:** if usage data unavailable (API down, no context, surface doesn't support it), **silently omit the Usage line** — no error, no placeholder, command succeeds either way.
- [ ] Placement: exactly one line after "Active ticket" (grouping it with decision context).
- [ ] No changes to other `/5bot-status` sections; no new state files; no writes.
- [ ] Verified: `/5bot-status` output includes Usage line when data available; omits it gracefully when unavailable.

**Files involved:**
- `C:\AI Bins\Code\plugins\5bot\commands\5bot-status.md` — modified to query + display usage
- `C:\AI Bins\Code\plugins\5bot\skills\five-bot\SKILL.md` — optional note on usage query mechanism (reference, not duplication)

**Out of scope:**
- No pause/resume logic (soft preference only — informational, non-blocking)
- No credential management (use Claude context only, never API keys in project)
- No rate-limit queuing or automation
- No changes to other commands or state files

---

## v2.2.0 Developer Notes

> **STATUS (2026-06-29): T1 REVERTED at the QA gate (verdict REVISE).** The API-usage approach was rejected (the model can't read usage %); the Usage line was removed from `commands/5bot-status.md`. The feature pivots to a knowable signal (recommended: git-awareness) — re-scope via `/product`. See `decisions.md`. Notes below retained for history.

### T1: Add API usage line to /5bot-status — implemented (2026-06-29)

**Summary of changes:** Added an optional, read-only **Usage** line to `commands/5bot-status.md`, placed after "Active ticket" and before "Last decision" per the architecture. Format: `Usage: API hourly usage: {percent}% (resets at HH:MM UTC)`, neutral tone, no emoji. Reset time computed in UTC as the top of the next hour `(current_hour + 1) % 24` → `HH:00 UTC`. Updated the line-count note to "~10 (~12 with Usage)". The command still writes nothing (read-only preserved).

**Files modified:**
- `plugins/5bot/commands/5bot-status.md` — added the Usage bullet + line-count note.

**Dev decision — no-fabrication guard (important):** The ticket/architecture propose asking Claude for its own usage % ("Claude already knows its own usage"). In practice the model does **not** have a reliable signal for the user's API hourly usage %, so the implementation emits the line **only** when a *real* signal is present in context (system hint / env var / MCP source) and otherwise **silently omits** it — and it explicitly forbids guessing/fabricating a number (anti-drift rule 6). Keeps the common case safe (no made-up percentages) and matches the approved "silent fallback." This is a faithful implementation of the AC, not an architecture change.

**Known limits / honest feasibility note:** On most surfaces today there is likely **no** usage signal in context, so the line will usually be omitted (the designed graceful fallback). The feature shows real data only if/where the host surface injects usage info. Maps directly to the recorded post-Dev QA question "Does the usage query work reliably across Code/Cowork/web?" — QA should verify per surface and expect omission to be the norm unless a signal exists. **Open risk for the gate:** the feature may rarely/never display in practice.

**SKILL.md note:** Architecture marked a SKILL.md reference note *optional*. Skipped intentionally — `/5bot-status` is the only consumer, so centralizing adds indirection with no DRY benefit; logic is self-contained in the command. Trivial to add if QA prefers.

**_framework sync:** Not applicable for T1 — it touches only the plugin command; `_framework/5bot/` holds rules/personas/templates, none of which changed.

**How to test (runtime — QA staging):** run `/5bot-status` in (a) a surface where usage info is present (expect the Usage line, correct position + format) and (b) a surface with none (expect the line silently omitted, no error, rest of snapshot intact). Confirm no fabricated number ever appears. Not exercisable in this Markdown-only authoring environment.

---

## v2.2.0 QA Review

### T1: Add API usage line to /5bot-status — QA (2026-06-29)

**Verdict: ✅ APPROVED WITH NOTES** (static review; runtime cross-surface test pending at staging)

| Acceptance criterion | Status | Notes |
|---|---|---|
| Command queries usage % + UTC reset | ✅ | Usage bullet added with query + UTC reset computation |
| Usage line after Active ticket, before Last decision | ✅ | Correct placement |
| Format `Usage: API hourly usage: {percent}% (resets at HH:MM UTC)`, neutral | ✅ | Exact format; no emoji/urgency |
| Prompt-based query from context | ✅ | Hardened to real-signal-only (system hint / env / MCP) |
| Reset = `(current_hour+1)%24` UTC | ✅ | `HH:00 UTC` |
| Silent fallback, no error/placeholder | ✅ | Omits line; no `unavailable` text; command still succeeds |
| No other sections changed; no writes; no new state | ✅ | Read-only preserved |
| Verified output includes/omits Usage line | ⚠️ NOT RUNTIME-TESTED | Markdown plugin; verify at staging across Code/Cowork/web |

**Findings:**
- ✅ Implementation faithfully meets every acceptance criterion; clean, minimal, read-only.
- ✅ **No-fabrication guard is the right call.** Without it, the "ask Claude for usage %" prompt risks a hallucinated number — a real trust/correctness bug. The guard converts the unreliable case into a safe silent omission. QA endorses keeping it.
- 🔴 **Feasibility risk (escalate to human gate — NOT a Dev fix):** the model has no reliable access to the user's API hourly usage %; on standard surfaces no such signal is in context. Net effect: **the Usage line will almost always be omitted** — the feature likely shows nothing in practice. This is inherent to the feature's premise (the Architect rationale "Claude already knows its own usage" is factually incorrect), not an implementation defect, so there is no code change for Dev to make.
- 🟡 **Record correction:** the Architect decision's claim that Claude knows its own usage via "internal rate-limit enforcement" should be marked inaccurate so future readers don't rely on it.
- 🟢 Security/safety: no credentials, no writes, no new deps — none.

**Recommendation to the gate** — the work is correct and safe to ship as-is (a harmless, forward-compatible no-op that activates only if a real signal ever appears). Because it likely never displays today, the human should choose:
- **(a) Ship as-is** — accept as forward-compatible; no harm.
- **(b) Revise** — pivot to a signal the model/harness CAN see (context-window / `/compact` pressure, or the read-only git-awareness idea already in Open Follow-ups), to actually deliver the "can I continue?" value.
- **(c) Reject/shelve** until a usage signal is genuinely exposed to commands.

**Test plan (staging):** install from staging; run `/5bot-status` on (a) a surface with usage info present → Usage line correct; (b) without → omitted, no error, snapshot intact; confirm no fabricated % ever; confirm placement + format.

---

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

## v2.2.0 (re-scoped) — T2: git-awareness line (build record)

### T2 — Ticket
**Goal:** Add a read-only git-awareness line to `/5bot-status` (branch · sync vs upstream · clean/dirty), per `ux.md` E1 (re-scoped) and the `architecture.md` git spec.

**Acceptance Criteria:**
- [ ] `commands/5bot-status.md`: append a git line to the freshness footer when in a git repo; **silent no-op** otherwise (not a repo / git absent / any git error).
- [ ] Uses only read-only, **no-network** commands (`rev-parse`, `symbolic-ref`, `rev-list --left-right --count @{u}...HEAD`, `status --porcelain`); never fetch/pull/push/commit/mutate.
- [ ] Render `git: <branch> · <sync> · <clean|uncommitted changes>`; terse all-good `git: <branch> · in sync · clean`; `(last fetch)` on behind/ahead; `no upstream` / `detached HEAD` handled.
- [ ] Informational only — does not change the recommended next command; read-only (writes nothing).
- [ ] No other `/5bot-status` sections changed.

**Files:** `commands/5bot-status.md`. **Out of scope:** fetch/pull/push/mutation/network; credentials; GitHub/Jira/Slack integration; git state on other commands; `/5bot-init` nudge (deferred).

### T2 — Developer Notes (2026-06-29)
Implemented in `commands/5bot-status.md`: extended the freshness-footer instruction to append the git line when in a repo, and added a "Git line" spec block (the four read-only commands + render + edge handling). No-network / no-mutation and omit-on-error are stated explicitly. No `_framework` sync needed (touches only the plugin command). Cross-platform: the git commands are identical on Windows/unix; only shell invocation differs (handled by the agent).
**Runtime test (staging):** `/5bot-status` in (a) clean+synced repo → `… · in sync · clean`; (b) behind/dirty → `N behind origin (last fetch) · uncommitted changes`; (c) no upstream → `no upstream`; (d) detached HEAD; (e) non-repo → line omitted, no error. Confirm no fetch/network and nothing written.

### T2 — QA Review (2026-06-29)
**Verdict: ✅ APPROVED WITH NOTES** (static; runtime cross-surface test pending at staging).
- ✅ Meets all AC; read-only, no-network, omit-on-error, informational-only — matches `ux.md` / `architecture.md`. Unlike the reverted API-usage T1, **this signal is genuinely knowable** (git is local), so the line will actually display.
- ✅ Honest staleness: `(last fetch)` label on behind/ahead (no fetch performed).
- 🟡 Note: ahead/behind reflects the last fetch — if the user hasn't fetched recently it may lag reality. Acceptable + labeled; flagged for the gate.
- 🟢 Security/privacy: local-only facts; no credentials, no network, no writes.
- ⚠️ Runtime not exercisable here (Markdown plugin) — verify per the staging test plan.
**Recommendation:** APPROVE for release as v2.2.0 (pending the staging smoke test).

## Release Checklist
