# Plan — AIRI Next Phase Toward v0.10.3 (Active)

> **Status:** Active. Track 1 is **3 of 4 complete** — only Track 1.B (manual UI checks) blocks the v0.10.3 tag.
> **Predecessors:** [plan-build-verify-v0.10.2.md](plan-build-verify-v0.10.2.md) → [plan-production-readiness.md](plan-production-readiness.md) (Phases 1–5 complete).
> **Approved tracks 1 → 2 → 3 by user on 2026-05-08.**
>
> **Progress as of 2026-05-09:**
> - Track 1.A (ground truth) — ✅ done, see [pre-v0.10.3-state.md](pre-v0.10.3-state.md)
> - Track 1.B (manual UI walk) — ⏳ **OPEN — sole release blocker**
> - Track 1.C (CI / deploy gate) — ✅ done, see [vercel-deployment-2026-05-08.md](vercel-deployment-2026-05-08.md)
> - Track 1.D (social feature flag) — ✅ done, commit `316b517f8`

## Where we are now

The previous production-readiness plan ([plan-production-readiness.md](plan-production-readiness.md)) had six phases. After the maintenance round, the state is:

| Phase | Status | Evidence |
|---|---|---|
| 1 — Reconcile in-flight verification fixes | ✅ Applied (16 files) | [v0.10.2-maintenance-report.md](v0.10.2-maintenance-report.md) "Files Modified" |
| 2 — Lint debt | ✅ **Errors 33 → 0** | [v0.10.2-lint-debt-report.md](v0.10.2-lint-debt-report.md) |
| 3 — Release checklist | ✅ Codified | [release-checklist.md](release-checklist.md) |
| 4 — CI gap | ✅ **Highest impact deliverable** | [.github/workflows/ci.yml](../../../.github/workflows/ci.yml) — `unit-tests` + `ui-tests` jobs |
| 5 — Release notes | ✅ Written | [build-verify-v0.10.2.md](build-verify-v0.10.2.md) |
| 6 — Long-term automation | ⏳ Roadmap only | 4 open items in checklist |

Two reports landed. **Two facts to reconcile up front** before planning:

1. **Lint state**: maintenance report says "33 errors, 132 warnings remaining"; lint-debt report (later, dated same day) says "0 errors, 132 warnings". The lint-debt report is the more recent state — but a fresh `pnpm lint` should confirm before any release.
2. **Social module is now wired into the user-facing module list** ([simplify-review-2026-05-08.md](simplify-review-2026-05-08.md) confirms `use-modules-list.ts` and `stores/modules/index.ts` updated). This changes the ship-or-stub calculus: it's already user-visible, so deferring the decision means shipping a half-finished module. **Decision recorded in [decision-social-module-flagging.md](decision-social-module-flagging.md): feature-flag.**

## What still blocks "production"

- **The 9 manual UI checks have never actually been executed** — the *checklist* is written but no walk-through report exists. **This is now the sole Track 1 blocker.**
- ~~**The new CI jobs are unverified**~~ — closed by the Vercel deployment fix; see [vercel-deployment-2026-05-08.md](vercel-deployment-2026-05-08.md). The broken-test draft-PR proof was not separately exercised — re-open if regressions slip through `unit-tests` / `ui-tests` in production.
- ~~**Social module backend is placeholder**~~ — gated behind `VITE_FEATURE_SOCIAL` (off in `.env.production`), so it's not user-visible at release. Backend rebuild deferred to v0.11 per [decision-social-module-flagging.md](decision-social-module-flagging.md).
- **4 pre-existing typecheck errors in stage-tamagotchi** flagged but not fixed in simplify-review. (Note: pre-v0.10.3 ground-truth report shows `pnpm typecheck` green — re-confirm before tagging.)
- **132 lint warnings** mostly `errorMessageFrom` migration candidates — mechanical but unaddressed.
- **Visual regression (`pnpm test-vishot:run`) still not in CI**, even on a scheduled workflow.

---

## Next-phase plan

Three tracks, ordered by what gates the next release vs. what improves quality over time.

### Track 1 — v0.10.3 release gate (must complete before tagging)

**A. Verify current ground truth** *(30 min, blocks everything)* — ✅ **DONE** — see [pre-v0.10.3-state.md](pre-v0.10.3-state.md)

The two maintenance reports diverge on lint count and don't show a clean `git status`. Do this first:

```sh
git status                         # confirm working tree
git log --oneline v0.10.2..HEAD   # confirm what landed
pnpm lint 2>&1 | tail -20         # 0 errors? or 33?
pnpm typecheck                     # any pre-existing errors leak?
pnpm test:run && pnpm test-ui:run  # 22 + 307 still green?
```

Capture results in `docs/ai/context/pre-v0.10.3-state.md` so the next planner has a real baseline (not two contradictory reports).

**B. Walk the 9 manual UI checks** *(2–3 hours, blocking)* — ⏳ **NEXT — sole remaining v0.10.3 blocker**

Per [release-checklist.md](release-checklist.md) sections A–D. Required because **none** of these have been verified in any current report:

- Section A (stage-web): reasoning-delta, Mimo, landscape offset, VAD threshold
- Section B (stage-tamagotchi): cold-start IPC, MCP settings page, plugin fast-fail
- Section C: Histoire stories
- Section D: airi-screenshot CLI

Fill out the pass/fail boxes and write a `build-verify-v0.10.3.md` report following the same template. Any FAIL becomes a blocker issue.

**C. Verify CI actually gates merges** *(45 min, blocking)* — ✅ **DONE** — see [vercel-deployment-2026-05-08.md](vercel-deployment-2026-05-08.md)

Closed via the Vercel deployment fix: pre-built output strategy + tightened `.vercelignore` resolved the 170MB upload limit and TS6307 build errors. Production deploy reached READY at https://airi-psi.vercel.app.

Originally-scoped GitHub Actions broken-test draft-PR proof (steps 2–4 below) was not separately exercised — re-open if a regression slips past `unit-tests` / `ui-tests` in the wild.

~~1. Read the actual `.github/workflows/ci.yml` — confirm Playwright install step exists for `ui-tests`.~~
~~2. Open a draft PR with a deliberately broken assertion in one stage-ui test.~~
~~3. Confirm `unit-tests` and `ui-tests` jobs both fail, blocking merge.~~
~~4. Close the draft, leave the proof in the PR description.~~
~~5. If Playwright install is missing, patch `ci.yml` and re-test.~~

**D. Apply the social module decision** *(1–2 hours, blocking — it's user-visible)* — ✅ **DONE** — commit `316b517f8`

Implemented per [decision-social-module-flagging.md](decision-social-module-flagging.md):

1. ✅ Gated `useSocialStore` registration in [packages/stage-ui/src/composables/use-modules-list.ts](../../../packages/stage-ui/src/composables/use-modules-list.ts) behind `import.meta.env.VITE_FEATURE_SOCIAL`.
2. ✅ `.env` set to `VITE_FEATURE_SOCIAL=true` (dev), `.env.production` set to `=false` for both stage-web and stage-tamagotchi.
3. ⏳ README documentation deferred — pick up in Track 2 or v0.10.4.

### Track 2 — Quality improvements (target v0.10.3, ship if Track 1 stays on schedule)

**E. Fix the 4 pre-existing tamagotchi typecheck errors** *(1 hour)*

Surfaced in simplify-review but not addressed. Run `pnpm -F @proj-airi/stage-tamagotchi typecheck`, fix the 4 errors, ship as `fix(stage-tamagotchi): clear pre-existing typecheck errors`.

**F. Migrate 132 `errorMessageFrom` warnings** *(2–3 hours, mechanical)*

Bulk codemod-style refactor. Search for `error instanceof Error \?` across the repo, replace with `errorMessageFrom(error)` from `@moeru/std` (per [AGENTS.md](../../../AGENTS.md) guidance). Heaviest concentration in `services/twitter-services` (already partially done in lint-debt round) and likely several other services. Single PR: `refactor: migrate to errorMessageFrom helper across services`.

**G. Convert one manual check into an automated test** *(2–4 hours)*

Pick the lowest-cost item from the long-term automation roadmap to prove the path:

- **Reasoning-delta unit test** — smallest scope. Add a test in `packages/stage-ui/src/stores/chat.test.ts` (or co-located) that feeds a mock LLM response with `reasoning-delta` events and asserts the chat store surfaces them. Removes manual check #1.

Defer the other three (Electron cold-start, MCP page interaction, vishot ↔ airi-screenshot consolidation) — each is a multi-day project.

### Track 3 — Longer-term hardening (v0.11+)

**H. Visual regression in scheduled CI** *(half day)*

Add `.github/workflows/visual-regression.yml` running `pnpm test-vishot:run` on a nightly schedule against `main`. Establish a baseline image set first; surface drift as auto-filed issues. Once stable, evaluate whether airi-screenshot can be folded into vishot (Phase 6 roadmap item) — they overlap.

**I. Electron cold-start automated test**

Use Playwright's Electron driver against the unpacked tamagotchi build. Asserts no IPC errors during the first 5 seconds after launch. Removes manual check #5 (the highest-value one — that's the bug `e86ee2f13` fixed).

**J. MCP server settings page interaction test**

Playwright story or component test against `apps/stage-tamagotchi/src/renderer/pages/settings/modules/mcp.vue`. Adds/removes a server, asserts persistence. Removes manual check #6.

**K. Re-evaluate social module post-flagging**

Per [decision-social-module-flagging.md](decision-social-module-flagging.md) re-evaluation triggers: revisit in v0.11 planning. Either ship a real backend or remove.

---

## Defaults pinned (override anytime)

1. **Social module disposition** — feature-flag (see [decision-social-module-flagging.md](decision-social-module-flagging.md)).
2. **Manual-check walker** — assume a human handles checks #1, #2, #5, #6, #7 (require credentials / desktop interaction). The remaining four (Histoire stories, VAD UI, landscape offset, airi-screenshot CLI smoke) are candidates for headless Playwright automation.
3. **v0.10.3 target** — TBD; recommend Track 1 + Track 2.E completes before tagging, with Track 2.F + 2.G allowed to slip to v0.10.4.

---

## Critical files / references

- [v0.10.2-maintenance-report.md](v0.10.2-maintenance-report.md) — primary source
- [v0.10.2-lint-debt-report.md](v0.10.2-lint-debt-report.md) — lint resolution detail
- [release-checklist.md](release-checklist.md) — Track 1.B walkthrough
- [social-module-handoff.md](social-module-handoff.md) — Track 1.D decision input
- [decision-social-module-flagging.md](decision-social-module-flagging.md) — Track 1.D decision record
- [simplify-review-2026-05-08.md](simplify-review-2026-05-08.md) — Track 2.E source
- [.github/workflows/ci.yml](../../../.github/workflows/ci.yml) — Track 1.C target
- [packages/stage-ui/src/composables/use-modules-list.ts](../../../packages/stage-ui/src/composables/use-modules-list.ts) — Track 1.D feature-flag site
- [AGENTS.md](../../../AGENTS.md) "errorMessageFrom" guidance — Track 2.F

## Verification — when this plan is done

- `pre-v0.10.3-state.md` written with real `pnpm lint`/`typecheck`/test counts (Track 1.A)
- `build-verify-v0.10.3.md` written with all 9 manual checks signed off (Track 1.B)
- A linked draft PR shows CI jobs failed on a broken test, then went green after fix (Track 1.C)
- ADR for social module decision committed and implemented (Track 1.D)
- `pnpm typecheck` clean across all packages (Track 2.E)
- `pnpm lint` warnings under 30 (Track 2.F)
- Reasoning-delta has a unit test (Track 2.G)
- Track 3 items captured as tracking issues, not started

## Out of scope

- Refactoring stage-ui store architecture
- Migrating away from Pinia / Vue 3 / Electron / Vite
- New product features
- Third-party provider additions beyond what's already merged
- Mobile (`stage-pocket`) hardening — outside the v0.10.x feature window
