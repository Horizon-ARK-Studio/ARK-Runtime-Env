# ARE — System Design Agreements

**Status:** Living document
**Scope:** whole project (architectural, not file-specific)

`CODE-STYLE.md` explains how code should be shaped once you know what
it owns. This document is one level up: **who is allowed to own
what**, across ARE's three layers — the JS frontend, the Python
backend, and the underlying Linux/GNOME services (D-Bus, NetworkManager,
the audio server, systemd/logind, GNOME Kiosk's own session and window
management).

This split matters more here than in most launcher-shaped projects,
because ARE never owns most of the state it displays. Volume, network
connectivity, battery level, and session/power state are all owned by
a Linux service that existed before ARE launched and keeps running
whether or not ARE is looking at it. ARE's job across all three layers
is to *read and act on* that state correctly, never to invent a
second, competing copy of it.

---

## The agreement

> **A system service that already owns a piece of state is the only
> source of truth for that state. Every layer of ARE — Python backend,
> JS frontend, and the bridge between them — must either (a) read from
> and act through the service that owns it, or (b) own something no
> other system component touches (ARE's own config: favorites, layout,
> wallpaper choice) — never (c) cache a value and treat the cache as
> if it were the state itself.**

This is the reason `system/audio.py` doesn't keep its own "current
volume" variable that the frontend polls — it asks the audio server,
every time, and reports whatever the server says. A cached value looks
identical to a correct one right up until something outside ARE (a
hardware volume key, another application, the system's own settings
panel) changes the real state and ARE's cache silently goes stale.

---

## Applying it: the ownership test

Before adding any code — on either side of the bridge — that touches
a piece of state a system service might also touch (network state,
audio state, power/session state, window/session lifecycle under
GNOME Kiosk, installed-application metadata):

1. **Does an existing Linux/GNOME service already own this state?** If
   yes, neither the Python backend nor the JS frontend keeps an
   independent copy. The backend reads from the service on demand (or
   subscribes to the service's own change notifications, where one
   exists) and the frontend reflects whatever the backend just
   reported — it does not maintain a separately-updated value of its
   own.
2. **If ARE must act on that state (toggling Wi-Fi, changing volume,
   suspending the session), does the action go through the service's
   own API, and is it safe to call more than once?** An action that
   silently double-applies (e.g. toggling Wi-Fi twice because a UI
   click handler fired twice) is a bug the same way an unguarded state
   transition is a bug in any other kind of system code — guard the
   action, not just the display.
3. **Can ARE's own UI update be observed by the underlying service (or
   by another application) as an event it reacts to, which ARE's UI
   then reacts to in turn?** A volume slider that re-reads and
   re-writes on every notification from the audio server, including
   notifications caused by its own last write, is exactly this loop.
   Steps 1 and 2 should already prevent it; if they don't, that's the
   bug to find before writing more system-integration code.

This test is the same regardless of which subsystem it's being
applied to (network, audio, power, or anything added later) — the
correct answer for each is discovered empirically, per service, as
`ROADMAP.md`'s v2 work actually happens, not assumed by analogy from
whichever subsystem was integrated first.

---

## Where ARE genuinely does own state

Not every disagreement resolves to "defer to the system." ARE owns,
outright, anything no other application or system service has any
stake in:

* launcher layout, favorites, and pinning order
* wallpaper selection (as ARE's own choice of what to display — not
  as a system-wide desktop-background setting some other tool also
  claims)
* widget configuration and any widget-local state
* the bridge's own internal bookkeeping (pending calls, logged errors)

This state lives in `config/` on the backend side, per
`CODE-STYLE.md`, specifically because it's the one category of state
where ARE is the only owner and there's no external source of truth
to defer to or drift out of sync with.

---

## Non-goals

This is not "the frontend and backend must never touch anything a
system service also touches" — reading and acting on network, audio,
and power state is the entire point of `ROADMAP.md`'s v2. The
agreement is narrower: when ARE reads or acts on that state, it must
do so *through* the owning service, honestly reflect whatever that
service reports, and never let a cached or assumed value stand in for
a live check — on either side of the bridge.
