# Index for Foundational Docs

* [PROBLEM-STATEMENT.md](PROBLEM-STATEMENT.md) -- what ARE is, the
  problem it exists to solve on Linux, and what it deliberately does
  not attempt.
* [ROADMAP.md](ROADMAP.md) -- the v1/v2/v3 staging: what each stage
  has to prove working (core launcher, then system integration, then
  personalization) before the next one starts, and what's held back
  until then.
* [CODE-STYLE.md](CODE-STYLE.md) -- how we write code on each side of
  the bridge: package-per-concern layout for the Python backend and
  the JS frontend, when a design pattern earns its place, and the
  shared logging convention across the bridge boundary.
* [SYSTEM-DESIGN-AGREEMENTS.md](SYSTEM-DESIGN-AGREEMENTS.md) -- who's
  allowed to own what, between ARE's own code and the Linux/GNOME
  services (D-Bus, NetworkManager, the audio server, systemd/logind)
  that actually hold most of the state ARE displays and acts on.
* [FRONTEND-VUE3-REFACTOR.md](FRONTEND-VUE3-REFACTOR.md) -- Stage 0
  plan for porting `src/frontend/`'s vendored GNOME Shell reference
  source into a Vue 3 SPA: target layout, the file-by-file porting
  method, and which vendored files feed which v1/v2/v3 stage.
* [DEPLOYMENT.md](DEPLOYMENT.md) -- where the WebView loads the built
  SPA from: a Cloudflare-hosted dev-loop preview with a stubbed
  bridge, versus the local/remote toggle for a real launcher install
  -- and why the bridge itself never leaves the local machine either
  way.

---

## Reading order

New to the project? `PROBLEM-STATEMENT.md` first, then `ROADMAP.md`
for the current stage, then `SYSTEM-DESIGN-AGREEMENTS.md` before
touching anything that reads or changes system state, then
`CODE-STYLE.md` once you're actually about to write code. Working on
the frontend specifically? `FRONTEND-VUE3-REFACTOR.md` after
`CODE-STYLE.md` -- it refines that document's frontend layout into the
concrete Vue 3 target and the plan for getting there. Setting up how
the built SPA gets served, whether that's your own dev-loop preview
or the launcher's local/remote toggle? `DEPLOYMENT.md`, after
`FRONTEND-VUE3-REFACTOR.md`.

## Philosophy

**The codebase should be legible to whoever opens it next, including
a future version of whoever wrote it.** A few things that follow from
that, here specifically:

* **Explain the constraint, not just the code.** `PROBLEM-STATEMENT.md`
  exists to answer "why a launcher, why this stack, why not a full
  desktop-environment rewrite" -- not "what does the code do," which
  the code itself already answers.
* **The bridge is a boundary to respect, not a shortcut to route
  around.** `SYSTEM-DESIGN-AGREEMENTS.md` exists because it's easy for
  a Python-owned piece of state to quietly grow a second copy in JS,
  or vice versa, the first time that's more convenient than going
  through the bridge properly.
* **Don't reach for structure the problem hasn't asked for yet.** A
  design pattern, a new module, a new abstraction earns its place
  because it's the accurate name for a constraint the code already
  ran into -- see `CODE-STYLE.md` Section 2.
* **Stay small on purpose, per stage.** `ROADMAP.md` exists
  specifically so "wouldn't it be nice to also do X now" has somewhere
  to go that isn't the current stage's scope.
* **Bugs stay visible until they're actually gone.** See
  `../bugs-caught/README.md` for the fixed/tested/confirmed bar every
  bug has to clear before it's removed from the tracker.
