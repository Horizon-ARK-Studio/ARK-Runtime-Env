# ARE — How We Write Code

**Status:** Living document
**Scope:** `src/backend/` (Python), `src/bridge/`, `src/frontend/` (HTML/CSS/JS)

This isn't a brace-placement or PEP 8 recap. It's the structural
decisions this codebase should keep making on purpose, so a change
follows the same shape instead of drifting toward one module that
quietly grows into "the file that talks to everything" — which is an
easy failure mode for exactly this kind of project, where a Python
backend, a JS frontend, and a thin bridge between them are tempted to
blur into each other the moment a feature needs both sides.

---

## 1. One Module, One Reason to Change

A module exists because one thing can change independently of
everything else — not because it was convenient to keep adding to
whatever file was already open when a feature needed "just one more
function."

**Backend (`src/backend/`)**, split by concern, not by technology:

```
backend/
├── discovery/     -- reading .desktop entries, building the app index
├── launch/        -- turning a discovered app into a running process
├── system/         -- network, audio, power, system-info: one submodule
│                      per subsystem, each owning exactly one D-Bus/
│                      system-service integration
├── config/         -- ARE's own persisted settings (favorites, layout,
│                      wallpaper choice) -- never another app's config
└── logging/         -- shared logging convention, see Section 3
```

`system/network.py`, `system/audio.py`, and `system/power.py` are
separate files, not one `system.py` — each owns a different backing
service, with a different failure mode and a different "is this
service even available" check. If a change to how volume works
requires touching the network code to compile, that's the signal
`system/` has drifted into one shared file's worth of concerns again.

**Frontend (`src/frontend/`)**, mirroring the same instinct in JS:

```
frontend/
├── main.js            -- entry point + wiring only
├── ui/
│   ├── appgrid.js       -- grid rendering, icon layout
│   ├── search.js         -- search-as-you-type
│   ├── widgets.js         -- v3: widget surface
│   └── controls.js         -- v2: network/audio/power/system-info UI
├── bridge.js           -- the one file that calls into Python; nothing
│                           else in frontend/ talks to the bridge directly
├── state.js             -- in-memory UI state, kept separate from
│                           whatever the bridge reports as source of truth
└── config-view.js        -- reads/writes ARE's own settings through the
                             bridge; does not persist anything itself
```

`bridge.js` being the only file that calls into Python is deliberate,
not incidental: it's the one place a change to the bridge's actual
call surface has to be reflected, and the one place to look when a
bridge call starts failing. `ui/*.js` files render and react to state;
they don't reach around `bridge.js` to talk to Python directly, even
if it would save a function call.

---

## 2. Reach for a Pattern When It Names a Real Constraint, Not by Default

A design pattern — on either side of the bridge — earns its place
because it's the accurate name for a constraint the code already hit,
not because "that's how you'd structure this in general."

Two constraints already known to apply, given ARE's own shape:

* **Multiple backends behind one system-control interface.** Linux
  audio might be PulseAudio or PipeWire depending on the distro;
  network control might go through NetworkManager's D-Bus API or need
  a fallback path. `system/audio.py` and `system/network.py` each
  need one stable interface the frontend calls through
  `bridge.js`, with the actual backend implementation swapped out
  underneath. That's a Strategy-shaped constraint, reached for because
  v2 genuinely has more than one real implementation to choose
  between — not applied speculatively to `discovery/` or `launch/`,
  which don't have that problem.
* **A single place bridge failures get logged.** The JS ↔ Python
  boundary is exactly the kind of seam that can fail silently — a
  Python exception in a bridge-called function doesn't automatically
  surface as a JS error, and vice versa. That's the constraint
  Section 3's logging convention exists to name, not a general
  "let's have logging" decision.

Any other pattern — on the Python side or the JS side — gets added
the same way: identify the constraint first, name the pattern after
it, not before.

---

## 3. Logging Convention

The bridge boundary is the single most likely place for a failure to
go unnoticed: a Python-side exception inside a bridge-called function,
or a JS-side error thrown from a bridge callback, can each fail in a
way that never reaches the other side as a visible error.

* Every Python function exposed across the bridge wraps its body in a
  try/except that logs through `logging/`'s shared logger before
  re-raising or returning an explicit error value — never a bare
  `except: pass`.
* Every `bridge.js` call site checks for and logs a bridge-reported
  error the same way, rather than assuming a call succeeded because it
  didn't throw.
* System-service integration (`system/*.py`) additionally logs when a
  backing service is unavailable at all (no D-Bus service running, no
  network manager present) — that's a distinct, expected failure mode
  from an unexpected exception, and `ROADMAP.md`'s v2 "done when" list
  requires it to be visible, not swallowed.

The specific logger implementation (Python's standard `logging` module
configured once in `logging/`, mirrored to a JS-side console wrapper
in the frontend) is a v1 decision, made once there's real bridge code
to log around — not speculated ahead of it.

---

## 4. This Document Grows With the Code, Not Ahead of It

Sections 1-3 cover what's already a known, real constraint, from the
shape v1-v3 already imply. Anything else — test conventions, build
tooling, a concrete first cut at the widget API's own package layout —
gets added here once real code exists to derive it from, consistent
with `docs/foundational/README.md`'s own philosophy: don't reach for
structure the problem hasn't asked for yet.
