# ARE — Deployment: Local Shell, Remote-Refreshable UI

**Status:** Living document
**Scope:** `src/frontend/` build output (`dist/`) and how it reaches
the WebView. Does not change `bridge.js`, the Python backend, or
anything in `SYSTEM-DESIGN-AGREEMENTS.md`.

`FRONTEND-VUE3-REFACTOR.md` defines the Vue 3 SPA that becomes
`src/frontend/`'s build output. This document covers a separate
question: where the WebView loads that build *from*, and why that
choice never touches the bridge boundary.

---

## 1. The problem this solves

Rapid UI iteration -- checking layout, animation, and CSS changes --
currently requires a full local ARE session: GNOME Kiosk, the Python
backend, and the WebView all running, just to see a component render.
That's a slow loop for frontend-only work, and it doesn't match how
`dist/` is actually built (a static Vite output with no dependency on
the machine it runs on).

Separately, once ARE ships, there's a real question of whether a user
should always be pinned to whatever `dist/` shipped in their install,
or be able to pick up UI fixes without a full release cycle.

Both are solved by treating `dist/` as something that can be *served*
from more than one place, without changing what serves the bridge.

## 2. What moves, what doesn't

**Moves:** the static `dist/` build -- HTML, CSS, JS, `assets/theme/`.
This is presentation only; see `FRONTEND-VUE3-REFACTOR.md` Section 2
for what's in it.

**Never moves:** `bridge.js`'s counterpart, the Python backend, and
the IPC channel between them. `SYSTEM-DESIGN-AGREEMENTS.md` already
establishes that D-Bus, NetworkManager, the audio server, and app
launching are owned by local system services ARE talks to directly --
none of that is reachable from, or safe to expose to, a remote edge
worker. Serving `index.html` from a CDN does not change where
`index.html`'s JS actually executes: it still runs in the same local
WebView, on the same machine, talking to the same local backend.

This is the one fact the rest of this document follows from: **the
source of the static files is a deployment detail; the bridge is
always local, always same-machine IPC, with no exception.**

## 3. Two consumers of `dist/`

### Dev-loop preview

`dist/` deployed to a Cloudflare Worker (static assets binding,
`not_found_handling = "single-page-application"`) on every build,
opened in an ordinary browser tab -- no GNOME Kiosk, no Python
backend, no WebView.

`bridge.js` feature-detects whether it's running inside ARE's own
WebView (a bridge object is injected there) or as a plain page in a
browser (no bridge object present). Outside the WebView, calls that
would normally cross the bridge resolve against fixture/stub data
instead of a real backend. This is a `bridge.js`-only concern --
components and composables call the same functions either way and
don't know which mode they're in.

This mode is for frontend iteration and demoing, not for anything
that exercises real system state. It's the natural home for
`FRONTEND-VUE3-REFACTOR.md`'s per-stage components before the backend
calls they depend on exist yet.

### Launcher runtime

An explicit setting, not an automatic fallback chain -- the user (or
a sane default) decides where the WebView's `dist/` comes from:

* **Local (default).** The WebView loads the `dist/` bundled in the
  release. `bridge.js` talks to the local backend. No network
  involved at all. This is the "wget the release, run it fully
  offline" path, unchanged from today.
* **Live/Remote UI.** On launch, ARE fetches the current `dist/` from
  the Cloudflare Worker and caches it locally before pointing the
  WebView at it. `bridge.js` still talks to the same local backend --
  only the source of the static files changed. If the fetch fails
  (no connectivity, Worker unreachable), ARE falls back to the last
  successfully cached `dist/`, or the bundled copy if nothing's been
  cached yet. This fallback only ever happens at launch time, once,
  not as a mid-session retry loop.

Both modes are the same WebView pointed at a different `index.html`
origin. Nothing about `bridge.js`'s real (non-stub) code path
differs between them.

## 4. Explicitly deferred

* Any bridge call reachable from the Cloudflare-hosted preview mode --
  stub data only, per Section 3. A remote HTTP surface onto D-Bus or
  app-launch control is not a feature to build later; it's outside
  what this document (or `SYSTEM-DESIGN-AGREEMENTS.md`) permits at
  all.
* Automatic background refresh of a running session's UI. The launch-
  time fetch-and-cache in Section 3 is the only network interaction
  this document defines; hot-swapping `dist/` under a live session is
  out of scope until there's a concrete need for it.
* The Worker deploy pipeline itself (CI wiring, `wrangler.toml`,
  preview-URL-per-branch). That's an implementation detail of the
  dev-loop workflow, not an architecture decision, and can be set up
  without amending this document.
* Signing/integrity verification of a remotely-fetched `dist/` before
  it's loaded into the WebView. Worth resolving before "Live/Remote
  UI" ships as a real setting, not before this document is agreed.
