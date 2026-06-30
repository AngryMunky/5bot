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

_Superseded version sections → archive.md § Stage history._
