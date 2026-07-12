# Floating Focus v6 — tiny native hover widget (Tauri)

A real desktop application that **hovers above every app and browser**, with a
📌 **pin** button to toggle always-on-top, **native OS dragging** (grab the top
bar or the big digits), and the same UI/features as v5. Tasks and settings save
locally and survive restarts. Nothing leaves your machine.

Why it's small and instant: Tauri uses the webview already inside your OS
(WebKitGTK on Linux, WebView2 on Windows) instead of bundling a browser like
PyQt/Electron. The finished app is roughly **5–10 MB** and opens in under a
second. No Python, no venv.

```
floating-focus/
├─ src/index.html            ← the whole UI (edit this to add features)
├─ src-tauri/
│  ├─ tauri.conf.json        ← window: frameless, on-top, 300×285 mini
│  ├─ src/main.rs            ← 8-line native shell
│  ├─ Cargo.toml
│  ├─ capabilities/default.json  ← window permissions (pin, drag, resize…)
│  └─ icons/
└─ .github/workflows/build.yml   ← cloud build (free, via GitHub Actions)
```

---

## Option A — build in the cloud (recommended, nothing to install)

GitHub compiles the installers for **both Linux and Windows** for free:

1. Create a GitHub repo and push this folder to it.
2. Go to the repo's **Actions** tab → **build installers** → **Run workflow**
   (or push a tag like `v6.0.0`).
3. ~10 minutes later, download from the workflow run / Releases page:
   - Linux: `.deb` (Ubuntu/Debian) and `.AppImage` (any distro, just `chmod +x` and run)
   - Windows: `.msi` installer

Install once per machine; from then on it launches instantly from your app
menu / Start menu.

## Option B — build locally (one-time developer setup)

Linux (Ubuntu/Debian):
```bash
sudo apt install libwebkit2gtk-4.1-dev build-essential curl wget file \
                 libssl-dev libappindicator3-dev librsvg2-dev
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
cargo install tauri-cli --locked
cd floating-focus
cargo tauri dev      # run it now, with live reload while editing src/index.html
cargo tauri build    # produce .deb / .AppImage in src-tauri/target/release/bundle/
```

Windows: install [Rust](https://rustup.rs) + "Desktop development with C++"
from Visual Studio Build Tools (WebView2 is already part of Windows 10/11),
then the same `cargo tauri dev` / `cargo tauri build`.

Note: the Rust toolchain is a *build* tool — the app you ship stays ~5–10 MB
and end machines need nothing else. If you don't want the toolchain at all,
use Option A and let GitHub's servers do the compiling.

---

## Controls

| Button | Action |
|---|---|
| 📌 | pin / unpin — pinned means it hovers above **every** application |
| ☾ / ☀ | dark / light theme |
| — | minimize to taskbar |
| ⤢ / ⤡ | expand (tasks, timer setup) ⇄ mini widget |
| ✕ | close |
| top bar / big digits | **drag anywhere on screen** (handled natively by the OS — smooth) |
| double-click digits | start / pause |

## Adding features later

Everything visible is in `src/index.html` — plain HTML/CSS/JS, same structure
as your v5 file. Window superpowers come from `window.__TAURI__.window`
(`setAlwaysOnTop`, `setSize`, `startDragging`, `minimize`, …). If a new
window API call is blocked, add its permission in
`src-tauri/capabilities/default.json`.

Cloud sync can be added later without touching the Rust side — it's the same
JS-only change as a web app (GitHub Gist or Supabase), since data is just the
JSON blob this app already saves.
