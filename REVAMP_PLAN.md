# Ran Hub — Revamp Plan

Living checklist for turning the existing "Ran's Hub" checklist into a polished, installable
**Ran Hub** PWA. Guiding principle: **Order, not chaos** — preserve everything that works,
enhance in tested increments, never ship a broken tree.

## Architecture audit (source of truth: existing `index.html`)

- **Stack:** single self-contained `index.html`, no build step, deployed on Vercel (push to `main` → auto-deploy). Vanilla JS, CSS variables, RTL Hebrew UI.
- **Persistence:** `localStorage` key **`ttg_v2`**. `load()` merges saved state over `defaultState()`; `save()` writes the whole `S` object. Daily `rollover()` snapshots to `S.hist[day]`.
- **State model (`S`):** `tasks[]`, `totalExp`, `streak`/`lastDoneDate`, `doneToday[]`, `log[]` (completion records), `expByDate{}`, `projects[]`, `wins[]`, per-module quiz XP (`noteMod`/`fifthsMod`/`guitarMod`), `checkin{mood,energy,focus,note,music,helped}`, `cigs[]` (smoking log), `weed`/`genshin` avoid-streaks, `musicIdea`, `studio`, `overwhelm`, `muted`, `hist{}`.
- **Progression:** `EXP_PER_LEVEL=80`, `levelOf()`, `RANKS[]` (22 ranks, lv 1→100), `QUIZ_RANKS[]`. Level-up + rank-up celebrations with `spawnConfetti()` + WebAudio `beep()` chimes.
- **Views (nav, 8 tabs):** Home, Tasks, Projects, Calendar, Training (note reading / circle of fifths / guitar quizzes), Notepad, History, More. Progress screen has Daily / Done / Ranks segments.
- **Existing feedback:** confetti, generated sounds (mute toggle), floating level-up overlay, streak flame, mood check-in, "overwhelm" focus mode, quick-win chips.

## Feature parity map (spec → current status)

| Spec area | Status | Notes |
|---|---|---|
| Categorized tasks | ✅ partial | tasks have cat/tag/exp/icon; add priority/effort/energy/due/repeat |
| Daily quest system | ⚠️ implicit | tasks + overwhelm mode exist; no explicit main/side/minimum alignment flow |
| XP / levels / streak | ✅ | single-ish source; harden against duplicate awards (completion IDs) |
| Task completion feedback | ✅ | confetti + sound + level overlay; add floating +XP near item |
| Mood / energy check-in | ✅ | `checkin` object already stored |
| Smoking reduction | ⚠️ basic | daily `cigs[]` log + avoid-streaks; add timed smoke-free **sessions** |
| Reward vault | ❌ | not present — to build |
| Music / creative focus | ✅ | music tasks, ideas, training quizzes |
| PWA (installable) | ✅ **done** | manifest + SW + icons + splash (this pass) |
| App icon / branding | ✅ **done** | astronaut-R artwork wired as icon/favicon/apple-touch/splash |

## Done in this pass (shipped)

- [x] Rename to **Ran Hub** (title, manifest, apple title, in-app wordmark reads RAN / HUB).
- [x] Generate icon set from the supplied artwork (LANCZOS, preserves lighting): `icons/icon-192/512(.maskable).png`, `apple-touch-icon.png`, `favicon-16/32.png`, `favicon.ico`.
- [x] `manifest.webmanifest` — standalone, portrait, scope `/`, id `/`, theme/background `#0c0a1e`, any + maskable icons, categories, RTL.
- [x] `sw.js` service worker — app-shell precache, cache-first assets, network-first navigations with offline fallback, cross-origin (fonts) stale-while-revalidate, cache versioning.
- [x] Head meta: manifest, theme-color, `mobile-web-app-capable`, Apple standalone metadata, `apple-touch-icon`, favicons.
- [x] Branded launch **splash** using the icon, auto-hides on load (with safety timeout + reduced-motion respect).
- [x] SW registration on load (guarded, silent-fail).
- [x] Full-width gold-emblem header title up to the action buttons.
- [x] Smoke-tested in headless Chromium: title = "Ran Hub", splash hides, nav renders, no app JS errors (only sandbox font fetch fails, which the SW handles gracefully online).

## Status: revamp roadmap complete

All planned increments (#1–#6) and the cross-cutting spec areas (§17 return, §18 completion feedback, §19 performance, §20 accessibility, §21 data safety, §22 install, Phase 7 support systems) are implemented, headless-tested, committed, and deployed to the branch preview. Remaining known non-goals: weekly task repeat (deferred), a deep global CSS-token rewrite (intentionally skipped for regression safety), and a formal contrast audit. On-device (Pixel 7 / iOS) verification is the outstanding manual step before merge to production.

## Migration / risk notes

- **No storage-key change.** Still `ttg_v2`; all existing user data loads unchanged. Any new fields will be added via `defaultState()` merge (non-destructive) — never rename/drop existing fields.
- Icons/manifest/SW are additive files; zero impact on existing logic.
- SW is scoped to `/` and precache uses `Promise.allSettled` so a single missing asset can't block install.

## Planned next increments (proposed order)

1. ~~**Reward Vault**~~ ✅ shipped — `S.rewards[]` (title, emoji, condition type xp/streak/smoke/manual, target, base, repeatable, status). Vault view under More, grouped Ready / In-progress / Claimed, progress bars, add-form, stronger unlock celebration. Additive state (merges over old saves).
2. ~~**Timed smoke-free sessions**~~ ✅ shipped — `S.smoke{active,sessions[]}`. Live HH:MM:SS timer, duration picker, craving log, stats (total clean hours / sessions / best), neutral language, completion celebration. Feeds the `smoke` reward condition. Existing daily `cigs[]` untouched.
3. ~~**Daily Quest alignment flow**~~ ✅ shipped — `S.quest{date,mainId,sideIds[],minId,rewardId,setAt}`. Home-top quest card (empty → "build" CTA; active → main/side/minimum slots, live progress `done/total`, optional linked reward, done-state celebration). Guided builder modal (radio main + minimum, multi side, optional reward from Vault). Completion routes through existing `completeTask`/`uncompleteTask` so XP/streak stay single-source. Additive state (date-scoped; stale automatically next day).
4. ~~**Task attribute expansion**~~ ✅ shipped — tasks gain optional `priority` (high/normal/low), `due` (date), `energy` (low/med/high), `repeat` (daily/once). Non-destructive: `normalizeTasks()` fills defaults for any legacy task on load. Editor modal gets priority/due/repeat/energy controls. Rows show meta badges (due w/ overdue state, once, high-priority, low-energy) + priority accent, and every list is `sortTasks()`-ordered (undone → priority → due). One-off (`once`) tasks completed on a day are archived (already in the log) and retired at rollover. Weekly repeat deferred (needs per-task rest tracking). Reward-link lives on the Daily Quest for now.
5. ~~**Centralized XP config + duplicate-award guard**~~ ✅ shipped — single `XP` table (`quickAction`/`sub`/`project`) now feeds music quick-actions and the project constants (`SUB_XP`/`PROJECT_XP`). Duplicate-award guard verified (re-toggle 0→30→0, no double-award; `doneToday` gate). Also added the **floating "+N XP"** completion primitive (rises from the tapped element, 1000ms, reduced-motion aware, self-cleaning) — the plan's outstanding "floating +XP near item" feedback item.
6. ~~**Design-token pass**~~ ✅ shipped (safe scope) — added a documented spacing scale (`--s1..--s6`) and a z-index tier system (`--z-bg…--z-splash`) to `:root`, and adopted the z-index tokens across every global stacking layer (bg, grain, header, nav, fx, toast, modal, quiz, level-up, splash) with **identical numeric values** (verified: zero visual change). A deeper global rewrite of the hundreds of existing rules was intentionally skipped — high regression risk on the working UI for no user-visible gain.

## Support systems (Phase 7) — shipped

- [x] **Rest / low-energy mode** — a Home toggle (`S.rest`) that narrows the day to low-energy tasks + the Daily Quest minimum, with calm, non-judgmental copy and an SR announcement. Builds on the task `energy` attribute. Reward Vault, smoke-free sessions, and Music/Studio mode were delivered earlier; the low-energy recovery flow completes the phase.

## Performance (§19) — shipped

- [x] **HTML payload trim** — the cosmic-jungle backdrop (~320KB JPEG) was inlined as a base64 data-URI; externalized to `bg.jpg` and referenced via `url('bg.jpg')`. `index.html` 609KB → 180KB (−70%), faster first paint + parse. Added `bg.jpg` to the SW precache (cache `v1`→`v2`) so offline keeps the backdrop.

## Final QA sweep (headless Chromium) — all green

- [x] Legacy `ttg_v2` (pre-revamp shape) migrates: tasks backfill priority/repeat/energy/due; XP/streak/log preserved.
- [x] All 11 secondary views + Home render with no JS errors.
- [x] Task create w/ attributes, Daily Quest build→complete, export download (`ran-hub-backup-<date>.json`), reduced-motion completion — all pass.
- [x] Modal dialog semantics + focus trap/return + Escape.
- [x] z-index tokens resolve to expected values; `bg.jpg` decodes (941×1672).

## Return experience (§17) — shipped

- [x] **Welcome-back states** — `S.lastVisit` tracks the last open; `computeVisitGap()` derives days away at load. Same-day → nothing. 1-day gap → compact time-of-day greeting + "continue where you left off" (or a "build daily quest" CTA). 2+ day gap → a gentle, shame-free welcome card ("everything's saved") with a 7-day recap (tasks + XP) and a fresh-Daily-Quest CTA. Dismissible; shows once per load. Verified for 0/1/3-day gaps.

## Accessibility (§20) — first pass

- [x] **Status announcements** — an `aria-live="polite"` visually-hidden region (`#sr`) announces completion ("+N XP, total today …") and level/rank-ups for screen readers, via `announce()` from `awardExp`.
- [x] **Keyboard operability** — a `kbd()` helper gives the primary interactive surfaces (task checkboxes, Daily-Quest slots, quest-builder picker rows) `role="button"`, `tabindex`, `aria-label`, and Enter/Space activation. Verified: Tab-focus + Enter completes a task and announces it.
- [x] Reduced-motion already respected globally + in the new floating-XP/welcome animations.
- Remaining: focus trap/return for modals, full sweep of legacy div-buttons, contrast audit.

## Also shipped this pass

- [x] **Data safety (§21)** — Export backup (`ran-hub-backup-YYYY-MM-DD.json`) and Import/restore in Settings. Import validates JSON shape (`tasks` array), confirms before replacing, merges over `defaultState()` (non-destructive), local-only (no upload).
- [x] **Install experience (§22)** — captures `beforeinstallprompt`, shows a header install button, triggers the native prompt on intent, hides on `appinstalled`/standalone; iOS Safari falls back to an "Add to Home Screen" hint (no fake Android prompt). `display-mode: standalone` detection.
- [x] Headless-tested (Chromium): quest build→save→complete→uncomplete flow, EXP 0→30→0, progress `1/2`, export/import present, no app JS errors (only the sandbox cross-origin font fetch, which the SW handles online).

## Testing checklist

- [x] `manifest.webmanifest` valid JSON; `sw.js` parses; `index.html` loads with no JS errors.
- [x] Headless render: header, nav, splash.
- [ ] On-device (Pixel 7 / Chrome): install prompt, standalone launch, offline reload, maskable icon crop.
- [ ] iOS Safari: add-to-home-screen, apple-touch-icon, safe-area insets, standalone status bar.
- [ ] Existing-data load test: open with a pre-existing `ttg_v2` blob and confirm no data loss.
