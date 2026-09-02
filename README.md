# ARK Runtime Environment (ARE)

A Linux desktop environment inspired by the simplicity of modern mobile interfaces.

ARE is a launcher-oriented runtime environment for Linux, designed around a simple idea: the desktop should feel like a place you use, not a collection of things you have to manage.

Built with **Python, WebKit/WebView, HTML, CSS, JavaScript, and GNOME Kiosk**, ARE brings applications, search, widgets, system controls, and customization into a single interface.

---

## A different way to use Linux

Traditional Linux desktops expose the machinery of the system.

ARE puts the experience first.

Applications are presented as a unified collection. Search is always within reach. System controls live alongside the rest of the environment. The interface is designed to stay out of the way until it is needed.

```text
                    ARE
                     │
       ┌─────────────┼─────────────┐
       │             │             │
   Applications    Search       Widgets
       │             │             │
       └─────────────┼─────────────┘
                     │
              Linux / GNOME
```

The underlying system remains Linux. ARE simply provides a different way to interact with it.

## Designed for Linux

ARE does not attempt to replace the Linux application ecosystem.

It works with it.

Installed applications are discovered through standard Linux application metadata, while native system functionality is accessed through the appropriate Linux and GNOME interfaces.

This means existing applications remain existing applications.

No special ARE version.
No new application format.
No artificial ecosystem.

Just Linux applications, presented differently.

## Web technologies. Native capabilities.

The interface is built using technologies that make rapid iteration possible.

**HTML** defines the structure.
**CSS** defines the visual system.
**JavaScript** handles interaction and application state.
**Python** connects the interface to Linux.

The boundary between them is deliberately small.

```text
┌──────────────────────────────────────┐
│              ARE UI                  │
│                                      │
│       HTML · CSS · JavaScript        │
└──────────────────┬───────────────────┘
                   │
              Native Bridge
                   │
┌──────────────────▼───────────────────┐
│             ARE Runtime              │
│                                      │
│              Python                  │
└──────────────────┬───────────────────┘
                   │
          Linux · GNOME · D-Bus
```

The UI handles the things a UI should handle.

The runtime handles the things the operating system should handle.

## What ARE provides

* Application discovery and launching
* Application search
* Favorites and launcher organization
* Widgets
* Wallpaper and appearance configuration
* System information
* Network and audio controls
* Power controls
* Keyboard and pointer interaction
* A full-screen, kiosk-oriented experience
* A foundation for future touch interaction

The goal is not to reproduce another operating system.

The goal is to create an environment that feels coherent on Linux.

---

## Project structure

The repository keeps the user-facing project description separate from its engineering documentation.

```text
ARE/
├── README.md
│
├── doc/
│   ├── problem-statement.md
│   ├── architecture.md
│   ├── requirements.md
│   └── design.md
│
├── src/
│   ├── backend/
│   ├── bridge/
│   └── frontend/
│
├── tests/
│
└── ...
```

`README.md` describes what ARE is.

The `doc/` directory describes how it is built and why.

---

## Status

ARE is an early-stage project.

The architecture, interface, and runtime APIs are expected to evolve as the project develops.

The current priority is establishing a clean runtime foundation and a responsive launcher experience before expanding into deeper desktop integration.

---

## License

See [LICENSE](LICENSE) for licensing information.
