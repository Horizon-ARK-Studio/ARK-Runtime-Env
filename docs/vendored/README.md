# Index for Vendored Source

* [Gnome js and styling/README.md](Gnome%20js%20and%20styling/README.md)
  -- upstream GNOME Shell theming notes (SCSS -> CSS build process),
  carried over from the vendored source.
* [Gnome js and styling/COPYING](Gnome%20js%20and%20styling/COPYING)
  -- the GPL-2.0-or-later license text that covers the vendored code
  described below.
* [Gnome js and styling/gnome-shell-sass-README.md](Gnome%20js%20and%20styling/gnome-shell-sass-README.md)
  -- upstream README for the `gnome-shell-sass` subtree specifically
  (moved out of `src/frontend/gnome-shell-theme/gnome-shell-sass/`,
  which should hold vendored source only, not docs).
* [Gnome js and styling/gnome-shell-sass.doap](Gnome%20js%20and%20styling/gnome-shell-sass.doap)
  -- upstream project/maintainer metadata for `gnome-shell-sass`, same
  reason as above.
* `Gnome js and styling/gnome-shell-sass-NEWS` -- upstream changelog
  stub for `gnome-shell-sass` (empty upstream, kept for provenance).

---

## What's vendored, and why

`src/frontend/gnome-shell-ui/` and `src/frontend/gnome-shell-theme/`
are an unmodified copy of GNOME Shell's `js/ui/` (GJS) and
`data/theme/` (SCSS/CSS), pulled in from the upstream
[GNOME/gnome-shell](https://github.com/GNOME/gnome-shell) repository.

They're here as **reference source for the Vue 3 frontend rewrite**,
not as code ARE runs as-is. GNOME Shell's UI runtime (GJS + St/Clutter)
is a different environment from ARE's WebKit/HTML/CSS/JS frontend, so
this code won't execute unmodified inside ARE -- the value is in the
UI logic, layout, and interaction patterns it encodes, to be read and
ported deliberately rather than compiled in.

## License

The vendored code is `GPL-2.0-or-later` (see
[COPYING](Gnome%20js%20and%20styling/COPYING)). ARE itself is GPLv3
(see the repo-root [LICENSE](../../LICENSE)) -- `-or-later` is
explicitly compatible with GPLv3, so no license conflict here.

This holds through the Vue 3 rewrite, too: porting or rewriting the
vendored logic still produces a derivative work under GPL, regardless
of how much the code ends up looking like the original. Practically:

* Keep this directory and its license text around as the record of
  provenance, even once `src/frontend/` no longer resembles it.
* Preserve upstream copyright headers in any vendored file that's
  copied wholesale before editing; add ARE's own copyright alongside
  when a file is substantially rewritten, rather than replacing the
  original notice.
* The rewritten frontend stays GPLv3 (or later) -- it can't be
  relicensed more permissively on the basis that it's "just inspired
  by" GNOME Shell once the derivative-work line has been crossed.

## Source

Pulled from `GNOME/gnome-shell` (GitHub mirror) on 2026-09-02:
`js/ui/` -> `src/frontend/gnome-shell-ui/`,
`data/theme/` -> `src/frontend/gnome-shell-theme/`.
