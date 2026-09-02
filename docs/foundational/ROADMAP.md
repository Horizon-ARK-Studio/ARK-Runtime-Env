# ARE — Roadmap

**Status:** Living document
**Scope:** whole project

`PROBLEM-STATEMENT.md` explains why ARE looks the way it does. This
document says what has to actually work, concretely, before a stage
counts as done — and just as importantly, what's deliberately held
back until then. A stage that quietly grows scope mid-way is exactly
the kind of undocumented decision that looks like an accident later,
so each stage below gets a fixed "in scope" list it isn't allowed to
drift past.

---

## v1 — Core launcher

**Goal:** an app grid and search that actually launch real
applications on a real GNOME session, driven by a small, explicit
JS ↔ Python bridge — everything else in `PROBLEM-STATEMENT.md`'s
goal list is downstream of this working first.

**In scope:**
* application discovery from standard `.desktop` entries (name, icon,
  exec line, categories) — no bespoke app manifest format
* the launcher UI: app grid, search-as-you-type, and launching an
  app by dispatching its `.desktop` exec line through the bridge
* the bridge itself: a small, explicit set of JS-callable functions
  backed by Python, not a general-purpose RPC surface
* running full-screen under GNOME Kiosk

**Explicitly deferred:**
* anything that isn't "find an app, launch an app" — widgets,
  wallpaper, network/audio/power controls, favorites, layout
  customization all wait for v2/v3
* touch input specifically — the architecture should not preclude it
  later, but nothing in v1 is built or tested against it
* any bridge call that isn't needed by app discovery, search, or
  launching yet

**Done when:**
* every installed application with a valid `.desktop` entry appears
  in the grid, with its correct name and icon
* search narrows the grid by name/keyword as the user types, with no
  visible lag on modest hardware
* launching an app from the grid or from search starts that
  application normally, with no modification to the application
  itself
* the launcher runs as the GNOME Kiosk session's shell and survives a
  normal session lifecycle (login, logout, restart) without manual
  intervention
* no outstanding entries in `bugs-caught/` for anything in this list

---

## v2 — System integration

**Goal:** the system-facing controls from `PROBLEM-STATEMENT.md`'s
goal list — network, audio, power, and system information — each
backed by the actual Linux/GNOME service that owns that state, not by
a value the launcher invents and tracks on its own.

**In scope:**
* network status and basic control (view current connection, toggle
  Wi-Fi, join/forget a known network) via the system's existing
  network service
* audio control (volume, mute, output device selection) via the
  system's existing audio service
* power actions (suspend, restart, shut down, and whatever the
  session's idle/lock behavior should be) via the system's existing
  session/power service
* a system information surface (battery, storage, basic hardware
  info) sourced the same way — read, not re-implemented

**Explicitly deferred:**
* any control surface for a subsystem the launcher doesn't have a
  working v1 pattern to extend from
* anything in v3's scope (widgets, wallpaper, favorites, layout)

**Done when:**
* each control listed above reflects real system state on load, not
  a cached or assumed default
* changing a value through the launcher (e.g. adjusting volume) is
  visible immediately in the system's own tooling, and a change made
  through the system's own tooling is reflected back in the launcher
  without requiring a restart
* a missing or unavailable backing service (no D-Bus service running,
  no network manager present, etc.) is handled as an explicit,
  visible state in the UI — never a silent no-op or a crash
* no outstanding entries in `bugs-caught/` for any control covered by
  this stage

---

## v3 — Personalization

**Goal:** the parts of `PROBLEM-STATEMENT.md`'s goal list that make
the launcher feel like *your* launcher rather than a fixed shell:
widgets, wallpaper, favorites, and layout customization, all
persisted independently of system application data.

**In scope:**
* a widget surface (at minimum: something time/date-based and
  something backed by v2's system-info surface) with a defined way
  for a widget to read state without reaching past the bridge
* wallpaper selection and persistence
* favorites and launcher layout customization (pinning, reordering,
  grouping)
* a configuration store for all of the above that lives entirely
  under ARE's own config path, never inside another application's or
  the system's own configuration

**Explicitly deferred:**
* a third-party/plugin widget API — v3 ships the widgets ARE itself
  needs; a general extension surface is a later conversation once
  there's more than one real widget's worth of precedent to design it
  from
* touch-specific customization gestures — same reasoning as v1

**Done when:**
* wallpaper, favorites, and layout choices persist across restart and
  logout/login
* removing or resetting ARE's own configuration does not affect any
  installed application's own settings, and vice versa
* at least the two widgets described above render correctly and stay
  in sync with the state they display
* no outstanding entries in `bugs-caught/` for anything in this list

---

## Deferred indefinitely, not a numbered stage

* touch input as a first-class interaction model (see
  `PROBLEM-STATEMENT.md` — the architecture should allow it, nothing
  currently commits to shipping it)
* a general third-party plugin/extension system
* replacing GNOME's window manager, or any of the non-goals listed in
  `PROBLEM-STATEMENT.md`

These aren't "not yet" in the sense of "next after v3" — they're held
outside the staged plan entirely until there's a concrete reason,
grounded in a real v1-v3 constraint, to open one of them up as its
own fully-scoped stage.
