---
name: tauri-expert
description: Expert skill for Tauri (v2) development, covering Rust backend, IPC, security (capabilities/plugins), and frontend integration.
author: Roedy Rustam
tags: [tauri, rust, desktop, mobile, cross-platform]
---

# Tauri Expert 🦀

> Best practices untuk membangun aplikasi desktop dan mobile lintas platform menggunakan **Tauri v2**.

## Kondisi Pemicu

Skill ini aktif saat:
- Membangun, men-debug, atau melakukan refactoring pada aplikasi Tauri
- Menulis sistem IPC antara frontend (React/Svelte/Vue) dan backend Rust
- Mengonfigurasi keamanan (tauri.conf.json, capabilities, permission)

---

## 1. Arsitektur & IPC (Inter-Process Communication)

### Rust → Frontend (Commands)

```rust
// Backend Rust — define command
#[tauri::command]
fn greet(name: &str) -> String {
    format!("Hello, {}!", name)
}

// Register in main
fn main() {
    tauri::Builder::default()
        .invoke_handler(tauri::generate_handler![greet])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

```typescript
// Frontend — invoke
import { invoke } from "@tauri-apps/api/core";

const msg = await invoke<string>("greet", { name: "World" });
```

### ⚡ Event System (Real-time)

Gunakan event untuk push dari Rust ke frontend, hindari polling:

```rust
// Rust emit
app_handle.emit("progress", ProgressPayload { current: 50, total: 100 }).ok();
```

```typescript
// Frontend listen
import { listen } from "@tauri-apps/api/event";

const unlisten = await listen<ProgressPayload>("progress", (event) => {
    console.log(event.payload.current, event.payload.total);
});
```

### Aturan IPC
- Operasi **berat** dan pemrosesan file di sisi Rust
- Frontend hanya fokus pada UI
- Gunakan `serde::{Serialize, Deserialize}` untuk data kompleks
- **Parameter naming (camelCase vs snake_case):** Tauri v2 secara otomatis memetakan key camelCase dari JavaScript ke parameter snake_case di Rust. Contoh: JS `invoke('open_folder', { folderPath: path })` → Rust `fn open_folder(folder_path: String)`. **Jangan gunakan snake_case di object JS** (misal `{ folder_path: path }`) — ini akan memicu error deserialisasi `missing required key folderPath` karena Tauri v2 sudah menangani pemetaan secara otomatis dan mengharapkan camelCase pada sisi JavaScript.

---

## 2. Manajemen State (Rust Backend)

```rust
use std::sync::Mutex;
use tauri::State;

pub struct AppState {
    pub counter: Mutex<i32>,
}

fn main() {
    tauri::Builder::default()
        .manage(AppState { counter: Mutex::new(0) })
        .invoke_handler(tauri::generate_handler![increment])
        .run(tauri::generate_context!())
        .expect("error");
}

#[tauri::command]
fn increment(state: State<AppState>) -> i32 {
    let mut val = state.counter.lock().unwrap();
    *val += 1;
    *val
}
```

- `std::sync::Mutex` untuk state sinkron
- `tokio::sync::Mutex` untuk state async (diperlukan jika di-hold melewati `.await`)
- Manage state di `tauri::Builder::default().manage(...)`

---

## 3. Keamanan (Security & Capabilities)

### Tauri v2 — Capabilities System

File `src-tauri/capabilities/default.json`:

```json
{
  "identifier": "default",
  "description": "Default capabilities for main window",
  "windows": ["main"],
  "permissions": [
    "core:default",
    "shell:allow-open",
    "dialog:default",
    "fs:allow-read-text-file",
    {
      "identifier": "fs:scope",
      "allow": [{ "path": "$APPDATA/**" }]
    }
  ]
}
```

### Prinsip Keamanan
- Isolasi konteks: `"devUrl": "http://localhost:1420"` → hanya akses di dev
- Jangan pernah ekspos filesystem, shell, atau API jaringan secara global
- Gunakan scoped permissions (path-specific)
- Validasi input dari frontend di sisi Rust

---

## 4. Kinerja & Asinkronisitas

```rust
#[tauri::command]
async fn read_large_file(path: String) -> Result<String, String> {
    let content = tokio::fs::read_to_string(&path)
        .await
        .map_err(|e| e.to_string())?;
    Ok(content)
}
```

- `async fn` untuk I/O berat — jangan blokir main thread
- Gunakan `tokio::fs` bukan `std::fs` untuk operasi file dalam command async
- `serde::Serialize` + `#[tauri::command]` untuk return data kompleks

---

## 5. Plugin Ecosystem (Tauri v2)

```rust
fn main() {
    tauri::Builder::default()
        .plugin(tauri_plugin_fs::init())
        .plugin(tauri_plugin_store::Builder::default().build())
        .plugin(tauri_plugin_dialog::init())
        .run(tauri::generate_context!())
        .expect("error");
}
```

Prefer plugin daripada modul I/O sendiri:
- `tauri-plugin-fs` — filesystem
- `tauri-plugin-store` — persistent key-value store
- `tauri-plugin-dialog` — native dialogs
- `tauri-plugin-shell` — shell commands (restricted!)
- `tauri-plugin-sql` — SQL database
- `tauri-plugin-upload` — file upload

---

## 6. Cross-Platform & Build

### Path Resolution (jangan hardcode)

```rust
use tauri::Manager;

// Di dalam command
fn get_app_dir(app: tauri::AppHandle) {
    let app_data = app.path().app_data_dir().unwrap();
    let cache = app.path().app_cache_dir().unwrap();
}
```

### Cargo.toml — Target Setup

```toml
[package]
name = "app"
version = "0.1.0"
edition = "2021"

[build-dependencies]
tauri-build = { version = "2", features = [] }

[dependencies]
tauri = { version = "2", features = [] }
tauri-plugin-fs = "2"
tauri-plugin-store = "2"
tauri-plugin-dialog = "2"
serde = { version = "1", features = ["derive"] }
serde_json = "1"

[features]
default = ["custom-protocol"]
custom-protocol = ["tauri/custom-protocol"]
```

### Build Commands
```bash
npm run tauri build          # Production build
npm run tauri dev            # Dev mode with hot-reload
```

---

## 7. Tauri v2 + Frontend Integration

### Prerequisites
```json
// package.json
{
  "dependencies": {
    "@tauri-apps/api": "^2.0.0",
    "@tauri-apps/plugin-fs": "^2.0.0",
    "@tauri-apps/plugin-dialog": "^2.0.0"
  }
}
```

### File Structure
```
my-tauri-app/
├── src/                    # Frontend (React/Svelte/Vue)
│   └── App.tsx
├── src-tauri/              # Rust backend
│   ├── src/
│   │   ├── main.rs         # Entry point
│   │   ├── lib.rs           # Library
│   │   └── commands/        # Command modules
│   ├── capabilities/        # Permissions
│   ├── Cargo.toml
│   ├── tauri.conf.json
│   └── icons/
└── package.json
```

---

## ⚠️ Pitfalls

| Masalah | Solusi |
|---------|--------|
| Command tidak terdaftar | Tambahkan ke `generate_handler![]` |
| Permission denied | Cek `capabilities/*.json` |
| Plugin error | Init plugin di `Builder::default().plugin(...)` |
| CORS di dev | Tauri dev server sudah handle otomatis |
| Binary terlalu besar | Cek `Cargo.toml`, minimal dependencies |
| `no method named 'path'` / `'manage'` | Tambahkan `use tauri::Manager;` di `lib.rs` — trait ini WAJIB di-import untuk akses `.path()`, `.manage()`, `.emit()` pada `App` / `AppHandle` |
| `cannot find __tauri_command_name_...` | Tauri `generate_handler!` membutuhkan command-command Rust untuk dapat diselesaikan ke modul aslinya agar macro compiler `__cmd__<name>` bekerja. Jika mere-export commands di barrel file (seperti `commands/mod.rs`), gunakan path lengkap modul aslinya di handler (misal `commands::download_commands::my_cmd` bukan `commands::my_cmd`). |
| `tokio::spawn` panic: "no reactor running" in `setup` | `tauri::Builder.setup()` closure runs OUTSIDE the Tokio async runtime context. Calling `tokio::spawn()` here causes panic. **Fix:** Use `tauri::async_runtime::spawn()` instead — it accesses Tauri's internal Tokio runtime handle. Inner spawns inside the async block can use either `tokio::spawn` or `tauri::async_runtime::spawn`. |
| Frontend `invoke()` returns "Command X not found" | Ensure: (1) command registered in `generate_handler![]`, (2) `@tauri-apps/api` is in `dependencies` (NOT `devDependencies`) in `package.json`, (3) import from `@tauri-apps/api/core` not `@tauri-apps/api`. Vanilla template doesn't include this package by default. |
| Field type mismatch between commands and DB queries | Rust is strict: if DB query expects `i64` but command receives `i32`, compilation fails. Always check the model struct field types and match them exactly in command function signatures. Common: `priority: i64` in model but `i32` in command arg. |
| `AppError` tidak bisa di-return dari command | Error type harus implement `serde::Serialize`. Pattern: derive `thiserror::Error` + impl custom `Serialize` yang convert ke string (lihat contoh di bawah) |
| Stale DB saat schema berubah | `CREATE TABLE IF NOT EXISTS` tidak update kolom. Hapus/rename DB lama atau pakai `PRAGMA user_version` untuk migration tracking |
| TailwindCSS v4 butuh `tailwind.config.js` | Tidak perlu! TailwindCSS v4 pakai `@tailwindcss/postcss` plugin di `postcss.config.js`. Cukup `@import "tailwindcss"` di CSS |
| Vanilla template tidak punya Vite | Install manual `npm i -D vite`, pindahkan `index.html` ke root, update `tauri.conf.json` (`frontendDist: "../dist"`, `devUrl`, `beforeDevCommand`) |
| WebkitGTK (Tauri Linux) `<select>` options text invisible | Native `<select>` options in WebkitGTK on Linux (Ubuntu) often inherit conflicting desktop styling causing white-on-white text. **Fix:** Force options style in global base CSS by disabling appearance and styling options explicitly: `select { appearance: none; } select option { background-color: #0a0a0a !important; color: #e5e5e5 !important; }`. Or completely replace native `<select>` dropdowns with modern custom grid button lists. |
| URL Parser issues (missing protocol) | Users often omit protocols (e.g. typing `www.youtube.com/...`). Tauri backend parsers must prepand `https://` if `http://` or `https://` is missing to avoid "Validation error: URL must start with http:// or https://". |
| yt-dlp/ffmpeg local user environment path mismatch | Tauri GUI apps run outside the user's login shell shell context and might miss binary paths in custom environments (like pyenv, hermes virtualenv, or `~/.local/bin`). **Fix:** Ensure binary paths are resolved by: (1) using robust `which` checks, (2) adding symbolic links from virtualenv paths directly to `/usr/local/bin/` so the system-wide daemon can locate them. |
| yt-dlp playlist dump-json NDJSON parser crash | When parsing `--dump-json` output of a playlist or generic link with warnings, `yt-dlp` returns NDJSON (Newline-Delimited JSON) or multiple lines of text. Standard `serde_json::from_str(&stdout)` fails with `Validation error: trailing characters at line 2 column 1`. **Fix:** Only read/parse the first non-empty line of the stdout. |
| yt-dlp progress parser stuck bug | Progress template outputs (e.g. `--progress-template "download:%(progress._percent_str)s ..."`) often include leading space paddings (e.g., `  0.0%` vs ` 10.5%`). Standard index checks on split whitespaces will drift, failing to parse percent values and leaving UI progress stuck at 0.0%. **Fix:** Standardize parsing by stripping the prefix, calling `split_whitespace()`, removing `%` characters, and checking array bounds dynamically. |
| Queue bypass in Tauri Commands | Direct invocation of async tasks (like `tokio::spawn(execute_download(...))`) inside an IPC command bypassing the main queue loop will result in multiple simultaneous runs, causing download concurrency limits to fail. **Fix:** Do not spawn background execution immediately in the command API. Write the task payload to a database queue table with status `Waiting`, and let a background loop (`QueueManager`) run sequentially. |
| Flat playlist metadata parsing | When analyzing links with both video and playlist parameters (e.g. `watch?v=...&list=...`), standard single-video queries miss other items. **Fix:** Force playlist extraction using `--flat-playlist` and capture multi-line JSON outputs. Iterate through each line to construct a `PlaylistItem` list with checkboxes, allowing users to selectively add to the active queue. |

### System Health Check Pattern (Desktop Apps)

For apps that depend on external binaries (FFmpeg, yt-dlp, etc.), implement a `check_health` command:

```rust
#[derive(Serialize, Clone)]
pub struct HealthStatus {
    pub db_connected: bool,
    pub disk_free_mb: u64,
    pub write_permission: bool,
    pub ffmpeg_found: bool,
    pub ffmpeg_version: String,
}

#[tauri::command]
pub fn check_health(state: State<'_, AppState>) -> Result<HealthStatus, AppError> {
    let (found, version) = match Command::new("ffmpeg").args(["-version"]).output() {
        Ok(out) => (true, String::from_utf8_lossy(&out.stdout).lines().next().unwrap_or("").to_string()),
        Err(_) => (false, "Not found".to_string()),
    };
    // ... check disk, DB, write perms
    Ok(HealthStatus { ffmpeg_found: found, ffmpeg_version: version, .. })
}
```

Use on startup for sidebar live status and in Settings → Advanced tab for full diagnostics.

### Background Task Spawning in setup()

**CRITICAL:** `tauri::Builder.setup()` runs outside Tokio runtime. Use `tauri::async_runtime::spawn()`:

```rust
.setup(|app| {
    let handle = app.handle().clone();
    // ✅ Correct — uses Tauri's internal runtime handle
    tauri::async_runtime::spawn(async move {
        loop {
            tokio::time::sleep(Duration::from_secs(3)).await;
            // ... queue polling, background work
        }
    });
    Ok(())
})
```

### AppError Serialization Pattern (Tauri v2)

Tauri v2 commands yang return `Result<T, E>` membutuhkan `E: serde::Serialize`. Pattern yang clean:

```rust
use serde::{Serialize, Serializer};
use thiserror::Error;

#[derive(Error, Debug)]
pub enum AppError {
    #[error("Database error: {0}")]
    Database(#[from] rusqlite::Error),
    #[error("I/O error: {0}")]
    Io(#[from] std::io::Error),
    #[error("{0}")]
    Custom(String),
}

impl Serialize for AppError {
    fn serialize<S>(&self, serializer: S) -> Result<S::Ok, S::Error>
    where S: Serializer {
        serializer.serialize_str(&self.to_string())
    }
}
```

### Background Post-Processing & Cleanup Pattern

When implementing post-download file manipulation (e.g., transcoding, decryption, extracting zip files):
1. **Download to Temp Path**: Save the download to a temporary path (`output_path.temp_source`).
2. **Execute Converter/Post-Processor**: Run the converter (e.g., FFmpeg, unzip).
3. **File System Cleanup**: If post-processing succeeds, clean up the temporary file (`std::fs::remove_file`) to prevent duplicate storage footprint. Fallback gracefully if it fails.
4. **Update DB Metadata**: Update the task status, final `filename`, and `output_path` properties in the SQLite database to match the newly generated file.

```rust
// Pattern implementation in Rust
pub async fn execute_task_with_postprocess(task: &DownloadTask) -> Result<(), AppError> {
    let temp_path = format!("{}.temp_source", task.output_path);
    
    // 1. Download
    ytdlp::download_media(&task.url, &temp_path, ...).await?;
    
    // 2. Post-Process (e.g. convert to MP3)
    let final_path = match ffmpeg::convert_to_audio(&temp_path, "mp3").await {
        Ok(path) => {
            // 3. Clean up temp source file
            let _ = std::fs::remove_file(&temp_path);
            path
        }
        Err(e) => {
            // Fallback: keep original temp if conversion fails
            let fallback = format!("{}.m4a", task.output_path);
            let _ = std::fs::rename(&temp_path, &fallback);
            fallback
        }
    };
    
    // 4. Update Database records
    let final_filename = Path::new(&final_path).file_name().unwrap().to_string_lossy();
    update_db_task_completion(task.id, &final_path, &final_filename).await?;
    
    Ok(())
}
```
