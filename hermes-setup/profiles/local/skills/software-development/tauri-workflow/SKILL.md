---
name: tauri-workflow
description: Use when building, testing, or deploying Tauri v2 cross-platform desktop applications. Provides end-to-end workflow from PRD setup, IPC safety, state handling to builds.
version: 1.0.0
author: Antigravity
license: MIT
metadata:
  hermes:
    tags: [tauri, rust, desktop, frontend, workflow]
    related_skills: [tauri-expert, rust-programming-expert, plan, github-pr-workflow]
---

# Tauri v2 Desktop App Development Workflow 🦀

> **Trigger Condition**: Skill ini HANYA aktif secara otomatis jika Profile Hermes aktif adalah `tauri` DAN terdeteksi file proyek Tauri (`tauri.conf.json` atau folder `src-tauri`) pada direktori kerja saat ini. Di luar kondisi tersebut, status skill ini adalah DORMANT (non-aktif).

## 💡 Profile Isolation Setup Pitfall
Jika Anda ingin skill ini hanya diakses pada profile `tauri`, pastikan file aslinya terletak di folder global (`~/.hermes/skills/software-development/tauri-workflow/SKILL.md`) dan di-symlink ke folder profile target (`~/.hermes/profiles/tauri/skills/software-development/tauri-workflow`). Hapus folder kategori yang tidak terpakai di bawah direktori profile untuk menyembunyikan skill yang tidak relevan dengan Tauri.

## Overview
Panduan end-to-end untuk pengembangan aplikasi desktop menggunakan Tauri v2. Menghubungkan proses perancangan PRD, inisialisasi scaffold proyek, konfigurasi sistem keamanan capability-based, IPC (Inter-Process Communication) asinkron, hingga penanganan state backend Rust dan deployment/build.

## When to Use
- Menginisialisasi aplikasi desktop Tauri v2 baru.
- Merancang kapabilitas keamanan (capabilities/permissions) untuk frontend-backend.
- Melakukan troubleshooting build Tauri yang gagal.
- Mengatur komunikasi IPC tipe aman (`serde`) antara UI dan Rust backend.
- **Dormant default:** Skill ini tidak membebani session profile lain secara otomatis kecuali dipanggil manual.

## 1. Phase 1: Scoping & PRD Definition

Before writing any code, draft the product requirements document (PRD) specifying the scope:
1. **Frontend Architecture:** Choose the framework (React, Vue, Svelte, Next.js, etc.) and builder (Vite, Cargo).
2. **Backend Services:** Detail what operations must run natively in Rust (heavy computing, local SQLite, shell execution, hotkeys, native protocols).
3. **Capabilities & Permission Matrix:** Explicitly list required OS permissions (Filesystem scopes, Network, Dialogs, Window management, Auto-start).
4. **Target Platforms:** Windows (`.msi`, `.exe`), macOS (`.dmg`, `.app`), and/or Linux (`.deb`, `.AppImage`).

---

## 2. Phase 2: Project Scaffolding

Generate the project template securely and consistently.

### Initialization Command
Run inside your workspace folder:
```bash
# Using npm/npx
npx create-tauri-app@latest
```
Follow prompts:
- **Project name:** `my-tauri-app`
- **Frontend language:** TypeScript / JavaScript
- **Package manager:** npm / pnpm / yarn / bun
- **UI Template:** Svelte / React / Vue / HTML (Vite-backed is highly recommended)

### Core Folder Structure Checklist
Verify the generated project matches:
- `src/` - Frontend assets & components (React/Svelte/Vue)
- `src-tauri/` - Rust crate (Backend)
  - `capabilities/` - Security permissions (JSON definitions)
  - `src/` - Rust source code (`main.rs`, `lib.rs`)
  - `Cargo.toml` - Rust dependencies
  - `tauri.conf.json` - Tauri settings, build configs, and plugin configuration

---

## 3. Phase 3: Security & Capabilities Architecture

Tauri v2 uses strict capability-based authorization. Frontend calls to core modules or plugins fail unless explicitly allowed.

### Designing a Default Capability
Create/edit `src-tauri/capabilities/default.json`:
```json
{
  "$schema": "../gen/schemas/desktop-schema.json",
  "identifier": "default-runtime",
  "description": "Required APIs for basic app operations",
  "windows": ["main"],
  "permissions": [
    "core:default",
    "fs:allow-read-text-file",
    "fs:allow-write-file",
    {
      "identifier": "fs:scope",
      "allow": [
        { "path": "$APPDATA/**" },
        { "path": "$RESOURCE/**" }
      ]
    },
    "dialog:allow-open",
    "dialog:allow-save"
  ]
}
```

### Capability Scopes
- Always bind access to scopes like `$APPDATA` or `$DOCUMENT` instead of allowing access to the whole root filesystem (`/` or `C:\`).
- Avoid letting the frontend run raw shell commands. Implement custom shell wrapper functions in Rust commands instead of using `tauri-plugin-shell` directly if safety is high priority.

---

## 4. Phase 4: Safe IPC & State Implementation

Bridge the frontend and backend using typesafe, asynchronous communication.

### Rust Command (with robust error handling)
Return structured results (`Result<T, E>`) to handle exceptions gracefully in JavaScript.
```rust
// src-tauri/src/commands.rs
use serde::Serialize;

#[derive(Serialize)]
pub struct ProcessResult {
    success: bool,
    message: String,
}

#[tauri::command]
pub async fn process_data(payload: String) -> Result<ProcessResult, String> {
    if payload.is_empty() {
        return Err("Payload cannot be empty".into());
    }
    // Simulate complex operations (e.g. database, computation)
    Ok(ProcessResult {
        success: true,
        message: format!("Processed payload size: {}", payload.len()),
    })
}
```

### Frontend Binding
```typescript
import { invoke } from "@tauri-apps/api/core";

interface ProcessResult {
  success: boolean;
  message: string;
}

async function triggerProcess(data: string) {
  try {
    const result = await invoke<ProcessResult>("process_data", { payload: data });
    console.log("Success:", result.message);
  } catch (error) {
    console.error("Rust Command Failed:", error);
  }
}
```

---

## 5. Common Pitfalls & Troubleshooting

### IPC Command Fails quietly or drops error details
* **Reason:** Rust command doesn't implement `serde::Serialize` on custom struct outputs or returns a type that JavaScript doesn't natively map.
* **Fix:** Ensure all command output structs derive `serde::Serialize` and input parameters derive `serde::Deserialize`.

### Development Window is Blank
* **Reason:** The web dev-server is not running or the port in `tauri.conf.json` (`build.devUrl`) doesn't match the port Vite is running on.
* **Fix:** Run frontend server manually `npm run dev` and confirm the local address, then verify `tauri.conf.json` points to it.

### Link errors / Missing C Compiler on Build
* **Reason:** Tauri requires system-specific C compiler, build tools, and GTK libraries on Linux or MSVC Build tools on Windows.
* **Fix:** Run system dependencies installer:
  - **Debian/Ubuntu:** `sudo apt install libwebkit2gtk-4.1-dev build-essential curl wget file libssl-dev libgtk-3-dev libayumu-dev libayatana-appindicator3-dev librsvg2-dev`
  - **Windows:** Install Visual Studio C++ Build Tools.

### Kehilangan Konteks Multi-Profile Saat Berbagi Skill
* **Reason:** Skill dibuat hanya di satu direktori profile sehingga tidak terbaca saat berganti profile Hermes.
* **Fix:** Gunakan symbolic link (symlink) dari direktori global `~/.hermes/skills/` ke direktori profile target `~/.hermes/profiles/<profile-name>/skills/` untuk berbagi file skill secara efisien tanpa duplikasi.

---

## 6. Verification Checklist

- [ ] Project compiles locally in dev mode (`npm run tauri dev`).
- [ ] IPC command payloads and response formats match frontend expectation.
- [ ] Folder paths are resolved via Rust API `app.path()` instead of hardcoded strings.
- [ ] Release packages are successfully bundled (`npm run tauri build`).
- [ ] Unused system plugins are stripped from dependencies to minimize binary size.
