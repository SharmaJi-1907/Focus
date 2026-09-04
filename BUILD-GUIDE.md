# Build guide

How to compile **Floating Focus v7** into installers. If you only want to *use*
the app, you don't need any of this — see [README.md](README.md) → Install.

The Rust toolchain here is a **build-time** tool only. What you ship stays
~5–10 MB and end machines need nothing extra installed.

---

## Option A — let GitHub build it (recommended)

No toolchain on your machine at all. GitHub compiles Linux and Windows in
parallel, for free.

1. Push this repository to GitHub.
2. **Actions** tab → **build installers** → **Run workflow**.
3. Wait ~10 minutes. Download from the run's **Artifacts**.

### Publishing a release

A manual **Run workflow** only builds — the installers land as workflow
artifacts, which expire after 90 days and need a login to download. To publish
a real Release with permanent download links, push a tag:

```bash
git tag v7.0.0
git push origin v7.0.0
```

That triggers the same workflow, but `tagName` is set, so tauri-action creates
the GitHub Release and attaches the `.deb`, `.AppImage`, `.msi` and `.exe`.
Those are the files the README's download links point at, so **the links stay
dead until a tag is pushed.**

Asset filenames carry the version (`Floating.Focus_7.0.0_amd64.AppImage`), so
the direct links in README.md need updating each release. The main download
button points at `/releases/latest` and never needs touching.

The workflow lives at
[`.github/workflows/build.yml`](.github/workflows/build.yml)
and produces:

| Runner | Output |
|---|---|
| `ubuntu-22.04` | `.deb` and `.AppImage` |
| `windows-latest` | `.msi` |

---

## Option B — build locally

### 1. System dependencies

**Ubuntu / Debian:**

```bash
sudo apt update
sudo apt install -y libwebkit2gtk-4.1-dev libayatana-appindicator3-dev \
                    librsvg2-dev patchelf build-essential \
                    curl wget file libssl-dev
```

| Package | Needed for |
|---|---|
| `libwebkit2gtk-4.1-dev` | The webview that renders the UI |
| `libayatana-appindicator3-dev` | The system tray (new in v7 — without it the build fails) |
| `librsvg2-dev`, `patchelf` | AppImage packaging |
| `build-essential`, `libssl-dev` | Compiling and linking |

**Windows:** install [Rust](https://rustup.rs) and *Desktop development with
C++* from the Visual Studio Build Tools. WebView2 already ships with Windows
10 and 11.

### 2. Rust

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
cargo install tauri-cli --locked
```

This installs into `~/.cargo` and needs no root.

### 3. Run and build

```bash
cd floating-focus

cargo tauri dev      # live-reloads while you edit src/index.html
cargo tauri build    # installers land in src-tauri/target/release/bundle/
```

`cargo tauri dev` is the fast loop: edit
[`src/index.html`](floating-focus/src/index.html), save, and the window
refreshes. Rust changes trigger a recompile.

---

## Project layout

```
.
├─ .github/workflows/build.yml      ← cloud build; GitHub only runs workflows
│                                     from the repository root
├─ README.md                        ← download page for users
├─ BUILD-GUIDE.md                   ← this file
└─ floating-focus/
   ├─ src/
   │  └─ index.html                 ← the entire UI: HTML + CSS + JS, one file
   ├─ src-tauri/
   │  ├─ tauri.conf.json            ← window size, frameless, always-on-top
   │  ├─ src/main.rs                ← native shell: tray, hotkeys, window state
   │  ├─ Cargo.toml                 ← Rust dependencies
   │  ├─ build.rs
   │  ├─ capabilities/default.json  ← allowlist of native calls the UI may make
   │  └─ icons/                     ← app icon, every size
```

### The icon files

`floating-focus/src-tauri/icons/` holds every size, and all of them are listed
in the `bundle.icon` array in `tauri.conf.json`:

| File | Used for |
|---|---|
| `32x32.png` | Small window and tray sizes |
| `128x128.png` | Standard app icon |
| `128x128@2x.png` | HiDPI / Retina displays |
| `icon.png` | Master source, largest size |
| `icon.ico` | Windows (`.msi`, Start menu, taskbar) |

The system tray reuses the default window icon at runtime, so it needs no
separate file.

---

## Editing the app

Almost everything is in one file.

**[`floating-focus/src/index.html`](floating-focus/src/index.html)** — the whole
UI, in plain HTML/CSS/JS. No framework, no bundler, no build step.

**[`floating-focus/src-tauri/src/main.rs`](floating-focus/src-tauri/src/main.rs)**
— the native shell: system tray, the global-shortcut plugin, and the
window-state plugin.

**[`floating-focus/src-tauri/tauri.conf.json`](floating-focus/src-tauri/tauri.conf.json)**
— window geometry and behaviour. Two settings matter more than they look:

- `"resizable": true` — **required.** The app resizes its own window when you
  expand or enter Zen mode. On GTK, a window marked non-resizable ignores those
  requests, and the expanded panel gets clipped off-screen.
- No `minWidth` / `minHeight` — Zen mode is 236×124, smaller than the mini
  widget, and a minimum size would clamp it.

Window sizes are defined together near the top of the JS:

```js
const MINI=[300,344], EXPANDED=[352,700], ZEN=[236,124];
```

### Adding a native call

The UI may only make native calls that are explicitly allowed in
[`capabilities/default.json`](floating-focus/src-tauri/capabilities/default.json).
A missing permission fails **silently** — the call simply does nothing. If a new
`window.__TAURI__` call appears to be ignored, add its permission there first.

### Replacing the icon

Drop a square PNG of 512×512 or larger at
`floating-focus/src-tauri/icons/icon.png`, then:

```bash
cd floating-focus
cargo tauri icon
```

Every size regenerates in place, including the Windows `.ico`. Rebuild after.

---

## Troubleshooting

**`failed to run custom build command for tauri` / `ayatana-appindicator3-0.1 not found`**
The tray dependency is missing. Install `libayatana-appindicator3-dev` and
rebuild.

**The expanded panel is cut off and won't scroll**
`"resizable"` is `false` in `tauri.conf.json`. It must be `true`.

**`Ctrl`+`Alt`+`F` does nothing**
You're on Wayland, which does not permit global key grabs. Expand the panel and
check the **Shortcut** row — it reports the real state. This is a desktop
restriction, not a bug in the app.

**No tray icon on GNOME**
Install and enable `gnome-shell-extension-appindicator`. Until then avoid ✕,
which hides the window; use — (minimize) instead.

**Black corners instead of rounded ones**
Your window manager isn't compositing. Every mainstream desktop composites by
default.

**Verifying a change without building**
The UI runs in an ordinary browser — `window.__TAURI__` is absent and every
native call is skipped safely:

```bash
cd floating-focus/src && python3 -m http.server 8778
```

Open <http://127.0.0.1:8778>. You get the full interface, timer, tasks and
Today panel. You do **not** get window resizing, dragging, the tray, or the
hotkey — those need a real build.
