# PRD & Plan — `stage-web` Production Readiness

> **Scope:** Web surface only. This document covers the production lifecycle of [`apps/stage-web`](../../../apps/stage-web), the unified web bundle that ships stage-web SPA + VitePress docs + Histoire UI catalog as a single static site. Electron (`stage-tamagotchi`), mobile (`stage-pocket`), and the streaming backend (`apps/server`, `packages/server-runtime`) are explicitly out of scope here; they have their own production stories.
>
> **Predecessors:** [plan-build-verify-v0.10.2.md](plan-build-verify-v0.10.2.md), [plan-production-readiness.md](plan-production-readiness.md), [plan-next-phase-v0.10.3.md](plan-next-phase-v0.10.3.md), [build-verify-v0.10.3.md](build-verify-v0.10.3.md), [decision-social-module-flagging.md](decision-social-module-flagging.md).
>
> **Status:** Proposed. Date: 2026-05-08.

---

# Part 1 — Product Requirements (PRD)

## 1.1 Goals

1. Ship a production-grade build of `stage-web` that runs on any static-asset host (Netlify, Cloudflare Pages, Vercel, S3+CloudFront, self-hosted Nginx via the existing [Dockerfile](../../../apps/stage-web/Dockerfile)).
2. Ensure each release is **gated by automated checks** (no manual-only releases) and **observable in production** (errors, performance, opt-in analytics).
3. Establish a repeatable **release cadence** with verification, deployment, monitoring, and rollback steps that any contributor can execute from `release-checklist.md`.

## 1.2 Target users

| User | Need |
|---|---|
| Self-hosters | Clone, configure env vars, run `docker build` (or static deploy), point at their AIRI server backend |
| Project-hosted users (https://airi.moeru.ai) | Working public web app at the canonical URL with reliable backend connectivity |
| Contributors | Preview deploys per PR; reproducible local dev |
| AI agents / automation | Stable URLs for `airi-screenshot`, vishot baselines, and external integrations |

## 1.3 Functional requirements

- **Bundles three sites into one origin:**
  - `/` → stage-web SPA (Vue 3 + Vite)
  - `/docs/` → VitePress documentation
  - `/ui/` → Histoire component catalog
  Build pipeline already established in [`netlify.toml`](../../../apps/stage-web/netlify.toml). Production must preserve this.
- **Configurable backend** — frontend talks to the AIRI server through [`@proj-airi/server-sdk`](../../../packages/server-sdk). Server URL must be env-var configurable per deploy.
- **Feature flags honored in production builds** — `VITE_FEATURE_SOCIAL=false` for production per [decision-social-module-flagging.md](decision-social-module-flagging.md).
- **All v0.10.x features functional:**
  - Reasoning-delta streaming (chat)
  - Xiaomi Mimo speech/transcription
  - ARK chat providers
  - VAD silence-duration tuning
  - Character offset slider in landscape
- **PWA installable** — Vite PWA plugin is already configured in [`apps/stage-web/vite.config.ts`](../../../apps/stage-web/vite.config.ts); production must produce a valid manifest + service worker.
- **Live2D + Three.js characters render** — heavy WASM/WebGL stack must work in production builds.
- **HuggingFace Transformers in-browser** — requires `SharedArrayBuffer` → cross-origin isolation headers (COOP/COEP) in production responses.
- **DuckDB-WASM** persistent storage — same SAB requirement.
- **i18n routing** — VueI18n configured; current translations in [`packages/i18n`](../../../packages/i18n).

## 1.4 Non-functional requirements

| Dimension | Target |
|---|---|
| **Bundle size budget** | Initial JS chunk < 500 KB gzipped; total `dist/` < 200 MB (fits all common host limits including Vercel Pro 250 MB). Validate with [`rollup-plugin-visualizer`](https://github.com/btd/rollup-plugin-visualizer) per release. |
| **TTI (broadband)** | < 5 s on Chrome, fast 3G < 15 s. Measure with Lighthouse CI. |
| **Browser support** | Latest 2 versions of Chrome, Edge, Firefox, Safari. WebGPU optional (gracefully degrade to WebGL/WASM). |
| **Accessibility** | Lighthouse a11y ≥ 90; keyboard navigation works for primary chat flow. |
| **Performance budget** | Lighthouse perf ≥ 80 on `/` (cold load). |
| **Uptime (project-hosted)** | 99% — hard SLA not committed to self-hosters. |
| **Test coverage gate** | 100% of currently-passing tests must continue to pass. New features require at least one targeted test. |

## 1.5 Constraints

- **Static-only frontend.** No Node runtime in the deployed bundle. Any runtime API must go through the configured backend.
- **Cross-origin isolation required.** Hosts that do not allow setting COOP/COEP response headers cannot host the full app (most modern hosts can; Vercel/Netlify/Cloudflare/Nginx all support this).
- **HTTPS required** for getUserMedia (mic), WebGPU, and SAB-related features.
- **Monorepo build context.** Build commands must run from repo root; `pnpm -F @proj-airi/stage-web build` is the correct invocation. Hosts that auto-detect must be configured for the workspace layout.
- **Asset CDN coupling.** [`@proj-airi/vite-plugin-warpdrive`](../../../packages/vite-plugin-warpdrive) downloads heavy static assets (Live2D SDK, models) at build time. Build runners need network access and cache space.
- **Backend deployed separately** and must be reachable from the production origin (configure CORS).

## 1.6 Out of scope

- Hosting choice — platform-agnostic; all current configs (`netlify.toml`, `wrangler.toml`, `Dockerfile`) stay supported. Adding a new host (e.g. `vercel.json`) is a future task, not a requirement.
- Marketing site / landing page beyond what the SPA already provides.
- Authentication & user accounts — `stage-web` is a single-tenant client; multi-user support is a backend concern.
- The streaming backend itself — separate production track.
- Mobile and Electron apps.

## 1.7 Success criteria

A release is "production-ready" when, in this order:

1. All gates in **Part 2 — Plan, Phase 4** pass on the release commit.
2. The **Release Checklist** ([release-checklist.md](release-checklist.md)) is fully signed off.
3. A staging deploy passes Lighthouse and bundle-size budgets.
4. The release has been live on a preview URL for ≥ 24 hours with no Sentry / error-tracking spike.
5. Rollback procedure (Phase 7) has been tested at least once on a non-release commit.

---

# Part 2 — Plan

## 2.0 Status snapshot — what's already in place

| Area | State |
|---|---|
| Build configs for static hosts | ✅ [`netlify.toml`](../../../apps/stage-web/netlify.toml), [`wrangler.toml`](../../../apps/stage-web/wrangler.toml), [`Dockerfile`](../../../apps/stage-web/Dockerfile) |
| Triple-site assembly (SPA + docs + Histoire) | ✅ Wired in `netlify.toml` and `Dockerfile` |
| Feature flag for social module | ✅ `VITE_FEATURE_SOCIAL` (per [decision-social-module-flagging.md](decision-social-module-flagging.md)) |
| Lint + typecheck + matrix builds in CI | ✅ [.github/workflows/ci.yml](../../../.github/workflows/ci.yml) |
| Unit + UI test jobs in CI | ✅ Closed in v0.10.2 maintenance round |
| PWA plugin | ✅ Configured in `vite.config.ts` |
| Analytics | ⚠️ PostHog gated behind `VITE_ENABLE_POSTHOG`; Plausible proxied via Netlify redirects |
| Release checklist | ✅ [release-checklist.md](release-checklist.md) |
| ADR pattern | ✅ [decision-social-module-flagging.md](decision-social-module-flagging.md) |
| Visual regression in CI | ❌ Not wired (deferred to scheduled per `plan-next-phase-v0.10.3.md` Track 3.H) |
| Bundle-size monitoring | ❌ Not in place |
| Lighthouse / perf budgets | ❌ Not in place |
| Error tracking (Sentry/etc.) | ❌ Not in place |
| Deploy preview per PR | ⚠️ Netlify likely auto-previews; not codified |
| Rollback runbook | ❌ Not in place |

## 2.1 Phase 1 — Resolve outstanding v0.10.3 release gates *(blocks production sign-off)*

Two open items from [build-verify-v0.10.3.md](build-verify-v0.10.3.md):

- **Track 1.B — manual UI checks** (`pnpm dev:web`): execute checks #1 (reasoning-delta), #2 (Mimo), #3 (landscape offset), #4 (VAD), and #8 (Histoire). Track 1.B's stage-tamagotchi-only checks (#5–#7) are out of scope for this PRD.
- **Track 1.C — CI gate proof:** open a synthetic broken-test PR and confirm `unit-tests` + `ui-tests` jobs both fail.

Also resolve three discrepancies in the v0.10.3 report:

1. **Test counts copy-paste:** report shows `49 files / 307 tests` for both `test:run` and `test-ui:run`. Re-run; populate real numbers.
2. **Lint regression:** `ui-structure.md:615` parsing error was added to ESLint ignores in [v0.10.2-lint-debt-report.md](v0.10.2-lint-debt-report.md) but reappears in v0.10.3. Verify [eslint.config.js](../../../eslint.config.js) ignore is present and effective.
3. **Track 1.D partial:** README docs for `VITE_FEATURE_SOCIAL` still pending. Add to [`apps/stage-web/README.md`](../../../apps/stage-web/README.md).

## 2.2 Phase 2 — Build hardening

### 2.2.1 Bundle analysis

- Add [`rollup-plugin-visualizer`](https://github.com/btd/rollup-plugin-visualizer) as dev dep in `apps/stage-web/package.json`. Wire it behind a `--analyze` build flag.
- Run on first production build; capture `dist/` size, largest chunks, total gzipped.
- Set budget tripwire: fail CI if `dist/` exceeds 200 MB or initial JS chunk exceeds 500 KB gzipped.
- Identify candidates for code-splitting / lazy loading: `@huggingface/transformers`, `@proj-airi/drizzle-duckdb-wasm`, Live2D SDK, fonts.

### 2.2.2 Source maps

- Generate hidden source maps in production (`build.sourcemap: 'hidden'`) — keeps stack traces useful for Sentry without exposing source to public.
- Upload source maps to error-tracker (Phase 6) via release-time hook; do not include them in the public deploy.

### 2.2.3 Cross-origin isolation headers

Production responses must include:
```
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: require-corp
```
Per host:
- **Netlify** — add to `netlify.toml` `[[headers]]` block.
- **Cloudflare Pages** — `_headers` file at `apps/stage-web/public/_headers`.
- **Nginx (Dockerfile)** — add `add_header` directives in a custom nginx config; replace bare `nginx:stable-alpine` `CMD`.
- **(Future) Vercel** — `vercel.json` `headers` array.

### 2.2.4 Env var inventory

Document every `VITE_*` and build-time env var in `apps/stage-web/README.md`:
| Variable | Purpose | Default | Required? |
|---|---|---|---|
| `VITE_FEATURE_SOCIAL` | Social module on/off | `false` (prod) | No |
| `VITE_ENABLE_POSTHOG` | PostHog analytics | `false` | No |
| `STAGE_WEB_ENABLE_MKCERT` | Local HTTPS dev | `false` | No |
| `VITE_AIRI_SERVER_URL` (proposed) | Backend base URL | `http://localhost:6121` (dev) | Yes for prod |
| `VITE_SENTRY_DSN` (proposed Phase 6) | Error tracking | unset | No |
| ... | (audit and add) | | |

Action: grep all `import.meta.env.VITE_*` references across `apps/stage-web` and the packages it imports; produce a complete table.

### 2.2.5 Privacy & licensing

- License audit: `pnpm licenses list` → produce `apps/stage-web/LICENSES.md`. Flag GPL/AGPL deps if any (project is MIT).
- Bundled fonts (`@proj-airi/font-chillroundm`, `@proj-airi/font-cjkfonts-allseto`, `@proj-airi/font-xiaolai`) — verify license headers in their package READMEs.
- Live2D SDK — usage terms; the asset is downloaded at build time, but redistribution rules apply. Cite in deploy docs.

## 2.3 Phase 3 — CI/CD pipeline

### 2.3.1 Pipeline shape

```
PR opened ──┬─► lint
            ├─► typecheck
            ├─► unit-tests          (added v0.10.2)
            ├─► ui-tests (Playwright) (added v0.10.2)
            ├─► matrix builds       (existing)
            ├─► bundle-size budget  (NEW — Phase 2.2.1)
            ├─► lighthouse-ci       (NEW — Phase 2.4)
            └─► preview-deploy      (NEW — per-PR preview URL)

Merge to main ──┬─► all of the above
                └─► production-deploy (gated on tagged release)

Nightly ───┬─► visual-regression (vishot, NEW — Track 3.H)
           └─► dependency-audit (pnpm audit --prod)
```

### 2.3.2 Branch & release strategy

- `main` is the development branch.
- Tagged releases (`v0.10.3`, `v0.11.0`, …) trigger production deploy.
- Hotfix flow: branch from tag, fix, cut patch tag, deploy.
- All deploys go through the same CI; production differs only in the `production-deploy` job.

### 2.3.3 Secrets management

Document required CI secrets in `.github/SECRETS.md` (gitignored sample committed):
- `NETLIFY_AUTH_TOKEN` / `CLOUDFLARE_API_TOKEN` / equivalent
- `SENTRY_AUTH_TOKEN` (Phase 6)
- `LIGHTHOUSE_CI_TOKEN` (if hosted)
- `POSTHOG_PROJECT_API_KEY` (build-time injection if used at build)

### 2.3.4 Preview deploys per PR

- Netlify and Cloudflare Pages do this automatically when wired to GitHub.
- Verify the existing wiring; if absent, enable in host settings.
- Post the preview URL as a PR comment via the host's GitHub app.

### 2.3.5 Rollback

- Production deploy must be **immutable per release** — every release tagged at the host level (Netlify/CFP/Vercel all support instant rollback to a prior deploy).
- Rollback runbook: see Phase 7.

## 2.4 Phase 4 — Test suite & quality gates

### 2.4.1 Required gates (block merge)

| Gate | Command | Threshold |
|---|---|---|
| Lint | `pnpm lint` | 0 errors |
| Typecheck | `pnpm typecheck` | 0 errors all packages |
| Unit tests | `pnpm test:run` | 100% pass |
| UI tests (Playwright) | `pnpm test-ui:run` | 100% pass |
| Build (matrix) | `pnpm build` | Success on linux/mac/win runners |
| Bundle size | rollup-plugin-visualizer + tripwire | < budgets in 1.4 |
| Lighthouse perf | `lhci autorun` against preview | perf ≥ 80, a11y ≥ 90 |

### 2.4.2 Recommended gates (warn, not block)

- **Visual regression** (`pnpm test-vishot:run`) — nightly schedule per Track 3.H. Drift opens an issue.
- **Dependency audit** (`pnpm audit --prod`) — nightly. High/critical opens an issue.
- **Targeted feature tests** (per [release-checklist.md](release-checklist.md) Step 6) — explicit run before tagging.

### 2.4.3 New tests needed for production

- **Reasoning-delta unit test** (Track 2.G) — covers manual check #1.
- **Histoire smoke test** — assert all stories under `packages/stage-ui/stories/` mount without console errors.
- **i18n coverage test** — assert all locale files have the same keys as the canonical `en` locale.
- **Env var presence test** — fails the build if a required `VITE_*` is unset in production mode.

## 2.5 Phase 5 — Review processes

### 2.5.1 Code review

- Every PR requires 1 human approval.
- Use the in-repo [`review`](../../../.claude/commands/review) and [`simplify`](../../../.agent/skills/simplify) skills as the AI second-opinion pass before requesting human review.
- Security-sensitive changes (auth, crypto, network, env vars) require the [`security-review`](../../../.claude/commands/security-review) skill.

### 2.5.2 Release review

For every tagged release:
1. Walk [release-checklist.md](release-checklist.md) end-to-end.
2. Use the [`release-note-writer`](../../../.agent/skills/release-note-writer) skill to draft notes.
3. Open a release PR; collect approvals from at least one maintainer.
4. Post-merge, tag and push.

### 2.5.3 Architecture decisions

- Use the ADR pattern established in [decision-social-module-flagging.md](decision-social-module-flagging.md).
- File path convention: `docs/ai/context/decision-<topic>.md`.
- Required for: dependency upgrades touching framework version, schema changes, feature flags, CI structural changes, hosting changes.

### 2.5.4 Accessibility review

- Run axe-core against the deployed preview as part of Lighthouse CI.
- Manual keyboard-only walkthrough of the chat flow before each release.
- Color-contrast: confirm theme tokens meet WCAG AA.

## 2.6 Phase 6 — Observability & runtime

### 2.6.1 Error tracking

- Add **Sentry browser SDK** (or alternative, e.g. PostHog error tracking which is already wired). Initialize in [`apps/stage-web/src/main.ts`](../../../apps/stage-web/src/main.ts) only when `VITE_SENTRY_DSN` is set.
- Source-map upload on release (Phase 2.2.2).
- Filter out network errors caused by self-hosters' misconfigured backends — they pollute prod telemetry.

### 2.6.2 Analytics (opt-in)

- Already supports PostHog (`VITE_ENABLE_POSTHOG`) and Plausible (proxied via Netlify redirects).
- Project-hosted deploy: enable Plausible by default (privacy-friendly, no cookies).
- Self-hosted: both off by default; document opt-in.
- Comply with whatever cookie banner / consent UI the project uses for the canonical deploy.

### 2.6.3 Performance monitoring

- Lighthouse CI on every PR (Phase 2.3.1) catches regressions pre-merge.
- Optional: Real-User Monitoring via PostHog or web-vitals beacon to backend.

### 2.6.4 Health endpoint

- Static site has no health endpoint. The frontend should ping the configured backend's `/health` and surface a banner if unreachable, so users self-diagnose.

## 2.7 Phase 7 — Documentation & launch

### 2.7.1 Deploy documentation

Create `apps/stage-web/docs/DEPLOYING.md` covering:
- Per-host quick start (Netlify, Cloudflare, Vercel, Docker)
- Required env vars (Phase 2.2.4)
- COOP/COEP header configuration (Phase 2.2.3)
- Backend URL configuration
- Custom domain setup
- Preview deploy verification

### 2.7.2 Runbook

Create `docs/ai/context/runbook-stage-web.md`:
- Common failure modes (backend unreachable, COOP/COEP missing, bundle limit exceeded, asset CDN miss)
- Rollback procedure: how to revert to prior deploy on each host
- Incident communication template

### 2.7.3 Release notes

Per release, [release-note-writer](../../../.agent/skills/release-note-writer) drafts notes following the v0.10.2 template. Cross-link from the release checklist artifact.

### 2.7.4 Launch checklist

Before tagging the first "production-ready" release (v0.11 candidate):
- [ ] Phase 1 gates all green
- [ ] Phase 2 build hardening done
- [ ] Phase 3 pipeline jobs all wired
- [ ] Phase 4 gates enforced in CI
- [ ] Phase 5 review docs in place
- [ ] Phase 6 error tracking live on staging for 1 week
- [ ] Phase 7 deploy docs + runbook merged

---

# Part 3 — Things easy to forget (checklist)

Items often missed in production planning. Each is a concrete to-do.

## 3.1 Security

- [ ] **Content Security Policy** — define a CSP header (with `'wasm-unsafe-eval'` for HF Transformers + DuckDB). Per-host header config.
- [ ] **Subresource Integrity** for any externally-loaded scripts (Plausible).
- [ ] **Dependency audit in CI** — `pnpm audit --prod` nightly; fail on critical.
- [ ] **Secrets scanning** — GitHub secret-scanning is on by default; add `gitleaks` pre-commit if any have leaked historically.
- [ ] **CORS policy on backend** — explicit allow-list for the canonical origin; reject `*` in production.
- [ ] **HTTPS-only**, HSTS header.

## 3.2 Accessibility

- [ ] Lighthouse a11y ≥ 90 (Phase 4.1).
- [ ] Keyboard-only navigation for chat flow.
- [ ] `prefers-reduced-motion` respected by character animations.
- [ ] Screen-reader labels on icon-only buttons (audit `packages/ui` and `packages/stage-ui`).
- [ ] Sufficient color contrast in light + dark themes.

## 3.3 Internationalization

- [ ] All strings sourced from [`packages/i18n`](../../../packages/i18n) (no inline English).
- [ ] Locale files have parity (Phase 4.3 test).
- [ ] RTL support if any RTL locales ship.
- [ ] Date/number formatting via `Intl` not hand-rolled.

## 3.4 Performance budgets

- [ ] Bundle size tripwire (Phase 2.2.1).
- [ ] Lighthouse perf ≥ 80 (Phase 4.1).
- [ ] Image optimization — verify any raster assets in `public/` are optimized; prefer SVG / WebP.
- [ ] Font loading strategy — `font-display: swap` for custom fonts; subset CJK fonts where possible.
- [ ] Code-split heavy ML / WASM dependencies behind dynamic imports.

## 3.5 Privacy & legal

- [ ] Privacy policy linked (project-hosted deploy).
- [ ] Cookie banner if cookies are used; current Plausible setup is cookieless.
- [ ] Terms of service / acceptable-use for the canonical deploy.
- [ ] Open-source license bundle in deploy footer or `/LICENSES.txt` (Phase 2.2.5).
- [ ] DPA / data-processing posture for any PII (chat content, voice samples).

## 3.6 Browser support

- [ ] Define matrix in `apps/stage-web/README.md`.
- [ ] Polyfill audit — confirm Vite targets reflect chosen matrix (`build.target`).
- [ ] Feature detection for WebGPU / SAB; user-visible fallback messaging.
- [ ] Test on Safari iOS for the public deploy (mic access, Live2D Canvas perf).

## 3.7 PWA

- [ ] Manifest icons present and correct.
- [ ] Service worker caching strategy reviewed; precache budget reasonable.
- [ ] "Update available" UX — surface to user when a new SW activates.
- [ ] Offline behavior — what happens when backend is unreachable?

## 3.8 Versioning

- [ ] Semantic versioning honored (semver post-v1.0; pre-1.0 minor bumps may be breaking — document).
- [ ] `package.json` `version` matches git tag.
- [ ] Build embeds version in UI (already supported via `unplugin-info`).
- [ ] Deprecation policy for env vars / API surfaces.

## 3.9 Documentation

- [ ] README at `apps/stage-web/README.md` updated each release.
- [ ] Top-level `README.md` deploy-status badges (CI, latest release).
- [ ] CHANGELOG either generated from release notes or kept manually.
- [ ] AGENTS.md updated when developer workflows change.

## 3.10 Operational

- [ ] On-call / contact for project-hosted incidents.
- [ ] Status page (optional; statuspage.io / Atlassian / self-hosted).
- [ ] Release cadence agreed (e.g. monthly minor, ad-hoc patch).
- [ ] Backup / DR for any project-hosted state (none for static site, but applies to backend).

---

# Critical files & references

- [`apps/stage-web/package.json`](../../../apps/stage-web/package.json) — scripts and deps
- [`apps/stage-web/vite.config.ts`](../../../apps/stage-web/vite.config.ts) — build config
- [`apps/stage-web/netlify.toml`](../../../apps/stage-web/netlify.toml) — Netlify config + triple-site assembly
- [`apps/stage-web/wrangler.toml`](../../../apps/stage-web/wrangler.toml) — Cloudflare Pages config
- [`apps/stage-web/Dockerfile`](../../../apps/stage-web/Dockerfile) — Nginx container
- [`.github/workflows/ci.yml`](../../../.github/workflows/ci.yml) — CI pipeline
- [`eslint.config.js`](../../../eslint.config.js) — lint config (Phase 1 regression check)
- [release-checklist.md](release-checklist.md) — verification protocol
- [build-verify-v0.10.3.md](build-verify-v0.10.3.md) — current verification state
- [decision-social-module-flagging.md](decision-social-module-flagging.md) — feature-flag pattern

# Verification — when this PRD is satisfied

- All Phase 1 gates green; v0.10.3 tagged.
- `pnpm build` produces a `dist/` under budget, with valid PWA manifest, COOP/COEP headers verified on at least one deployed host.
- CI runs all required gates from Phase 4.
- A staging preview URL has been live ≥ 24 h with no error-tracker spike.
- Rollback procedure executed at least once on a non-release commit.
- All checklist items in Part 3 either done or explicitly deferred with an issue link.

# Out of scope (recap)

- Electron / `stage-tamagotchi` production
- Mobile / `stage-pocket` distribution
- Backend (`apps/server`, `packages/server-runtime`) production
- New product features beyond what v0.10.x ships
- Migration off Vue 3 / Vite / Pinia / UnoCSS
