# Problem Statement

## Linux Android-Like Launcher

Linux desktop environments are designed around traditional desktop interaction models such as panels, taskbars, application menus, windows, and overlapping workspaces. While these interfaces are powerful, they do not provide the simple, touch-friendly, application-centric experience commonly found on mobile operating systems such as Android.

There is a need for a lightweight Linux launcher that provides an Android-inspired user experience while retaining the flexibility and openness of the Linux desktop.

The proposed project is a custom launcher environment built using **Python, GNOME Kiosk, WebKit/WebView, HTML, CSS, and JavaScript**. It will act as a graphical shell through which users can discover and launch applications, interact with system functionality, and customize their desktop environment.

## Core Problem

Existing Linux desktop environments generally separate application launching, system controls, widgets, search, and desktop customization across multiple components.

The project aims to provide these capabilities through a unified launcher interface:

* Application grid and application discovery
* Application search
* Favorites and customizable launcher layout
* Desktop widgets
* Wallpaper management
* System information
* Network and audio controls
* Power-management actions
* Keyboard, mouse, and potentially touch interaction
* Full-screen or kiosk-style operation

The launcher should feel more like a mobile home screen than a conventional Linux desktop while continuing to launch and interact with native Linux applications.

## Technical Problem

The project must bridge two fundamentally different environments:

1. **Web technologies**, which provide flexible and efficient UI development through HTML, CSS, and JavaScript.
2. **Native Linux/GNOME functionality**, which requires access to application metadata, system services, D-Bus interfaces, filesystem resources, and desktop APIs.

The architecture therefore requires a controlled communication layer between the JavaScript frontend and the Python backend.

```text
┌─────────────────────────────┐
│       HTML / CSS / JS       │
│                             │
│  Launcher UI                │
│  App Grid                   │
│  Search                     │
│  Widgets                    │
│  Settings                   │
└──────────────┬──────────────┘
               │
          JS ↔ Python
             Bridge
               │
┌──────────────▼──────────────┐
│          Python             │
│                             │
│ Application discovery       │
│ Application launching       │
│ System integration          │
│ D-Bus communication         │
│ Configuration               │
└──────────────┬──────────────┘
               │
┌──────────────▼──────────────┐
│        Linux / GNOME        │
│                             │
│ .desktop files              │
│ D-Bus services              │
│ GIO / system APIs           │
│ Native applications         │
└─────────────────────────────┘
```

## Goals

The project should:

* Provide an Android-inspired launcher experience on Linux.
* Use web technologies for the primary user interface.
* Use Python for native Linux integration.
* Discover installed applications through standard Linux application metadata.
* Launch native applications without requiring modifications to those applications.
* Integrate with GNOME and Linux system services where practical.
* Keep the JavaScript-to-Python bridge small, explicit, and secure.
* Maintain responsive UI interactions by keeping high-frequency UI operations inside the frontend.
* Support keyboard and mouse input, with the architecture allowing future touch support.
* Store launcher configuration independently from system application data.
* Operate in a GNOME Kiosk/full-screen environment when configured as the primary shell.

## Non-Goals

The initial project will not attempt to:

* Replace the Linux kernel.
* Implement its own window manager.
* Rewrite existing Linux applications.
* Reimplement the entire GNOME desktop environment.
* Execute arbitrary shell commands directly from JavaScript.
* Provide a complete mobile operating system.
* Replace Linux's existing application packaging systems.

## Expected Outcome

The final system should provide a Linux environment where the primary interaction model is a customizable launcher resembling a mobile home screen.

A typical interaction should look like:

```text
User
 │
 ├── Opens launcher
 │
 ├── Searches for application
 │
 ├── Selects application
 │
 ▼
JavaScript UI
 │
 ▼
Python bridge
 │
 ▼
Linux application launcher
 │
 ▼
Native Linux application
```

The result should combine the **UI flexibility of web technologies**, the **system-level capabilities of Python**, and the **native application ecosystem of Linux** into a single launcher-oriented desktop experience.
