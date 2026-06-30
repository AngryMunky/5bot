# Decision Log

<!-- Decisions and human-gate approvals. Newest on top. -->

## Approval: QA Gate — v2.3.0 Lean Context (T1–T4)

- Date / By / Stage: 2026-06-29 / Human / QA Gate
- Status: APPROVED
- Decision: v2.3.0 Lean Context (T1–T4) ACCEPTED. Cleared for release as v2.3.0.
- Notes carried: rollover relies on version-labeled section headers (add a README line); runtime verify at staging, incl. a dogfood `/5bot-archive` on this project.
- Release (T5): README + roster (`/5bot-archive`), bump `plugin.json` + `marketplace.json` 2.2.0→2.3.0, root `CLAUDE.md` + MEMORY, dual-push to both repos, tag `v2.3.0` (per `PUBLISH.md`). **Push to public + `main` requires explicit human authorization** (not implied by this gate).

## Approval: UX Gate — v2.3.0 Lean Context

- Date / By / Stage: 2026-06-29 / Human / UX Gate
- Status: APPROVED
- Decision: Lean Context UX approved — visible surfaces L1–L5 (single sectioned `archive.md`; per-file pointer lines; `/handoff` rollover note [auto but **never silent**]; idempotent `/5bot-archive`; `/5bot-status` archive note). Lazy loading invisible-by-design. Non-destructive throughout.
- UX recs adopted: OQ-6 auto-rollover + announce (no y/n prompt); OQ-7 no `--dry-run` for MVP.
- Architect gate: **skipped** (2-gate default; small, clear feature) — Architect resolves OQ-3 (lazy-read mechanism) inline, then Dev.
- Consequences: Architect spec + tickets → Dev → QA → QA gate → release v2.3.0.

## Approval: Product Gate — v2.3.0 Lean Context (lazy loading + archive rollover)

- Date / By / Stage: 2026-06-29 / Human / Product Gate
- Status: APPROVED
- Decision: v2.3.0 **Lean Context** approved — combine **#3 lazy/section loading** + **#1 archive rollover**. **Idea #2 (per-file budgets + summarize) EXCLUDED** (lossy / high-upkeep). Non-destructive (relocate, never delete), mostly automatic (rollover on `/handoff`); `project-state.md` stays canon.
- OQ-1: RESOLVED — APPROVED. Lazy loading (#3): confirmed.
- Product recommendations for the remaining OQs (carried to UX/Architect; human invited tweaks):
  - **OQ-2 (archive shape):** a **single `archive.md`**, off the default read path, sectioned (Stage history / Decisions / Dev-QA). Lowest file-count + upkeep; size is "free" since it's never loaded unless asked. (Alt: per-kind `decisions-archive.md` / `dev-qa-archive.md` / `history.md`.)
  - **OQ-4 (rollover thresholds):** conservative + deterministic — stage files keep only the *current* version section; `decisions.md` keeps the last ~8 blocks; `dev-qa.md` archives a ticket's full record once it is DONE and its version has shipped. Runs at `/handoff` + a sweep at release.
  - **OQ-5 (migration):** **include a one-time `/5bot-archive` command in MVP** — applies the rollover rules retroactively (idempotent, non-destructive); needed because every existing project (incl. this one) is already bloated; reuses the same logic.
- OQ-3 (lazy-read mechanism — file-split vs anchored/offset reads) → Architect.
- Consequences: proceed to `/ux`.

## Approval: QA Gate — v2.2.0 T2 git-awareness line

- Date / By / Stage: 2026-06-29 / Human / QA Gate
- Status: APPROVED
- Decision: v2.2.0 T2 (read-only git-awareness line in `/5bot-status`) ACCEPTED. Cleared for release as v2.2.0.
- Notes carried: ahead/behind = "as of last fetch" (labeled, accepted); runtime cross-surface smoke test recommended at staging before main.
- Release steps: bump `plugin.json` + `marketplace.json` 2.1.0→2.2.0; update READMEs / root `CLAUDE.md` / MEMORY; sync working copy → publish clone; **dual-push to both repos** (private `5bot-plugin` + public `5bot`); tag `v2.2.0` (per `PUBLISH.md`). **Push to public + `main` requires explicit human authorization** (not implied by this gate).

## Approval: UX Gate — v2.2.0 (re-scoped) git-awareness line

- Date / By / Stage: 2026-06-29 / Human / UX Gate
- Status: APPROVED
- Decision: UX design for the read-only git line in `/5bot-status` approved — placed in the **freshness footer**, format `git: <branch> · <sync, "last fetch"> · <clean|uncommitted changes>`, informational only, silent no-op when not a git repo.
- Detail: **always-show when in a git repo** (consistent trust indicator; avoids the "silence = fine-or-unchecked?" ambiguity), rendered tersely when all-good (`git: <branch> · in sync · clean`) and fuller when noteworthy. Confirmed by the human after asking for a recommendation.
- Architect gate: per the 2-gate default, **skipped** for this small feature (human chose go-straight-to-Dev). OQ-3 (exact no-network commands + edge handling) resolved inline by the Architect.
- Consequences: Architect specs the git commands → Dev implements in `commands/5bot-status.md` → QA → QA gate → release v2.2.0.

## Approval: Product Gate — v2.2.0 (re-scoped) Sync-Awareness via read-only git status

- Date / By / Stage: 2026-06-29 / Human / Product Gate
- Status: APPROVED
- Decision: v2.2.0 re-scoped feature APPROVED — a **read-only git-awareness line** in `/5bot-status` (branch · ahead/behind vs upstream "as of last fetch" · uncommitted-changes flag); silent no-op when not a git repo / no upstream / git absent.
- OQ-1 (scope ruling) RESOLVED: read-only local git-awareness is **acceptable local tooling, NOT** the out-of-scope "External integrations (GitHub/Jira/Slack)." Decider: it never fetches/networks/uses credentials/mutates — it reads local refs + `git status` only (same category as reading on-disk Markdown). The API-usage approach is fully retired.
- Carried: OQ-2 (line content + placement → UX); OQ-3 (cross-platform no-network commands + detached/no-upstream behavior → Architect); OQ-4 (`/5bot-init` nudge → deferred).
- Consequences: proceed to `/ux`. Architect gate optional (2-gate default) — small, well-scoped feature.

## Decision: QA Gate — v2.2.0 T1 → REVISE (pivot off the API-usage signal)

- Date / By / Stage: 2026-06-29 / Human / QA Gate
- Status: REVISE
- Context: T1 implemented the API hourly-usage line in `/5bot-status` and passed all acceptance criteria, but QA escalated a feasibility flaw — the model has no reliable access to the user's API usage %, so the line would almost always silently omit. The feature's premise ("Claude knows its own usage") is incorrect.
- Decision: **REVISE — pivot the v2.2.0 feature off API-usage % to a *knowable* signal.** The API-usage approach is rejected; the implemented Usage line in `commands/5bot-status.md` is **reverted**. The *intent* (a read-only "am I in sync / can I keep working?" awareness line in `/5bot-status`) stands.
- Recommended pivot: **git-awareness** (read-only `git status` / behind-origin line) — genuinely knowable (the command can run git), and it solves the real stale-copy divergence pain hit this session. Context-window-pressure was considered and rejected: same unknowability flaw + overlaps the existing F1 heuristic.
- Open scope question for Product: does a read-only local git line count as the approved out-of-scope "External integrations (GitHub/Jira/Slack)", or is it acceptable local-tooling? (Already flagged in project-state Open Follow-ups.)
- Consequences: re-scope via `/product` (signal choice + the integrations scope question) → UX → Architect (gate optional under the 2-gate default) → Dev → QA → gate → release v2.2.0. The Architect rationale "Claude knows its own usage" is marked inaccurate in the record.

## Approval: Architect Gate — v2.2.0 Usage-Aware Status Reporting

- Date / By / Stage: 2026-06-26 / Human / Architect Gate
- Status: APPROVED
- Decision: v2.2.0 architecture approved. Resolved all three blocking OQs from UX gate: (1) API feasibility → query via Claude context + graceful fallback if unavailable, (2) Reset time format → UTC (HH:MM UTC), (3) Placement in `/5bot-status` → after Active ticket, before Last decision. Ticket T1 defined in dev-qa.md with clear acceptance criteria (query mechanism, format, placement, silent fallback behavior, no scope expansion).
- Architecture quality: Minimal, lean, no new dependencies or state files. Fits existing plugin design cleanly. Low-risk Markdown edit.
- Consequences: Proceed to `/dev T1` to implement the API usage query + display in `/5bot-status`. Then `/qa`, gate, release as v2.2.0. Post-Dev verification questions (surface reliability, timeout behavior, reset time accuracy) are QA scope.


_Older decisions → archive.md § Decisions._
