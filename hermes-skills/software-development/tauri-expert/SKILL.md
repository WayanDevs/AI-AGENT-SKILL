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
