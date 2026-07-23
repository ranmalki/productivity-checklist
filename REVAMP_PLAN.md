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

## Migration / risk notes

- **No storage-key change.** Still `ttg_v2`; all existing user data loads unchanged. Any new fields will be added via `defaultState()` merge (non-destructive) — never rename/drop existing fields.
- Icons/manifest/SW are additive files; zero impact on existing logic.
- SW is scoped to `/` and precache uses `Promise.allSettled` so a single missing asset can't block install.

## Planned next increments (proposed order)

1. **Reward Vault** — new `S.rewards[]` (title, condition, XP/streak/smoke-free target, locked/ready/claimed, timestamps); vault UI grouped In-progress / Ready / Claimed; stronger celebration on unlock.
2. **Timed smoke-free sessions** — extend smoking module with start/duration/elapsed/remaining, craving log + reason, money-saved, neutral language; keep existing daily `cigs[]`.
3. **Daily Quest alignment flow** — quick guided picker: review unfinished → pick main quest / side quests / minimum action / reward → confirm. Framed on Home.
4. **Task attribute expansion** — priority, effort, energy, due/time, repeat, reward link; clearer visual hierarchy (main / side / bonus / minimum / deferred).
5. **Centralized XP config + duplicate-award guard** — single `XP` table; completion transaction IDs so toggling can't farm XP.
6. **Design-token pass** — consolidate spacing/radius/elevation/z-index tokens for the "order, not chaos" grid.

## Testing checklist

- [x] `manifest.webmanifest` valid JSON; `sw.js` parses; `index.html` loads with no JS errors.
- [x] Headless render: header, nav, splash.
- [ ] On-device (Pixel 7 / Chrome): install prompt, standalone launch, offline reload, maskable icon crop.
- [ ] iOS Safari: add-to-home-screen, apple-touch-icon, safe-area insets, standalone status bar.
- [ ] Existing-data load test: open with a pre-existing `ttg_v2` blob and confirm no data loss.
