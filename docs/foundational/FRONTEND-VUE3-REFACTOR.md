# ARE — Frontend Refactor: Porting to a Vue 3 SPA

**Status:** Living document
**Scope:** `src/frontend/` only
**Stage:** 0 (planning — no application code lands until this doc is
agreed and `ROADMAP.md`'s v1 work actually starts)

`docs/vendored/README.md` already states the plan this document makes
concrete: `gnome-shell-ui/` and `gnome-shell-theme/` are reference
source, not code ARE runs, and `src/frontend/` is meant to become a
Vue 3 rewrite. This document is the "how" — the target shape of
`src/frontend/`, how each vendored file maps to it, and the porting
method — so that work doesn't drift into an ad hoc rewrite the same
way `CODE-STYLE.md` Section 4 warns against for the rest of the
codebase.

---

## 1. Why Vue 3, and what "port" means here

ARE's frontend runs inside WebKit/WebView (`PROBLEM-STATEMENT.md`), so
it's an SPA the same way any embedded web UI is — one HTML document,
one JS runtime, no server-rendered navigation. Vue 3 gives that SPA
component boundaries, reactive state, and a build step, without
requiring the UI *logic* already sitting in `gnome-shell-ui/` to be
reinvented.

**"Port," not "rewrite from scratch," means:**

* Where a vendored file's value is in its *logic* — layout math,
  filtering/sorting, state machines, animation timing, event
  sequencing — that logic is carried over largely as-is: same
  algorithm, same structure, same names where they still make sense,
  moved into a plain JS module (a composable or a plain helper) and
  called from a Vue component instead of a GJS `St.Widget` subclass.
* Where a vendored file's value is in its *rendering* — building
  `St`/`Clutter` actor trees — that part is not portable at all
  (`St`/`Clutter` don't exist in a browser) and gets replaced by a
  Vue SFC's `<template>`, informed by the same visual/interaction
  intent, not translated line-by-line.
* Nothing gets pulled in unless a stage in `ROADMAP.md` actually needs
  it yet — see Section 4.

This keeps faith with `CODE-STYLE.md` Section 2: Vue components,
composables, and the rest of the structure below are being reached
for because the frontend already has a concrete constraint (a large
body of reference UI logic in a non-web runtime that needs a real web
home) — not adopted because "that's how you'd structure a SPA."

## 2. Target `src/frontend/` layout

This refines `CODE-STYLE.md` Section 1's frontend sketch into the
concrete Vue 3 shape, rather than replacing it:

```
frontend/
├── index.html            -- SPA shell; single mount point
├── main.js                -- entry point + wiring only, per CODE-STYLE.md
├── App.vue                 -- top-level layout (shell chrome, view switch)
├── components/
│   ├── appgrid/             -- v1: app grid rendering, icon layout
│   ├── search/               -- v1: search-as-you-type UI
│   ├── controls/              -- v2: network/audio/power/system-info UI
│   └── widgets/                 -- v3: widget surface
├── composables/
│   ├── useAppGrid.js          -- v1: grid/icon layout logic, ported
│   ├── useSearch.js             -- v1: search-as-you-type logic, ported
│   ├── useControls.js             -- v2: control-surface logic, ported
│   └── useWidgets.js                -- v3: widget-surface logic, ported
├── bridge.js               -- unchanged role: the one file that calls
│                               into Python; nothing else talks to the
│                               bridge directly (CODE-STYLE.md Section 1)
├── state.js                  -- in-memory UI state, kept separate from
│                                 whatever the bridge reports as source
│                                 of truth (CODE-STYLE.md Section 1)
├── config-view.js              -- reads/writes ARE's own settings
│                                   through the bridge
└── assets/
    └── theme/                   -- ported from gnome-shell-theme/, see
                                     Section 3
```

`components/*` hold markup and presentation only. `composables/*` hold
the ported logic — the part of each vendored file worth keeping — as
plain, framework-light functions a component calls into. This mirrors
the existing `bridge.js`/`ui/*.js` split in `CODE-STYLE.md`: rendering
files don't grow business logic any more than `ui/*.js` files were
meant to reach around `bridge.js`.

`gnome-shell-ui/` and `gnome-shell-theme/` stay in place, unmodified,
for the duration of the port — they're the reference, per
`docs/vendored/README.md`, and get removed only once nothing in
`components/` or `composables/` still needs to be read against them.

## 3. Porting method, file by file

For each vendored file pulled into a stage's scope:

1. **Read it against what it's replacing.** Identify which parts are
   GJS/`St`/`Clutter`-specific rendering (not portable) versus
   plain-JS logic (portable): layout calculations, filtering/sorting,
   debounce/throttle timing, state transitions, event ordering.
2. **Extract the portable logic into a composable**, keeping its
   internal structure and naming recognizable against the original —
   this is what "use JS as is, don't reinvent the wheel" means
   concretely: the algorithm doesn't get redesigned on the way over,
   just re-hosted.
3. **Write a new Vue SFC for the rendering half**, using the
   composable for its data/behavior. The SFC's template is new (DOM/
   CSS instead of `St`/`Clutter` actors); the logic it calls is not.
4. **Preserve provenance per `docs/vendored/README.md`.** Every ported
   file keeps a short header comment naming the original upstream
   path it was ported from (e.g. `// Ported from
   gnome-shell-ui/appDisplay.js`), since the rewrite remains a
   GPL-2.0-or-later-derived, GPLv3 work regardless of how different it
   ends up looking.
5. **Wire system-facing calls through `bridge.js` only**, per
   `SYSTEM-DESIGN-AGREEMENTS.md` — a composable ported from vendored
   code that assumed direct system access (GJS can call into GNOME
   Shell internals directly; ARE's frontend cannot) gets that access
   routed through the bridge, not reimplemented as a direct call.

`gnome-shell-theme/`'s SCSS is handled separately from the JS port: it
compiles to CSS today the same way it will inside a Vite build (Sass
is a preprocessing step, not a GJS-runtime dependency), so the theme
assets move into `assets/theme/` largely unchanged, with GNOME
Shell-specific selectors (`#panel`, `.workspace-thumbnail`, etc.)
adapted to whatever ARE's own component markup actually produces.

## 4. Staging: what gets ported when

Porting follows `ROADMAP.md`, not the other way around — a vendored
file doesn't get touched until its stage is active, even if it would
be convenient to port everything up front:

| Stage | Pulls from (`gnome-shell-ui/`) | Lands in |
|---|---|---|
| v1 | `appDisplay.js`, `iconGrid.js`, `search.js`, `searchController.js` | `components/appgrid/`, `components/search/`, `composables/useAppGrid.js`, `composables/useSearch.js` |
| v2 | `status/network.js`, `status/volume.js`, `status/system.js`, `status/backlight.js`, relevant `quickSettings.js` pieces | `components/controls/`, `composables/useControls.js` |
| v3 | `calendar.js` (date/time widget precedent), relevant `panel.js`/`layout.js` layout logic, `backgroundMenu.js` (wallpaper) | `components/widgets/`, `composables/useWidgets.js` |

Anything in `gnome-shell-ui/` not named above (window management,
workspaces, multi-monitor handling, accessibility dialogs tied to
GNOME Shell's own session model, etc.) stays reference-only
indefinitely, consistent with `ROADMAP.md`'s non-goals and
`PROBLEM-STATEMENT.md`'s non-goals — ARE is not reimplementing a
window manager or the full GNOME session, so there's no stage that
needs those files ported.

## 5. Build tooling

* Vite as the dev server/bundler — matches a WebView-hosted SPA and
  Vue 3's own tooling default, no separate justification needed.
* `vue` (no Options API-only constraints; Composition API is what
  `composables/*` are written against) and `sass` (for the vendored
  SCSS carried into `assets/theme/`) as the two frontend-specific
  dependencies this refactor introduces.
* No router. `PROBLEM-STATEMENT.md`'s launcher is a single view with
  internal mode-switching (grid ↔ search ↔ controls ↔ widgets), not
  distinct navigable pages — `App.vue` handles that switch directly,
  per Section 2's `App.vue` role.

## 6. Done when (Stage 0)

* This document is in place and cross-referenced from
  `docs/foundational/README.md`.
* No `components/`, `composables/`, `bridge.js`, `state.js`, or
  `config-view.js` exist yet — Stage 0 is the plan, not the port.
  Actual files land as `ROADMAP.md` v1 work begins, per Section 4's
  staging table.
* Anyone opening `src/frontend/` for the first time can find, from
  this doc plus `docs/vendored/README.md`, which vendored file
  explains which piece of upcoming UI logic, before any of it exists
  as Vue code.
