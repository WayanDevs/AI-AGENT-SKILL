---
name: tauri-troubleshooting
description: Setup, build, bundle, and troubleshoot Tauri v2 desktop applications on Linux. Covers project structure, migration from web-only projects, .deb packaging, and common compilation fixes.
version: 2.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [Tauri, Desktop, Rust, Node, Build, Packaging]
---

# Tauri v2 — Setup, Build & Troubleshooting

Trigger: user wants to create, migrate, build, package, or debug a Tauri v2 desktop application.

## Tauri v2 Project Structure (Required Files)

```
project-root/
├── index.html              # Entry point (loaded by WebView)
├── package.json            # Must have @tauri-apps/cli, "tauri" script
├── vite.config.ts          # Frontend bundler config
├── tsconfig.json
├── src/                    # Frontend source (TS/JS/CSS)
│   ├── main.ts
│   └── index.css
└── src-tauri/
    ├── Cargo.toml          # edition="2021", tauri v2, tauri-build v2
    ├── build.rs            # Just: tauri_build::build()
    ├── tauri.conf.json     # App config, bundle config, build commands
    ├── capabilities/
    │   └── default.json    # Permissions (core:default minimum)
    ├── icons/              # Generated via `npx tauri icon`
    │   ├── 32x32.png
    │   ├── 128x128.png
    │   ├── 128x128@2x.png
    │   ├── icon.icns
    │   └── icon.ico
    └── src/
        └── main.rs         # Rust backend
```

## Setup / Migration Workflow

### 1. Install Tauri CLI
```bash
npm install --save-dev @tauri-apps/cli@latest
npx tauri --version  # verify
```

### 2. package.json — Required Shape
- Scripts MUST include: `"dev": "vite"`, `"build": "vite build"`, `"tauri": "tauri"`
- Remove server-side deps (express, better-sqlite3, multer, etc.) — Tauri backend is Rust
- Keep only frontend deps (tailwindcss, vite, etc.)

### 3. vite.config.ts — Tauri-Compatible Config
Key settings for Tauri dev:
```ts
const host = process.env.TAURI_DEV_HOST;
export default defineConfig({
  clearScreen: false,
  server: {
    port: 3000,       // Must match tauri.conf.json devUrl
    strictPort: true,
    host: host || false,
    hmr: host ? { protocol: 'ws', host, port: 3001 } : undefined,
    watch: { ignored: ['**/src-tauri/**'] },
  },
});
```

### 4. tauri.conf.json — Key Fields
```json
{
  "build": {
    "beforeDevCommand": "npm run dev",
    "beforeBuildCommand": "npm run build",
    "frontendDist": "../dist",
    "devUrl": "http://localhost:3000"
  },
  "bundle": {
    "active": true,
    "targets": ["deb"],
    "icon": ["icons/32x32.png", "icons/128x128.png", ...]
  }
}
```
- **Pitfall:** If project was cloned with `pnpm` commands, change to `npm run ...`
- **targets**: Use `["deb"]` for Linux .deb only, `"all"` for everything

### 5. Generate Icons (MUST do before build)
```bash
# Create or provide a 1024x1024 PNG source icon, then:
npx tauri icon app-icon.png
```
If no source icon exists, generate one programmatically with Python PIL:
```python
from PIL import Image, ImageDraw
img = Image.new('RGBA', (1024, 1024), (0,0,0,0))
draw = ImageDraw.Draw(img)
# Draw your design...
img.save('app-icon.png')
```
Then run `npx tauri icon app-icon.png`.

### 6. Capabilities — default.json
Minimal working config:
```json
{
  "identifier": "default",
  "description": "Default permissions",
  "windows": ["main"],
  "permissions": ["core:default"]
}
```
- **Pitfall:** Permissions like `shell:allow-spawn`, `dialog:all`, `fs:allow-read` require their respective Tauri plugins to be installed in Cargo.toml. If the Rust code doesn't use them, remove them.
- See `references/tauri-v2-capabilities.md` for the plugin-to-permission mapping.
- See `references/fetch-to-invoke-migration.md` for concrete fetch→invoke patterns, two-phase UX (cek link → download), and event-driven logging.
- See `references/linux-crash-diagnosis.md` for step-by-step Linux desktop app crash forensics (journalctl, Apport, dmesg).

### 7. Build .deb
```bash
npx tauri build
# Output: src-tauri/target/release/bundle/deb/<ProductName>_<version>_amd64.deb
```

### 8. Install .deb
```bash
sudo dpkg -i "path/to/package.deb"
```
- **Pitfall:** Agent sessions often lack interactive sudo. Options:
  - Use `pty=true` on the terminal call — this enables interactive password prompts in Hermes desktop GUI sessions
  - Ask user to run the `sudo dpkg -i` command manually in their terminal
  - The bare binary at `src-tauri/target/release/<binary-name>` can be run directly without installing

## Frontend IPC Migration: fetch('/api/...') → invoke()

When migrating a web app (Express/Node backend) to Tauri, the frontend's `fetch('/api/...')` calls MUST be rewritten to use Tauri's IPC `invoke()`. This is the #1 cause of "feature doesn't work" after migration — the UI looks fine but every action fails silently or shows generic errors.

### 1. Enable `withGlobalTauri` in tauri.conf.json
```json
{
  "app": {
    "withGlobalTauri": true,
    ...
  }
}
```
Without this, `window.__TAURI__` is undefined and all invoke calls fail.

### 2. Access invoke and listen in frontend TS
```ts
const { invoke } = (window as any).__TAURI__.core;
const { listen } = (window as any).__TAURI__.event;
```

### 3. Replace every fetch() with invoke()
```ts
// BEFORE (Express pattern — broken in Tauri):
const res = await fetch('/api/analyze', { method: 'POST', body: JSON.stringify({ url }) });
const data = await res.json();

// AFTER (Tauri IPC):
const data = await invoke('analyze_url', { url });
```

Key differences:
- `invoke()` returns the result directly (no `.json()` needed)
- Errors come as rejected promises with string messages (not HTTP status codes)
- Parameter names in `invoke('cmd', { param })` must match Rust function signatures exactly (e.g. if the Rust parameter is named `file_type: String`, the JS call must pass `file_type: extSelected`. Tauri does NOT perform automatic case conversion for bare command arguments).
- Rust `#[tauri::command]` functions returning `Result<T, String>` map to resolved/rejected promises

### 4. Real-time events: use Tauri event system
```ts
// Rust backend emits:
let _ = app.emit("download-progress", dl.clone());

// Frontend listens:
listen('download-progress', (event: any) => {
  updateUI(event.payload);
});
```
Replace any polling of `/api/status` endpoints with Tauri events + periodic `invoke()` polling as fallback.

### 5. Common migration checklist
- [ ] All `fetch('/api/...')` calls replaced with `invoke('command_name', { params })`
- [ ] Each frontend invoke has a matching `#[tauri::command]` in main.rs
- [ ] All commands registered in `.invoke_handler(tauri::generate_handler![...])`
- [ ] `withGlobalTauri: true` set in tauri.conf.json
- [ ] Error handling: `try/catch` around invoke, display error string to user
- [ ] Progress/status updates use `app.emit()` + `listen()` pattern
- [ ] Remove any Express/Node server startup code from frontend

### 6. Pitfall: URL parameters with special chars (playlist URLs)
YouTube URLs with `&list=...` or other query params work fine through invoke — no URL encoding needed since they're passed as string arguments, not in HTTP query strings. The old fetch-based pattern often broke on these because of improper URL construction.

### 7. Pitfall: YouTube URL ambiguity — `watch?v=XXX&list=YYY`
A common YouTube URL pattern is a single video inside a playlist: `https://www.youtube.com/watch?v=ID&list=PLID`. 

Depending on application requirements:
1. **Prioritize single video**: Treat it as a single video because it has a specific video ID.
2. **Prioritize playlist (Recommended for downloaders)**: Treat it as a playlist because the user wants to load the entire list, extract items, or choose what to download.
If prioritizing playlists, simply check if `list=` or `/playlist` is in the URL:
```rust
let is_playlist = url.contains("list=") || url.contains("/playlist");
```

### 8. GTK/WebKit Dropdown Option Invisibility
On Linux Tauri (WebKitGTK), styling a `<select>` element or its `<option>` tags individually can fail because WebKitGTK renders native selects using the host's GTK theme by default, leading to white-on-white or unreadable (invisible text) options.

**Fix:** Disable native styling via `appearance: none`, style the background and text color explicitly, and use a custom SVG background-image for the dropdown arrow to keep the layout correct:
```css
select {
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
  background-color: #020617 !important; /* Latar belakang gelap */
  color: #e2e8f0 !important;             /* Warna teks terang */
  padding-right: 2rem !important;        /* Beri jarak untuk panah */
  background-image: url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' fill='none' viewBox='0 0 20 20'%3e%3cpath stroke='%2394a3b8' stroke-linecap='round' stroke-linejoin='round' stroke-width='1.5' d='M6 8l4 4 4-4'/%3e%3c/svg%3e") !important;
  background-position: right 0.75rem center !important;
  background-repeat: no-repeat !important;
  background-size: 1.25em 1.25em !important;
}
select option {
  background-color: #0c1222 !important; /* Warna gelap menu opsi */
  color: #ffffff !important;
}
```

### 9. UX Pattern: Inline progress and Preventing UI Freezes ("Not Responding")
For operations that take time (link checking, downloads), show progress clearly and prevent the UI thread from locking up:
1. **Preventing "Not Responding" Freezes**: 
   - Never run blocking operations synchronously on the Rust main/Tauri thread; always spawn tasks via `tokio::spawn`.
   - **Tokio Reactor Panic in Sync Commands (SIGABRT / app closes instantly)**: If a Tauri command is synchronous (`fn my_command`), calling `tokio::spawn` or executing async operations within it will crash the entire application with a `there is no reactor running, must be called from the context of a Tokio 1.x runtime` panic. This manifests as the app window closing instantly with no visible error — the panic message is only visible in `journalctl --user` or `/var/crash/` (see "Crash Diagnosis" section below). **Fix:** Always declare the Tauri command as an async function (`async fn my_command`) to ensure it executes inside the Tokio runtime context. Check ALL `#[tauri::command]` functions that use `tokio::spawn` — if any are `fn` instead of `async fn`, they will crash.
   - On the frontend, avoid synchronous blocking operations or tight loops. When invoking commands sequentially (e.g., in a loop over playlist tracks), ensure you don't block the browser's paint cycles.
   - **Tauri Command Param Deserialization**: In Tauri commands, argument names defined in Rust as snake_case (e.g. `file_type: String`) are automatically converted to camelCase (e.g. `fileType`) in the JavaScript glue layer. Always invoke with camelCase keys (e.g., `invoke('add_download', { fileType: 'mp4' })`). Passing a snake_case key directly in JS will cause Tauri to throw a `missing required key` deserialization error.
2. **Inline Step-by-Step Progress**: Show exact sub-states directly under the form or input field (e.g. `1. Cek URL...`, `2. Menghubungkan ke API...`, `3. Mengunduh Metadata...`) before rendering the final results or preview. This provides immediate visual feedback if a process gets stuck at a specific step.
3. **CLI terminal logs**: emit step-by-step logs from Rust with emoji markers (▶ start, ⟳ in-progress, ✓ done, ✗ error).
4. **Optimizing Log Polling**: To prevent browser layout reflows and flickering (which looks like jumping or continuous auto-scrolling), do not use `setInterval` to periodically poll and clear the terminal logs view. Instead, rely on Tauri's event emission listener (`listen('log-updated')`) to trigger redraws only when new log events are dispatched, and set `terminal.scrollTop = terminal.scrollHeight` once at the very end of the redraw function rather than after appending each log item.

If errors occur on the frontend (validation failure) or inside a catch block, expose a `write_log` Tauri command to report them directly to the backend's shared log queue:
```rust
#[tauri::command]
fn write_log(state: State<'_, AppState>, app: AppHandle, level: String, message: String) -> Result<(), String> {
    push_log(&state.logs, &level, &message);
    let _ = app.emit("log-updated", ());
    Ok(())
}
```
In frontend TS:
```ts
async function logToTerminal(level: 'INFO' | 'WARN' | 'ERROR', message: string) {
  await invoke('write_log', { level, message });
}
```
This ensures frontend errors and catch-block stacktraces appear inside the CLI logs component.

### 10. Extracting YouTube/Media Metadata without yt-dlp binary
If `yt-dlp` is not installed or too heavy to bundle, you can parse public YouTube metadata dynamically by requesting the video/playlist HTML using Python's standard `urllib` and extracting the `ytInitialData` JSON block.

To make this parsing robust:
1. **URL Rewriting for Playlists**: When processing playlist watch URLs (e.g. `https://www.youtube.com/watch?v=XXX&list=YYY`), rewrite the URL to a clean playlist browse URL (`https://www.youtube.com/playlist?list=YYY`) first. YouTube watch pages have a different HTML layout that lacks the standard playlist JSON structure.
2. **Length-Based Video ID Filtering**: When extracting video IDs (`contentId`) from `lockupViewModel` items, ensure you check `len(vid_id) == 11`. YouTube mix pages and recommended section lockups can return playlist IDs (e.g. starting with `VL`) or mix IDs (e.g. starting with `RD`) which will cause errors down the road if treated as a video ID.
3. **Capture stdout and stderr**: Run a python script from Rust using `std::process::Command`, but always capture and inspect both standard output and standard error:
   ```rust
   let output = std::process::Command::new("python3")
       .arg("path/to/extract_youtube.py")
       .arg(&url)
       .output();

   match output {
       Ok(out) => {
           let stdout_str = String::from_utf8_lossy(&out.stdout).trim().to_string();
           let stderr_str = String::from_utf8_lossy(&out.stderr).trim().to_string();

           if !stderr_str.is_empty() {
               push_log(&state.logs, "WARN", &format!("[cek-link] Python stderr: {}", stderr_str));
           }
           // parse stdout_str...
       }
       Err(e) => { ... }
   }
   ```
4. **HTML Parsing**: In the python script, search the HTML for `ytInitialData = ` and count matching braces to extract the raw JSON. In the new YouTube layout, search for `lockupViewModel` under tabs contents to retrieve playlist video metadata (`contentId`, title under `metadata` / `rendererContext` / `accessibilityContext`, and duration under `overlays`).

### 11. Download Queue UX: Auto-Clear Completed Items
When a download finishes (status = `"completed"`), the active queue badge and card list should reflect this immediately:

1. **Badge count**: Filter to only count truly active items:
   ```typescript
   const activeCount = items.filter(i => i.status === 'downloading' || i.status === 'queued').length;
   ```
   NOT `items.length` — that includes completed/failed items and makes the badge appear "stuck".

2. **Auto-remove from backend state**: After saving to history DB and emitting the final progress event, remove the item from `state.downloads` (or schedule removal after a short delay so the frontend can show the completion animation):
   ```rust
   // After INSERT INTO history and emitting final progress:
   state_clone.lock().unwrap().remove(&id_clone);
   ```

3. **Cancel button on completed cards**: The "X" button should call `control_download(id, "delete")` (not `"cancel"`) for completed items so they're removed from the queue map.

4. **Log the actual file path**: Always construct `path_file` BEFORE logging it. A common mistake is logging `path_file.to_string_lossy()` before the variable is declared, causing a compile error (`cannot find value path_file in this scope`). Pattern:
   ```rust
   // 1. Build path and write file
   let mut path_file = dirs::download_dir().unwrap_or_default();
   path_file.push("YibYib_Media");
   let _ = std::fs::create_dir_all(&path_file);
   let filename = format!("{}.{}", final_title, ft_clone);
   path_file.push(&filename);
   let _ = std::fs::write(&path_file, content);
   // 2. THEN log it
   push_log(&logs, "INFO", &format!("✓ Tersimpan: {}", path_file.to_string_lossy()));
   ```

5. **Hide speed/stats on completed cards**: When rendering a download card with `status === 'completed'`, hide the "Kecepatan" (speed) line since it's no longer relevant.

## Crash Diagnosis for Linux Desktop Apps

When a Tauri app closes itself instantly without any visible error:

### Step 1: Check systemd user journal (most reliable)
```bash
journalctl --user -b -0 --no-pager | grep -i <app-name>
```
This captures Rust panic messages, SIGABRT traces, and stack backtraces that the GUI launcher swallows silently.

### Step 2: Check Apport crash reports
```bash
ls /var/crash/*<app-name>*
grep -i "Signal:" /var/crash/_usr_bin_<app-name>.*.crash
```
Signal 6 = SIGABRT = Rust panic. Signal 11 = SIGSEGV = memory corruption.

### Step 3: Run binary from terminal with RUST_BACKTRACE
```bash
RUST_BACKTRACE=1 /usr/bin/<app-binary> 2>&1 | tee /tmp/crash.log
```
If the app needs a display: `DISPLAY=:0 RUST_BACKTRACE=1 /usr/bin/<app-binary> 2>&1 | tee /tmp/crash.log`

### Common crash signatures
| Panic message | Root cause | Fix |
|---|---|---|
| `there is no reactor running, must be called from the context of a Tokio 1.x runtime` | `tokio::spawn()` called from a sync `fn` Tauri command | Change `fn cmd()` → `async fn cmd()` |
| `called unwrap() on a poisoned Mutex` | A previous thread panicked while holding the lock | Replace `.unwrap()` with `.lock().unwrap_or_else(\|e\| e.into_inner())` |
| `thread caused non-unwinding panic. aborting.` | Double-panic (panic inside a panic handler or FFI boundary) | Fix the original panic first |

## Common Troubleshooting

### 1. Proc Macro Panic: Failed to open icon
- **Error:** `proc macro panicked ... failed to open icon src-tauri/icons/32x32.png: No such file or directory`
- **Fix:** Run `npx tauri icon <source.png>` to generate the full icon set

### 2. Capabilities Permission Not Found
- **Error:** `Permission shell:allow-spawn not found, expected one of core:default...`
- **Cause:** Permission references a Tauri plugin not installed in Cargo.toml
- **Fix:** Either add the plugin crate to Cargo.toml, or remove the permission from capabilities/default.json

### 3. Rust Compiler: `add_log` / State Borrow Issues
- **Pattern:** Helper functions that take `&State<AppState>` can cause borrow conflicts when the state's inner Mutex is already locked
- **Fix:** Change helper signature to take the specific `Arc<Mutex<...>>` field directly instead of the whole State:
  ```rust
  // Instead of: fn add_log(state: &State<AppState>, ...)
  fn push_log(logs: &Arc<Mutex<Vec<SystemLog>>>, level: &str, message: &str) { ... }
  // Call: push_log(&state.logs, "INFO", "message");
  ```

### 4. Lint False Positives: "async move blocks only allowed in Rust 2018"
- The Hermes linter for `.rs` files may not read `Cargo.toml` edition field
- If `Cargo.toml` has `edition = "2021"`, these errors are false positives — ignore them

### 5. Web-to-Tauri Migration: Dead Dependencies
When migrating a web project (express/React/server-based) to Tauri:
- Remove server-side Node deps: express, better-sqlite3, multer, dotenv, @google/genai
- Remove React if frontend is vanilla TS (check `src/main.ts` — if no JSX, no React needed)
- Keep: vite, tailwindcss, @tailwindcss/vite, typescript
- The Rust backend (`main.rs`) replaces the Node server

### 6. Ubuntu 26.04 (Resolute) — System Dependencies
Required packages (usually pre-installed on GNOME desktop):
- `libwebkit2gtk-4.1-0` (runtime) + `libwebkit2gtk-4.1-dev` (build)
- `libgtk-3-0` / `libgtk-3-dev`
- `librsvg2-dev`, `libssl-dev`
- `libappindicator` is NOT required for Tauri v2 (was v1 only)
