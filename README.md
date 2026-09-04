<img src="floating-focus/src-tauri/icons/128x128.png" width="88" align="left" alt="Floating Focus icon">

# Floating Focus

**A tiny always-on-top focus timer that hovers over every window on your screen.**
Pomodoro rhythm, a progress ring, your task list, and a daily streak — in a
widget the size of a coaster. Everything stays on your machine.

<br clear="left">

---

## Why it exists

You lose focus in the gap between *deciding* to work and *actually* starting.
Floating Focus removes that gap: the timer is always visible, above the browser,
above the editor, above everything. No tab to find, no app to switch to.

It is a real native desktop app, not a web page. It uses the webview already
built into your OS (WebKitGTK on Linux, WebView2 on Windows) instead of shipping
a whole browser like Electron would, so the finished app is **~5–10 MB** and
opens in well under a second. No Python, no Node, no runtime to install.

---

## Features

**Timer**
- Drift-free countdown — computed from an absolute end-time, so it never lags
- Progress ring that drains as the session burns down
- Set any length from 1 minute to 24 hours

**Pomodoro rhythm**
- **Focus / Break / Long** modes, switchable by pill
- `●●●○` dots track where you are in the four-session cycle
- After four focus sessions you get the long break automatically
- Optional **auto-start next** — the cycle runs itself, hands-free
- Break lengths are yours to set

**Your work**
- Task list with 12 icons and 6 colours per task
- Optional schedule note per task (`6–7 pm`)
- Mark done, delete, reorder by selecting
- The active task shows inside the ring while you work

**Today**
- Minutes focused today, broken down per task
- 🔥 Day streak, counted from your session log
- All-time session count

**Staying out of the way**
- 📌 **Pin** — hover above every application, or drop back into the stack
- ◐ **Zen mode** — shrinks to a translucent countdown; hover to reveal controls
- **System tray** — ✕ hides the window and *keeps the timer running*; reopen or
  quit from the tray icon
- Remembers where you left the window
- Dark and light themes
- Three finish sounds (Chime / Bell / Beep) with a volume slider

---

## Install

### Option A — download a build (easiest)

Grab the latest from the repository's **Releases** page:

| Platform | File |
|---|---|
| Ubuntu / Debian | `.deb` — double-click, or `sudo dpkg -i "Floating Focus_7.0.0_amd64.deb"` |
| Any Linux | `.AppImage` — `chmod +x` it and run |
| Windows 10/11 | `.msi` installer |

Install once; from then on it launches from your app menu or Start menu with its
own icon.

### Option B — build in the cloud (nothing to install)

GitHub compiles both platforms for you, for free:

1. Push this repository to GitHub.
2. **Actions** → **build installers** → **Run workflow**.
   Or push a tag: `git tag v7.0.0 && git push --tags`.
3. About ten minutes later, download the installers from the workflow run
   (or from the Release, if you pushed a tag).

### Option C — build it yourself

See **[BUILD-GUIDE.md](BUILD-GUIDE.md)** for the full local setup, dependencies
and troubleshooting.

---

## Using it

### The top bar

| Button | What it does |
|:---:|---|
| ◐ | **Zen mode** — tiny translucent countdown; hover to reveal ▶ / ⤢ |
| 📌 | **Pin** — toggle always-on-top. Lit amber means it hovers over everything |
| ☾ / ☀ | Dark / light theme |
| — | Minimize to the taskbar |
| ⤢ / ⤡ | Expand to the full panel ⇄ back to the mini widget |
| ✕ | **Hide to tray** — the timer keeps running. Quit from the tray menu |

### Everywhere else

| Action | Result |
|---|---|
| Drag the top bar or the ring | Move the window (handled natively by the OS) |
| Double-click the digits | Start / pause |
| Drag anywhere in Zen mode | Move the window |
| `Ctrl` + `Alt` + `F` | Start / pause from inside any application¹ |
| Tray icon | Show / Hide, or Quit |

¹ See [Known limits](#known-limits) — this does not work on Wayland.

### A normal session

1. Hit **⤢** to expand.
2. Add what you're avoiding under **New task**, pick an icon and a colour.
3. Set your **Focus length** (25 minutes is the default) and your break lengths.
4. Collapse back with **⤡**, then **▶ Start** — or just double-click the digits.
5. At zero: a sound plays, **DONE** flashes, the session is logged against your
   task, and the next mode begins.
6. Check **Today** whenever you want to see where the hours went.

---

## Where your data lives

Everything — tasks, settings, session log, streak — is stored in the app's own
local webview storage under the key `ff6`. It never leaves your machine. There
is no account, no sync, no telemetry, no network call of any kind.

Upgrading from v6 keeps your existing tasks and settings; the storage key is
deliberately unchanged.

---

## Project layout

```
.
├─ README.md                        ← you are here
├─ BUILD-GUIDE.md                   ← how to compile it yourself
└─ floating-focus/
   ├─ src/
   │  └─ index.html                 ← the entire UI: HTML + CSS + JS, one file
   └─ src-tauri/
      ├─ tauri.conf.json            ← window size, frameless, always-on-top
      ├─ src/main.rs                ← native shell: tray, hotkeys, window state
      ├─ Cargo.toml                 ← Rust dependencies
      ├─ build.rs
      ├─ capabilities/default.json  ← which native calls the UI may make
      └─ icons/                     ← app icon, every size
```

Two files hold almost everything:

- **[`src/index.html`](floating-focus/src/index.html)** — the whole interface.
  Plain HTML, CSS and JavaScript, no build step, no framework. Edit it and
  re-run; that's the entire front-end workflow.
- **[`src-tauri/src/main.rs`](floating-focus/src-tauri/src/main.rs)** — the
  native shell. Registers the system tray, the global shortcut plugin, and the
  window-state plugin that remembers your window position.

Window superpowers come from `window.__TAURI__.window` — `setAlwaysOnTop`,
`setSize`, `startDragging`, `minimize`, `hide`. **If a new window call silently
does nothing, its permission is missing** from
[`capabilities/default.json`](floating-focus/src-tauri/capabilities/default.json).
That file is an allowlist; add the permission there.

---

## The icon

<img src="floating-focus/src-tauri/icons/128x128.png" width="64" align="left" alt="icon">

A dark rounded square holding an amber progress arc — the same ring you watch
while a session runs — with a green check badge for the work you finished. It is
the app's identity everywhere: the window, your taskbar, the app menu, the
Start menu, and the system tray.

<br clear="left">

The source files live in
[`floating-focus/src-tauri/icons/`](floating-focus/src-tauri/icons/):

| File | Used for |
|---|---|
| `32x32.png` | Small window and tray sizes |
| `128x128.png` | Standard app icon |
| `128x128@2x.png` | HiDPI / Retina displays |
| `icon.png` | Master source, largest size |
| `icon.ico` | Windows (`.msi`, Start menu, taskbar) |

Every one of these is listed in the `bundle.icon` array in
[`tauri.conf.json`](floating-focus/src-tauri/tauri.conf.json), and the tray
reuses the default window icon at runtime.

**To use your own:** drop a square PNG of at least 512×512 at
`floating-focus/src-tauri/icons/icon.png` and run `cargo tauri icon` — it
regenerates every size, including the Windows `.ico`, in place. Then rebuild.

---

## Known limits

**`Ctrl`+`Alt`+`F` does not work on Wayland.** Global shortcuts need X11-style
key grabs, which Wayland deliberately does not allow any application to take.
The app detects this and tells you: expand the panel and read the **Shortcut**
row, which reports either *active* or *blocked by the desktop*. On a Wayland
desktop, use the tray or the window instead. On X11 and on Windows the shortcut
works normally.

**The tray icon needs an indicator extension on GNOME.** Ubuntu ships
`gnome-shell-extension-appindicator` enabled by default, so this is usually
fine. If no tray icon appears, do not use ✕ — it hides the window, and without a
tray you would have no way to bring it back. Use — (minimize) instead until the
extension is installed.

**Transparency needs a compositor.** Zen mode's translucency and the rounded
corners rely on one. Every modern desktop has one; on a bare window manager
without compositing you may see square black corners instead.

---

## Version history

**v7** — progress ring · Pomodoro cycle with auto-start · per-task session
logging · Today panel with day streak · three sounds with volume · Zen mode ·
system tray with hide-instead-of-quit · global hotkey · remembered window
position · rounded window

**v6** — the original Tauri rewrite: frameless always-on-top widget, pin button,
native dragging, tasks, themes

---

## License and privacy

Your data is yours and stays on your disk. The app makes no network requests.
