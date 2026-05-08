# Master Roadmap — `stage-web` to Production

> **Purpose:** Single ordered checklist synthesizing every remaining step across all AIRI plan files. Follow top to bottom. Each step cites its source plan file (in [docs/ai/context/](.)) and the section to consult for detail.
>
> **Built from:**
> - [plan-build-verify-v0.10.2.md](plan-build-verify-v0.10.2.md) (executed → [build-verify-v0.10.2.md](build-verify-v0.10.2.md))
> - [plan-production-readiness.md](plan-production-readiness.md) (Phases 1–5 executed → [v0.10.2-maintenance-report.md](v0.10.2-maintenance-report.md), [v0.10.2-lint-debt-report.md](v0.10.2-lint-debt-report.md))
> - [plan-next-phase-v0.10.3.md](plan-next-phase-v0.10.3.md) (Tracks 1.A, 1.D, 2.E executed → [build-verify-v0.10.3.md](build-verify-v0.10.3.md))
> - [decision-social-module-flagging.md](decision-social-module-flagging.md) (decision recorded; partial implementation)
> - [prd-stage-web-production.md](prd-stage-web-production.md) (proposed; not started)
> - [release-checklist.md](release-checklist.md) (verification protocol)
>
> **Date:** 2026-05-08

---

## Already done — context for the executor

✅ v0.10.2 build & verify (329/329 tests, all builds pass)
✅ v0.10.2 lint debt cleared (33 → 0 errors)
✅ Release checklist codified
✅ CI gap closed: `unit-tests` + `ui-tests` jobs added
✅ v0.10.2 release notes
✅ v0.10.3 ground truth verification (Track 1.A)
✅ Social module feature-flag *implementation* (Track 1.D — code only, README pending)
✅ Mimo `providerModels` typecheck fix (Track 2.E)

Everything below is **not yet done**.

---

# Stage 1 — Close the v0.10.3 release

> **Source:** [plan-next-phase-v0.10.3.md](plan-next-phase-v0.10.3.md) Track 1 + [prd-stage-web-production.md](prd-stage-web-production.md) §2.1
> **Goal:** v0.10.3 tag pushed and deployed. **This is the floor for everything below.**

## 1.1 Resolve build-verify-v0.10.3.md discrepancies

> **Source:** [prd-stage-web-production.md](prd-stage-web-production.md) §2.1

- [x] **1.1a** Re-run `pnpm test:run` and `pnpm test-ui:run` separately, capture **distinct** counts (the report shows identical 49/307 for both — copy-paste). Update [build-verify-v0.10.3.md](build-verify-v0.10.3.md) lines 67-68. **Done when:** the two rows show different file/test counts that match real output.
- [x] **1.1b** Verify [eslint.config.js](../../../eslint.config.js) still ignores `docs/ai/context/ui-structure.md` (it was added per [v0.10.2-lint-debt-report.md](v0.10.2-lint-debt-report.md) §10 but the parsing error reappears in v0.10.3). If the ignore was lost, restore it; if it never took effect, fix the regex. **Done when:** `pnpm lint` reports **0 errors** (132 warnings still acceptable).
- [x] **1.1c** Add `VITE_FEATURE_SOCIAL` documentation to [apps/stage-web/README.md](../../../apps/stage-web/README.md) under a "Feature flags" section, per [decision-social-module-flagging.md](decision-social-module-flagging.md) §Implementation step 4. **Done when:** README documents flag name, default, and how to flip it.

## 1.2 Walk Track 1.B — manual UI checks (web-relevant only)

> **Source:** [plan-next-phase-v0.10.3.md](plan-next-phase-v0.10.3.md) Track 1.B; full procedure in [release-checklist.md](release-checklist.md) §A, §C, §D.
> **Note:** Stage-tamagotchi checks (#5–#7) are out of scope for the web surface but should still be walked before v0.10.3 ships if the project tags both.

- [ ] **1.2a** Check #1 — Reasoning-delta for thinking models (`pnpm dev:web`). Requires real LLM credentials for a thinking-capable model.
- [ ] **1.2b** Check #2 — Xiaomi Mimo speech/transcription (`pnpm dev:web`). Requires Mimo credentials.
- [ ] **1.2c** Check #3 — Character offset slider in landscape (`pnpm dev:web`). No credentials needed.
- [ ] **1.2d** Check #4 — VAD threshold + silence-duration display (`pnpm dev:web`). No credentials needed.
- [ ] **1.2e** Check #8 — Histoire stories render without console errors (`pnpm -F @proj-airi/stage-ui story:dev`). No credentials needed.
- [ ] **1.2f** Check #9 — `airi-screenshot` CLI `--help` + capture against deployed stage-web. No credentials needed.
- [ ] **Done when:** [build-verify-v0.10.3.md](build-verify-v0.10.3.md) Track 1.B table updates each row to ✅ or ❌ with brief notes; any ❌ becomes a blocker issue.

## 1.3 Verify CI gates merges (Track 1.C)

> **Source:** [plan-next-phase-v0.10.3.md](plan-next-phase-v0.10.3.md) Track 1.C

- [ ] **1.3a** Read [.github/workflows/ci.yml](../../../.github/workflows/ci.yml). Confirm `ui-tests` job has `npx playwright install --with-deps chromium` step (the maintenance-report YAML snippet doesn't show one — it likely needs adding).
- [ ] **1.3b** Open a draft PR with one stage-ui test deliberately broken. Confirm `unit-tests` and `ui-tests` jobs both fail; merge is blocked.
- [ ] **1.3c** Close draft, capture proof in PR description, link from [build-verify-v0.10.3.md](build-verify-v0.10.3.md) Track 1.C row.
- [ ] **Done when:** A linked PR demonstrates broken-test → CI red → merge blocked.

## 1.4 Tag and ship v0.10.3

> **Source:** [release-checklist.md](release-checklist.md) §"Post-Release: Documentation"

- [ ] **1.4a** Use the [release-note-writer](../../../.agent/skills/release-note-writer) skill to draft v0.10.3 notes covering changes since v0.10.2.
- [ ] **1.4b** Bump version in `package.json` files (or via release tooling).
- [ ] **1.4c** Tag `v0.10.3`, push tag.
- [ ] **1.4d** Verify the host's auto-deploy (Netlify / Cloudflare Pages) succeeds for the new tag.
- [ ] **Done when:** `v0.10.3` tag exists on `main`, deploys are live, release notes published.

---

# Stage 2 — Quality improvements (target v0.10.3, may slip to v0.10.4)

> **Source:** [plan-next-phase-v0.10.3.md](plan-next-phase-v0.10.3.md) Track 2

## 2.1 Migrate `errorMessageFrom` warnings

> **Source:** Track 2.F. Current count: **132 warnings** ([build-verify-v0.10.3.md](build-verify-v0.10.3.md)).

- [ ] **2.1a** Grep `error instanceof Error \?` across the repo. Bulk-replace with `errorMessageFrom(error)` from `@moeru/std`.
- [ ] **2.1b** Heaviest concentration in `services/twitter-services` and `apps/stage-tamagotchi` per [build-verify-v0.10.3.md](build-verify-v0.10.3.md). Some `services/twitter-services` already migrated in the v0.10.2 round — start with the rest.
- [ ] **2.1c** Single PR: `refactor: migrate to errorMessageFrom helper across services`.
- [ ] **Done when:** `pnpm lint` warnings drop below 30.

## 2.2 Reasoning-delta unit test

> **Source:** Track 2.G

- [ ] **2.2a** Add a test in [packages/stage-ui/src/stores/chat.test.ts](../../../packages/stage-ui/src/stores/chat.test.ts) (or co-located) feeding a mocked LLM response with `reasoning-delta` events.
- [ ] **2.2b** Assert the chat store surfaces them in the expected shape.
- [ ] **Done when:** New test passes; manual check #1 (Stage 1.2a) is now redundant for regressions.

---

# Stage 3 — Production hardening for the web surface

> **Source:** [prd-stage-web-production.md](prd-stage-web-production.md) §2.2 – §2.7

## 3.1 Build hardening (PRD §2.2)

- [ ] **3.1a** Add [`rollup-plugin-visualizer`](https://github.com/btd/rollup-plugin-visualizer) behind a `--analyze` flag in [apps/stage-web/package.json](../../../apps/stage-web/package.json). PRD §2.2.1.
- [ ] **3.1b** Run analyzer; document `dist/` size, top 10 chunks, gzipped totals in [build-verify-v0.10.3.md](build-verify-v0.10.3.md) (or successor).
- [ ] **3.1c** Set bundle-size tripwire (script that fails CI if `dist/` > 200 MB or initial JS chunk > 500 KB gzipped). PRD §1.4.
- [ ] **3.1d** Identify and code-split heavy deps: `@huggingface/transformers`, `@proj-airi/drizzle-duckdb-wasm`, Live2D SDK, fonts. PRD §2.2.1.
- [ ] **3.1e** Configure `build.sourcemap: 'hidden'` in [apps/stage-web/vite.config.ts](../../../apps/stage-web/vite.config.ts). PRD §2.2.2.
- [ ] **3.1f** Add COOP/COEP headers per host:
  - Netlify → [netlify.toml](../../../apps/stage-web/netlify.toml) `[[headers]]` block
  - Cloudflare → create `apps/stage-web/public/_headers`
  - Docker/Nginx → custom nginx.conf in [Dockerfile](../../../apps/stage-web/Dockerfile)
  PRD §2.2.3.
- [ ] **3.1g** Inventory all `import.meta.env.VITE_*` references; produce env-var table in [apps/stage-web/README.md](../../../apps/stage-web/README.md). PRD §2.2.4.
- [ ] **3.1h** Add `VITE_AIRI_SERVER_URL` (or equivalent) for backend URL configurability. PRD §2.2.4.
- [ ] **3.1i** Generate `apps/stage-web/LICENSES.md` via `pnpm licenses list`. Audit for GPL/AGPL. Cite Live2D SDK terms. PRD §2.2.5.
- [ ] **Done when:** All sub-steps checked; budgets enforced in CI (Stage 3.2).

## 3.2 CI/CD pipeline (PRD §2.3)

- [ ] **3.2a** Add `bundle-size` job to [ci.yml](../../../.github/workflows/ci.yml) using the tripwire from 3.1c. PRD §2.3.1.
- [ ] **3.2b** Add `lighthouse-ci` job running against per-PR preview URL (perf ≥ 80, a11y ≥ 90). PRD §2.3.1, §1.4.
- [ ] **3.2c** Verify per-PR preview deploys are wired (Netlify/Cloudflare GitHub apps). Codify in CI status checks. PRD §2.3.4.
- [ ] **3.2d** Document required CI secrets in `.github/SECRETS.md` (sample committed; real secrets in repo settings). PRD §2.3.3.
- [ ] **3.2e** Branch & release strategy doc in [AGENTS.md](../../../AGENTS.md) (or new `docs/ai/context/release-process.md`). PRD §2.3.2.
- [ ] **3.2f** Add nightly `dependency-audit` workflow running `pnpm audit --prod`; opens issue on critical. PRD §2.3.1, §3.1 checklist.
- [ ] **Done when:** PRs show all required gates as status checks; nightly jobs running.

## 3.3 Test gates enforced (PRD §2.4)

- [ ] **3.3a** Codify gate thresholds in CI (lint=0 errors, typecheck=0, tests=100% pass, etc.). PRD §2.4.1.
- [ ] **3.3b** Add Histoire smoke test — assert all stories under `packages/stage-ui/stories/` mount without console errors. PRD §2.4.3.
- [ ] **3.3c** Add i18n parity test — all locale files in [packages/i18n/src/](../../../packages/i18n/src) have the same keys as the canonical `en` locale. PRD §2.4.3, §3.3.
- [ ] **3.3d** Add env-var presence test — fail the build if a required `VITE_*` is unset in production mode. PRD §2.4.3.
- [ ] **Done when:** All three new tests in CI; no required `VITE_*` can ship unset.

## 3.4 Review processes (PRD §2.5)

- [ ] **3.4a** Document PR review policy (1 human approval; AI second-opinion via `review`/`simplify` skills) in [AGENTS.md](../../../AGENTS.md). PRD §2.5.1.
- [ ] **3.4b** Document release review flow (release-checklist + release-note-writer skill). PRD §2.5.2.
- [ ] **3.4c** Document ADR convention (`docs/ai/context/decision-<topic>.md`) and required-decision triggers. PRD §2.5.3.
- [ ] **3.4d** Add accessibility review step (axe in Lighthouse CI + manual keyboard walkthrough). PRD §2.5.4.
- [ ] **Done when:** AGENTS.md covers all four review processes.

## 3.5 Observability & runtime (PRD §2.6)

- [ ] **3.5a** Add Sentry browser SDK (or equivalent) to [apps/stage-web/src/main.ts](../../../apps/stage-web/src/main.ts), gated by `VITE_SENTRY_DSN`. PRD §2.6.1.
- [ ] **3.5b** Wire source-map upload to Sentry on release tag. PRD §2.2.2, §2.6.1.
- [ ] **3.5c** Confirm Plausible/PostHog analytics opt-in defaults match privacy requirements. PRD §2.6.2.
- [ ] **3.5d** Add health-check banner: ping configured backend's `/health`; surface to user if unreachable. PRD §2.6.4.
- [ ] **Done when:** Errors visible in Sentry dashboard; analytics opt-in correct per privacy policy.

## 3.6 Documentation & launch (PRD §2.7)

- [ ] **3.6a** Write `apps/stage-web/docs/DEPLOYING.md` covering each host quickstart, env vars, COOP/COEP, custom domain. PRD §2.7.1.
- [ ] **3.6b** Write `docs/ai/context/runbook-stage-web.md` with failure modes and rollback per host. PRD §2.7.2.
- [ ] **3.6c** Test rollback procedure on a non-release commit; document the test in the runbook. PRD §1.7 success criterion #5.
- [ ] **3.6d** Walk PRD §2.7.4 launch checklist; confirm everything green before tagging the first "production-ready" release (likely **v0.11**).
- [ ] **Done when:** Both docs merged; rollback verified; launch checklist signed.

---

# Stage 4 — Long-term hardening

> **Source:** [plan-next-phase-v0.10.3.md](plan-next-phase-v0.10.3.md) Track 3 + [prd-stage-web-production.md](prd-stage-web-production.md) Part 3

## 4.1 Visual regression in scheduled CI (Track 3.H)

- [ ] **4.1a** Establish a baseline image set for `pnpm test-vishot:run`.
- [ ] **4.1b** Add `.github/workflows/visual-regression.yml` running nightly against `main`.
- [ ] **4.1c** Surface drift as auto-filed issues.
- [ ] **4.1d** Evaluate consolidation with `airi-screenshot` (Phase 6 roadmap item from [plan-production-readiness.md](plan-production-readiness.md)).
- [ ] **Done when:** Nightly runs visible; first drift surfaced as expected.

## 4.2 Re-evaluate social module (Track 3.K)

> **Source:** [decision-social-module-flagging.md](decision-social-module-flagging.md) §"Re-evaluation trigger"

- [ ] **4.2a** During v0.11 planning, decide: ship with real backend, remove, or extend the flag.
- [ ] **4.2b** Update the ADR with the new decision; cross-link follow-up issues.
- [ ] **Done when:** ADR Status moved off "Proposed".

## 4.3 PRD Part 3 — sweep "easy to forget" items

> **Source:** [prd-stage-web-production.md](prd-stage-web-production.md) Part 3 §3.1 – §3.10

Each is a checklist line. Many are completed by Stages 1–3, but a final sweep ensures nothing was missed.

- [ ] **4.3a** Security: CSP, SRI, CORS allow-list on backend, HSTS — many overlap with PRD §2.2.3, §2.6.
- [ ] **4.3b** Accessibility: keyboard, prefers-reduced-motion, screen-reader labels, contrast.
- [ ] **4.3c** Internationalization: Stage 3.3c covers parity; verify RTL support; `Intl` for dates/numbers.
- [ ] **4.3d** Performance budgets: Stage 3.1c covers bundle; verify font-display, image optimization.
- [ ] **4.3e** Privacy & legal: privacy policy, cookie banner (cookieless Plausible already), license bundle, DPA posture.
- [ ] **4.3f** Browser support: matrix in README; polyfill audit; WebGPU/SAB feature detection + fallback UX.
- [ ] **4.3g** PWA: manifest icons, SW caching, "update available" UX, offline behavior.
- [ ] **4.3h** Versioning: semver discipline, embed version in UI (already), deprecation policy.
- [ ] **4.3i** Documentation: per-release README, top-level badges, CHANGELOG, AGENTS.md.
- [ ] **4.3j** Operational: on-call, status page, release cadence, DR plan for backend.
- [ ] **Done when:** Each item has either a check, a "deferred" with a tracking issue, or a "not applicable" with rationale.

---

# Out of scope for this roadmap

- **Track 3.I** (Electron cold-start automated test) and **Track 3.J** (MCP page interaction test) — desktop surface, not web. Track on a separate roadmap if Electron production lifecycle is pursued.
- **`apps/stage-pocket`** — mobile, separate roadmap.
- **`apps/server` / `packages/server-runtime`** — backend, separate roadmap.
- **Net-new product features** — this is a hardening roadmap.

---

# Quick "what to do next" decision tree

```
Are stages 1.1, 1.2, 1.3 all green?
├── No  → Execute Stage 1 in order. Don't start anything else.
└── Yes → Tag v0.10.3 (1.4). Then:
          ├── Want v0.10.3 to also include quality wins? → Stage 2 (parallel-safe).
          └── Or push toward production-ready v0.11? → Stage 3 (sequential within each phase).
                                                          ↓
                                                       Stage 4 once Stage 3 lands.
```

# Estimated effort (rough, ±50%)

| Stage | Estimate |
|---|---|
| 1 | 4–6 hours (manual checks + CI proof PR) |
| 2 | 3–5 hours |
| 3 | 3–5 days (most substantial) |
| 4 | 1–2 days (after Stage 3) |
| **Total** | **~5–8 working days** |

# Dependencies

```
Stage 1 ──► Stage 2  (release before quality polish)
        └─► Stage 3  (Phase 1 of PRD lives in Stage 1)
Stage 3 ──► Stage 4
Stage 2 │ Stage 3   (largely parallel; pick whichever has owner-availability)
```
