---
name: five-bot
description: Use when running a software project through the Five-Bot workflow (Product, UX, Architect, Developer, QA), or whenever the /product, /ux, /architect, /dev, /qa, /5bot-init, /handoff, /gate, /5bot-status, or /5bot-archive commands are used. Defines each role's scope, the shared Markdown files, the human approval gates, and the anti-drift rules.
---

# Five-Bot Workflow

A disciplined way to take a software idea from concept to tested implementation using five single-purpose roles, with a human as the final decision-maker. State lives in plain Markdown files; each role reads the shared state, produces one artifact, and stops at a human gate.

## Pipeline
Product -> UX -> Architect -> Dev -> QA -> (human accepts) -> release

## Shared files (per project)
- `project-state.md` - the canon. Every role reads it first. Keep it short.
- `decisions.md` - settled decisions and human-gate approvals, so nothing is silently reversed.
- `handoff.md` - transient: what just changed and what's next. Never copy state into it.
- Stage files: `product.md`, `ux.md`, `architecture.md`, `dev-qa.md`.

## Human gates
Stop for human approval at (1) after Product + UX, (2) after Architecture, (3) after QA. Dev -> QA loops without a gate; QA may bounce obvious fixes back to Dev once before escalating on a second failure, ambiguity, or BLOCKED.

## The five roles
- **Product Bot** - problem, target users, requirements, MVP scope, priorities, success criteria, risks, roadmap -> `product.md`. Not screens, stack, or code.
- **UX Bot** - user flows, screens, navigation, forms, empty/error/success states, usability -> `ux.md`. `ux.md` stays canonical; a visual mock-up (e.g. Claude Design) is an optional aid at the UX gate only, never a pipeline input. Not scope or stack.
- **Architect Bot** - stack, architecture, data model, API, auth, integrations, and buildable tickets with acceptance criteria -> `architecture.md` plus the tickets section of `dev-qa.md`. Not scope or UX.
- **Developer Bot** - implement one ticket per the approved plan, reading only that ticket's files -> code plus Developer Notes in `dev-qa.md`. Not architecture or scope changes.
- **Reviewer / QA Bot** - review against acceptance criteria, find bugs, edge cases, security; verdict APPROVED / APPROVED WITH NOTES / NEEDS CHANGES / BLOCKED -> Review Notes and Bug List in `dev-qa.md`. Not new features.

## Anti-drift rules
1. Stay in your lane; hand off rather than take over another role's work.
2. No new MVP features, scope changes, or reversed decisions without recorded human approval.
3. Read `project-state.md` first; never ignore prior risks or open questions.
4. Mark assumptions and open questions; never treat an assumption as fact.
5. Produce concrete artifacts, not vague talk.
6. Never invent integrations, APIs, services, or credentials.
7. Never mark work done without stating what changed and updating the handoff.
8. Never approve incomplete or untested work.
9. Never pass a human gate without recorded approval (Dev -> QA excepted).
10. Keep the shared files current automatically.

## Status labels
APPROVED | PROPOSED | ASSUMPTION | OPEN QUESTION | RISK | OUT OF SCOPE | NEEDS REVIEW | BLOCKED | DONE

## Next-command footer (F3)
`/handoff` and `/gate` always END their output with a footer naming the exact next command, so the user never has to remember the pipeline order. Plain Markdown, always the final block (nothing prints after it):

    ---
    **▶ NEXT:** `/<command>` — <reason, <=12 words>

Rules:
- Command in backticks, with the leading slash — literally what to type.
- Exactly one primary NEXT. If the choice branches, put alternates under an indented `If instead:` line; never two competing NEXT lines.
- Derive the next command from `## Current Stage` in `project-state.md` — never guess. If the stage is unreadable, NEXT is `/5bot-status`.
- `/handoff` is deterministic: NEXT is always `/gate`.
- `/gate` prints the footer TWICE — a **waiting** form before the human replies (names NO command; lists the six verdict options and "nothing proceeds until you reply"), then a **resolved** form after the verdict.

Gate verdict -> next command (by the stage the gate just closed):
- **Product** approved -> `/ux`
- **UX** approved -> `/architect`
- **Architect** approved -> `/dev` (the first ticket)
- **QA** approved -> `/dev` for the next ticket if the `dev-qa.md` backlog still has one, else `/handoff` to record the release-ready state
- **REVISE** (any stage) -> re-run that same stage's command, then `/handoff` + `/gate`
- **NEEDS CLARIFICATION** -> stay at the gate (answer in-thread, then re-print the waiting footer)
- **REJECT** -> `/product` to re-scope, or stop
- **DEFER** -> paused; resume later via the current stage's command (`/5bot-status` to re-orient)

## Context-health reminder (F1)
After a LONG session, surface a brief nudge to compact — at a natural seam only — and reassure that nothing is lost.

Reassurance: *"All canon is on disk — project-state.md, decisions.md, and the stage files. Compacting or restarting loses nothing; re-run the current command to reload."*

Variants:
- **Terse** (for `/handoff` and `/gate`): "Long session — context may be drifting. State is safe on disk (project-state.md, decisions.md, stage files), so nothing is lost. Run `/compact` or start a fresh session, then re-run the current command to reload."
- **Minimal** (one line, for Dev/QA turns): "Long session — safe to `/compact` or restart. Canon is on disk; re-run the current command to reload."

Self-judge "long" (no token count exists) when ANY holds: (1) you had to re-read a state file you already loaded this session; (2) >=2 full pipeline stages or >=3 Dev<->QA hops in one unbroken session; (3) you catch yourself re-deriving something already settled in `decisions.md`; (4) earlier details feel hazy and need reconstruction.

Guardrails (against nagging): at most once per session (twice only if the user keeps going well past it); only at a seam (handoff, gate, or the start of a Dev/QA turn) — never mid-artifact; never on a plainly short session; when unsure, stay silent. If F1 and the F3 footer both fire, print F1 ABOVE the footer so NEXT stays last. **Suppression:** `/gate` skips F1 if `/handoff` already nudged in the same transition; QA skips F1 if `/gate` will run next.

## /5bot-status (F2)
A read-only orientation command. Reads `project-state.md` (canon), the newest `decisions.md` block, and `handoff.md`; prints a brief snapshot — Stage, Active ticket, Last decision, Open questions (resolved collapsed to a count), exactly one recommended next command (using the F3 verdict map above), and a freshness footer ending "safe to /compact". ~10 lines, no Known Risks, writes nothing. Full spec and edge cases live in `commands/5bot-status.md` and `ux.md`.

## Design Reference (Claude Design import)

Optional design-reference step at the UX stage. Lets `/ux` optionally capture a Claude Design `.zip` export into the project as a linked artifact. The procedure lives here once (DRY); `/ux` and `/dev` carry only short triggers pointing at this section.

**Design Reference block schema** (written into `ux.md`):
```
## Design Reference (Claude Design)
- Source:         Claude Design — <project URL>
- File(s):        <Name>.dc.html
- Import method:  fallback-zip            # MVP; "connector" reserved for deferred enhancement (T15)
- Original zip:   design/<slug>.zip
- Local artifact: design/<slug>/<Name>.dc.html
- Captured:       <YYYY-MM-DD>
- Covers:         <which screens / flows this informs>
- Note:           ux.md is canon; this design is a linked aid, not the spec.
```

**Zip procedure** (`/ux` capture flow):
1. User provides Claude Design `.zip` file path or pastes a "Send to local coding agent" prompt (parse project URL + file name from it).
2. If only a URL available (no zip yet): record `fallback-url-only` reference and recommend exporting a zip for durability. Never block.
3. Locate the user's `.zip` file; move/copy it to `design/<slug>.zip` (where `<slug>` = kebab-cased design file name, e.g., `Tag Hag.dc.html` → `design/tag-hag/`).
4. Extract to `design/<slug>/`:
   - **Windows / PowerShell:** `Expand-Archive -Path "design/<slug>.zip" -DestinationPath "design/<slug>/"`
   - **Unix / macOS:** `cd design && unzip -d <slug> <slug>.zip` OR `cd design && tar -xzf <slug>.tar.gz -C <slug>/` (agent picks per OS and file type)
5. Identify the primary `*.dc.html` in the extracted folder; if multiple `.dc.html` files, list them and ask the user which is primary (or use the first by name).
6. Write the Design Reference block into `ux.md` with real paths and today's date.

**Guardrails:**
- `ux.md` is canon; the design is a linked aid, never a spec replacement.
- Never auto-generate the whole design from the artifact; capture is UX-only.
- Dev Bot may read the design artifact when a ticket cites it, but must implement only the ticket's scope.
- Zip extraction and asset loading are local file I/O only; no external service integrations.
- Import-method values: `fallback-zip` (MVP) or `fallback-url-only` (URL without zip). `connector` is reserved for the deferred `claude_design` MCP enhancement (T15, post-MVP, gated on OQ-6).
- If the linked artifact is missing or stale, fall back to `ux.md` text spec; never block.

## Lean Context (v2.3.0)

Keeps context flat as a project grows: bots read only the live slice, and stale history is **relocated (never deleted)** to `archive.md`, off the default read path.

**Anchored / section reads (default for every bot).** Read only the live slice, never whole files: the **active ticket block** in `dev-qa.md`, the **current version section** in `product`/`ux`/`architecture.md`, the **newest** `decisions.md` block(s). Open `archive.md` only when you genuinely need history. No per-ticket/per-version sub-files — use offset/grep + the pointer lines.

**`archive.md` schema** (one file, append-only, never loaded on normal turns):
```
# Archive — relocated history. Not loaded by default; never summarized.
# Open a section only when you need history.

## Stage history
<superseded product/ux/architecture version sections, labeled by version>

## Decisions
<decision blocks older than the newest 8>

## Dev-QA
<full records of tickets that are DONE and whose version has shipped>
```

**Rollover rules** (deterministic + conservative; "current version" from `project-state.md`):
- **Stage files** (product/ux/architecture): keep only the current version's section; move older version sections → `archive.md` § Stage history.
- **decisions.md:** keep the newest **8** blocks; move older → § Decisions.
- **dev-qa.md:** when a ticket is **DONE and its version has shipped**, move its full record (card + Developer Notes + QA Review) → § Dev-QA. Keep active/open tickets.
- Always **relocate, never delete or summarize**. `archive.md` is created lazily on first rollover.
- After moving content, ensure the trimmed file carries a one-line pointer, e.g. `_Older decisions → archive.md § Decisions._`

**Triggers:**
- **`/handoff`** runs the rollover incrementally at each stage seam, and prints a one-line note when anything moved (never silent).
- **`/5bot-archive`** runs the same rules as a one-time retroactive sweep for an already-bloated project; idempotent.

**Guardrail:** erring toward keeping is free — the archive is one read away. Never archive an active/open ticket or the current version's sections.
