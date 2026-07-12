---
name: prd-tauri-v2
description: "PRD generator khusus Tauri v2 apps. Pemicu: buat app Tauri, bikin PRD Tauri, desktop app baru, Tauri v2 project. Enforces 33 mandatory standards: Vanilla JS + TailwindCSS + Vite + Rust, SQLite, logging system, log viewer, settings menu, SPA router, sidebar navigation, docs folder (9 files), localization (en.json default), constants, services layer, event system, IPC validation, toast/snackbar, health check, lifecycle hooks, responsive layout. Default target OS: Ubuntu 26.04."
version: 1.0.0
author: User
license: MIT
user-invocable: true
metadata:
  hermes:
    tags: [prd, tauri, tauri-v2, desktop-app, rust, vanilla-js, tailwindcss, vite]
    related_skills: [prd, plan, spike, architecture-diagram]
---

# PRD Generator — Tauri v2 Apps

## Overview

Skill ini menghasilkan **Product Requirements Document (PRD) khusus untuk aplikasi Tauri v2**. Setiap PRD yang dihasilkan **wajib memenuhi 33 standar mandatory** — tidak ada kode yang boleh ditulis sebelum PRD disetujui user.

Tech stack yang di-enforce: **Vanilla JS + TailwindCSS + Vite** (frontend) + **Rust** (backend) + **SQLite** (database).

## When to Use

**Pemicu (trigger):**

- "buat app Tauri" / "bikin aplikasi Tauri"
- "PRD Tauri v2" / "bikin PRD untuk Tauri"
- "desktop app baru" / "buat desktop app"
- "Tauri v2 project baru"
- User minta buat app desktop tanpa menyebut framework → default ke Tauri v2

**Jangan gunakan untuk:**

- App web murni (tanpa desktop wrapper)
- Electron/Flutter/Qt app — ini khusus Tauri v2
- Bug fix atau refactoring app Tauri yang sudah ada
- Task dengan spesifikasi yang sudah sangat detail

## Mandatory Standards (M1-M33)

Setiap app Tauri v2 yang dibuat melalui skill ini **WAJIB** memiliki semua item berikut. Tidak boleh ada yang di-skip.

### Core Requirements

| # | Mandatory | Detail |
|---|-----------|--------|
| M1 | **Error Logging System** | Sistem logging di Rust backend: `info`, `warn`, `error`, `debug`. Semua operasi penting harus ter-log. Gunakan `log` + `env_logger` atau `tracing` crate. |
| M2 | **Log Viewer Menu** | Halaman UI dedicated untuk menampilkan log secara realtime — HTTP requests, errors, failed operations, timestamps. Filterable by log level. |
| M3 | **Settings Menu** | Halaman Settings wajib: theme, language, log level, app info, reset settings. Persisted ke SQLite atau file config. |
| M4 | **Tech Stack** | Frontend: **Vanilla JS + TailwindCSS + Vite**. Backend: **Rust**. Tidak boleh pakai React/Vue/Svelte/Angular atau framework JS lainnya. |
| M5 | **CHANGELOG.md** | File `CHANGELOG.md` di root project. Format: [Keep a Changelog](https://keepachangelog.com/). Update setiap release. |
| M6 | **Localization** | Folder `src/localization/` dengan file `en.json` sebagai default. Semua teks UI statis (menu, label, button, placeholder) harus dari file localization, bukan hardcoded. Bahasa lain ditambahkan nanti sesuai kebutuhan. |
| M7 | **Target OS** | **Wajib ditanyakan** saat klarifikasi: Ubuntu atau Windows. **Default: Ubuntu 26.04** jika user tidak memilih. Mempengaruhi build target, installer type, dan platform-specific config. |
| M8 | **SQLite Database** | Wajib pakai SQLite via `tauri-plugin-sql` atau `rusqlite`. Schema terdokumentasi di `docs/database.md`. Migrations terkelola. |
| M9 | **Separated Folder Structure** | Frontend (`src/`) dan Backend (`src-tauri/`) terpisah jelas. Lihat section Folder Structure di bawah. |
| M10 | **README.md** | Wajib ada di root project. Isi: deskripsi app, screenshot placeholder, prerequisites, install, dev setup, build, license. |
| M11 | **Docs Folder** | Folder `docs/` dengan **9 file mandatory** — lihat section Docs Structure. |
| M12 | **Vite** | Vite sebagai build tool dan dev server. Fast HMR. Config di `vite.config.js`. |
| M13 | **Global Error Handler** | Frontend: `window.onerror` + `unhandledrejection` → catch semua error, kirim ke Rust backend via Tauri command → masuk log system. Rust: panic handler yang graceful. |
| M14 | **Window Config** | Wajib configure di `tauri.conf.json`: default size, min size, title, resizable, center on launch. |
| M15 | **SPA Router** | Custom router di `src/router.js` untuk navigasi antar halaman tanpa page reload. Hash-based atau custom routing. |
| M16 | **Sidebar/Navigation** | Component navigasi utama (sidebar atau navbar) — minimal link ke: Home, Settings, Log Viewer. |
| M17 | **`.env.example`** | Template environment variables: debug mode, log level, db path, API endpoints. `.env` di-gitignore, `.env.example` di-commit. |
| M18 | **Custom Rust Error Types** | Module `src-tauri/src/errors/` dengan custom error enum, `impl Display`, konversi ke serializable format untuk frontend. |
| M19 | **Constants** | Frontend: `src/constants.js` — app name, version, default settings, log levels, route paths. Rust: `src-tauri/src/constants.rs` — app constants, db name, log config. Semua nilai yang bisa berubah harus dari constants, bukan hardcoded. |
| M20 | **Tauri v2 Version Pinning** | Wajib specify versi Tauri v2 di `Cargo.toml` dan constants. Lock versi agar tidak break saat update. |
| M21 | **Managed State Module** | Module `src-tauri/src/state/` — app state, DB connection pool, runtime config. Pakai Tauri `manage()` pattern. |
| M22 | **Event System Module** | Module `src-tauri/src/events/` — Tauri emit/listen events untuk realtime data (log streaming, progress update). Beda dari commands. |
| M23 | **IPC Input Validation** | Semua Tauri commands WAJIB validasi input di Rust side. Frontend TIDAK boleh trusted. Sanitize & validate sebelum proses. |
| M24 | **Services Layer** | Folder `src/services/` — abstraksi antara UI ↔ Tauri invoke. Pages tidak boleh langsung panggil `invoke()`, harus lewat service. |
| M25 | **Loading/Splash State** | Loading indicator saat app startup (init DB, load config, load localization). User tidak boleh lihat blank screen. |
| M26 | **Events Documentation** | File `docs/events.md` — dokumentasi semua Tauri events (emit/listen), payload format, kapan dipanggil. |
| M27 | **Error Toast/Snackbar** | Component toast/snackbar untuk feedback visual error ke user. Error bukan cuma masuk log — user harus lihat notifikasi. |
| M28 | **Responsive Layout** | Layout harus handle resize dari min width sampai fullscreen. Sidebar collapsible. Content area fluid. |
| M29 | **Runtime Config Loader** | Folder `src/config/` — load dari `.env`, merge dengan defaults dari constants. Config tersedia secara global. |
| M30 | **Health Check Command** | Tauri command `check_health` — verifikasi DB connected, disk space, permissions OK. Tampilkan status di Settings. |
| M31 | **Export Log** | Button di Log Viewer untuk export log ke `.txt` atau `.csv`. User bisa download log untuk debugging. |
| M32 | **Keyboard Shortcuts Doc** | File `docs/shortcuts.md` — daftar semua keyboard shortcuts. |
| M33 | **Lifecycle Hooks** | Folder `src/hooks/` — custom event hooks: `onAppReady`, `onPageChange`, `onError`, `onConfigChange`. |

## Folder Structure (Mandatory)

Setiap project Tauri v2 **wajib** mengikuti struktur ini:

```
app-name/
├── src-tauri/                      # 🦀 Rust Backend
│   ├── src/
│   │   ├── commands/               # Tauri commands per module
│   │   ├── db/                     # SQLite schema, migrations, queries
│   │   ├── models/                 # Rust data structs
│   │   ├── errors/                 # Custom error types
│   │   ├── logging/                # Log config, levels, file output
│   │   ├── state/                  # Managed state (app state, DB pool)
│   │   ├── events/                 # Tauri event system (emit/listen)
│   │   ├── constants.rs            # Rust-side constants
│   │   ├── lib.rs
│   │   └── main.rs
│   ├── Cargo.toml
│   ├── tauri.conf.json
│   └── capabilities/              # Tauri v2 permissions ACL
├── src/                            # 🌐 Frontend (Vanilla JS + TailwindCSS + Vite)
│   ├── assets/                     # Images, icons, fonts
│   ├── components/                 # Reusable UI (sidebar, modal, log-viewer, toast)
│   ├── pages/                      # Page modules (home, settings, log)
│   ├── styles/                     # TailwindCSS + custom CSS
│   ├── utils/                      # Helpers (logger, i18n, fetch wrapper)
│   ├── services/                   # Tauri invoke abstraction layer
│   ├── config/                     # Runtime config loader
│   ├── hooks/                      # Lifecycle hooks (onAppReady, onError, etc.)
│   ├── localization/
│   │   └── en.json                 # Default English only
│   ├── constants.js                # App name, version, defaults
│   ├── router.js                   # SPA page router
│   ├── main.js                     # Entry point
│   └── index.html
├── docs/
│   ├── api.md                      # Tauri commands documentation
│   ├── database.md                 # Schema, tables, relations, migrations
│   ├── changelog.md                # Detailed version changes
│   ├── events.md                   # Tauri events (emit/listen) documentation
│   ├── layout.md                   # UI layout, wireframes, navigation
│   ├── setup.md                    # Dev environment setup guide
│   ├── structure.md                # Project structure explanation
│   ├── permissions.md              # Tauri v2 ACL permissions docs
│   └── shortcuts.md                # Keyboard shortcuts documentation
├── CHANGELOG.md
├── README.md
├── LICENSE
├── .env.example
├── .gitignore
├── package.json
├── vite.config.js
├── tailwind.config.js
└── postcss.config.js
```

## Docs Structure (9 Mandatory Files)

| File | Isi Wajib |
|------|-----------|
| `docs/api.md` | Semua Tauri commands: nama, input params, output, error codes, contoh pemanggilan dari frontend |
| `docs/database.md` | Schema SQLite: nama tabel, kolom, tipe data, relasi, index, migration history |
| `docs/changelog.md` | Detail perubahan per versi — lebih detail dari CHANGELOG.md root, bisa include reasoning |
| `docs/layout.md` | Layout UI per halaman: wireframe text-based, komponen yang dipakai, navigasi flow |
| `docs/setup.md` | Step-by-step setup dev environment: install Rust, Node.js, Tauri CLI, clone, run dev |
| `docs/structure.md` | Penjelasan setiap folder dan file — apa isinya, kapan diubah, konvensi penamaan |
| `docs/permissions.md` | Daftar Tauri v2 permissions/capabilities yang dipakai, alasan, security implications |
| `docs/events.md` | Semua Tauri events: nama event, payload format, emitter (Rust/frontend), listener, kapan dipanggil |
| `docs/shortcuts.md` | Daftar semua keyboard shortcuts: key combo, aksi, halaman mana, customizable atau tidak |

## Workflow

### ⛔ Step 0: GATEKEEPER — STOP. JANGAN NULIS KODE.

Begitu terpicu, **hentikan** semua pikiran tentang kode. Tugas pertama adalah **analisis kebutuhan produk** melalui klarifikasi.

### ❓ Step 1: Clarifying Questions (Bahasa Indonesia)

Ajukan **4-6 pertanyaan klarifikasi** dengan opsi A/B/C/D. User bisa jawab cepat seperti "1A, 2C, 3B".

**Pertanyaan WAJIB:**

1. **Target OS** — pertanyaan ini WAJIB ada di posisi pertama:
   A. Ubuntu 26.04 (default)
   B. Windows 11
   C. Keduanya (Ubuntu + Windows)
   D. Lainnya: [sebutkan]

2. **Tujuan utama aplikasi** — apa yang dilakukan app ini?
   A. [opsi sesuai konteks]
   B. [opsi sesuai konteks]
   C. [opsi sesuai konteks]
   D. Lainnya: [sebutkan]

3. **Target pengguna**
   A. Personal use
   B. Tim/internal
   C. Publik/end-user
   D. Lainnya

4. **Scope**
   A. MVP (fitur minimal dulu)
   B. Versi lengkap
   C. Prototype/experiment
   D. Lainnya

**Pertanyaan opsional (tanyakan jika relevan):**

- Perlu auto-update?
- Perlu system tray?
- Perlu multi-window?
- Perlu dark/light theme?
- Integrasi API eksternal?

### 📝 Step 2: Generate PRD (Bahasa Inggris)

Berdasarkan jawaban, generate PRD dengan struktur standar. **Semua 33 mandatory items HARUS tercakup** dalam PRD — baik di Functional Requirements, Technical Considerations, atau section tersendiri.

**PRD Structure:**

```markdown
# PRD: [App Name]

## 1. Introduction / Overview
Brief description + problem it solves.

## 2. Goals
Specific, measurable objectives.

## 3. Target Platform
- Primary: [Ubuntu 26.04 / Windows 11 / Both]
- Build targets: [deb, AppImage / NSIS, MSI / all]

## 4. Tech Stack
- Frontend: Vanilla JS + TailwindCSS + Vite
- Backend: Rust (Tauri v2)
- Database: SQLite
- Build: Vite
- Styling: TailwindCSS + PostCSS

## 5. Localization
- Default: `en.json` (English)
- Location: `src/localization/`
- All static UI text from localization file, never hardcoded

## 6. User Stories
US-001 format with acceptance criteria.

## 7. Functional Requirements
FR-1 numbered list.

## 8. Mandatory Features Checklist
Checklist of all M1-M33 with status.

## 9. Non-Goals (Out of Scope)

## 10. Folder Structure
Full tree as defined in skill.

## 11. Documentation Requirements
List of all 9 docs/ files + README.md + CHANGELOG.md.

## 12. Design Considerations
UI/UX, sidebar layout, pages, theme.

## 13. Technical Considerations
Tauri v2 specifics, permissions, IPC, error handling.

## 14. Success Metrics

## 15. Open Questions
```

Simpan ke: `tasks/prd-[app-name].md`

### ✅ Step 3: Minta Persetujuan (Bahasa Indonesia)

> "Apakah Anda setuju dengan PRD ini? Ada yang mau ditambah/kurangi sebelum lanjut ke implementasi?"

### 🚀 Step 4: Implementation Plan

**Hanya** setelah user setuju, lanjut ke:
1. Implementation plan (task list per user story)
2. Arsitektur teknis detail
3. Penulisan kode — mulai dari folder structure + constants + boilerplate

## Constants Convention

### Frontend (`src/constants.js`)

```javascript
export const APP = {
  NAME: 'My App',
  VERSION: '0.1.0',
  DESCRIPTION: 'Short description',
};

export const ROUTES = {
  HOME: '/',
  SETTINGS: '/settings',
  LOG_VIEWER: '/log',
};

export const LOG_LEVELS = {
  DEBUG: 'debug',
  INFO: 'info',
  WARN: 'warn',
  ERROR: 'error',
};

export const DEFAULTS = {
  THEME: 'light',
  LANGUAGE: 'en',
  LOG_LEVEL: 'info',
};
```

### Backend (`src-tauri/src/constants.rs`)

```rust
pub const APP_NAME: &str = "my-app";
pub const APP_VERSION: &str = "0.1.0";
pub const DB_NAME: &str = "app.db";
pub const LOG_FILE: &str = "app.log";
pub const DEFAULT_WINDOW_WIDTH: f64 = 1024.0;
pub const DEFAULT_WINDOW_HEIGHT: f64 = 768.0;
pub const MIN_WINDOW_WIDTH: f64 = 800.0;
pub const MIN_WINDOW_HEIGHT: f64 = 600.0;
```

## Window Config Convention (`tauri.conf.json`)

```json
{
  "app": {
    "windows": [
      {
        "title": "App Name",
        "width": 1024,
        "height": 768,
        "minWidth": 800,
        "minHeight": 600,
        "resizable": true,
        "center": true,
        "fullscreen": false
      }
    ]
  }
}
```

## Common Pitfalls

1. **Langsung nulis kode.** STOP. PRD dulu, approval dulu, baru kode.
2. **Skip mandatory items.** Semua M1-M33 wajib ada di PRD. Cek ulang sebelum present ke user.
3. **Hardcode teks UI.** Semua teks statis harus dari `localization/en.json`.
4. **Hardcode app name/version.** Harus dari constants — frontend `constants.js` + Rust `constants.rs`.
5. **Lupa tanyakan Target OS.** Pertanyaan pertama WAJIB Target OS. Default Ubuntu 26.04.
6. **Frontend pakai framework.** Vanilla JS only. Tidak boleh React/Vue/Svelte.
7. **Tidak buat docs/.** 9 file docs mandatory. Bukan opsional.
8. **Error tidak ter-log.** Setiap error harus masuk logging system — frontend errors via global handler → Rust backend → log file.
9. **Tauri v2 permissions salah.** Permissions ACL di Tauri v2 sering jadi sumber bug. Dokumentasikan di `docs/permissions.md`.
10. **Tidak ada .env.example.** Environment variables harus ada template-nya.
11. **Constants tidak sinkron.** APP_VERSION di `constants.js`, `constants.rs`, `package.json`, `Cargo.toml`, dan `tauri.conf.json` harus sama. Dokumentasikan ini di `docs/structure.md`.
12. **Tidak ada services layer.** Pages langsung panggil `invoke()` → sulit maintain. Wajib lewat `src/services/`.
13. **Tidak ada loading state.** App startup tanpa loading indicator → user lihat blank screen. Wajib ada splash/loading.
14. **Event system tidak terdokumentasi.** Events di `docs/events.md` — beda dari commands di `docs/api.md`. Jangan campur.
15. **IPC tidak divalidasi.** Semua input dari frontend WAJIB divalidasi di Rust side. Jangan trust frontend data.
16. **Toast/snackbar tidak ada.** Error hanya masuk log tapi user tidak dapat feedback visual → UX buruk.
17. **Layout tidak responsive.** Window bisa di-resize — layout harus handle min width sampai fullscreen. Sidebar harus collapsible.

## Verification Checklist

- [ ] STOP — tidak ada kode sebelum PRD disetujui
- [ ] Target OS ditanyakan (default Ubuntu 26.04)
- [ ] Semua M1-M33 tercakup di PRD
- [ ] PRD ditulis dalam Bahasa Inggris
- [ ] Komunikasi dengan user dalam Bahasa Indonesia
- [ ] Folder structure sesuai template
- [ ] 9 file docs/ tercantum
- [ ] Constants convention tercantum (frontend + Rust)
- [ ] Window config tercantum
- [ ] Localization default en.json saja
- [ ] Tech stack: Vanilla JS + TailwindCSS + Vite + Rust + SQLite
- [ ] User stories punya acceptance criteria yang verifiable
- [ ] Functional requirements di-number (FR-1, FR-2, ...)
- [ ] Non-goals jelas
- [ ] File disimpan ke `tasks/prd-[app-name].md`
- [ ] Minta persetujuan user sebelum implementasi
- [ ] Services layer ada (pages tidak langsung invoke)
- [ ] Event system terdokumentasi di docs/events.md
- [ ] IPC input validation di Rust side
- [ ] Toast/snackbar component untuk error feedback
- [ ] Loading state saat startup
- [ ] Responsive layout dengan collapsible sidebar
- [ ] Health check command tersedia
- [ ] Export log feature di Log Viewer
- [ ] Lifecycle hooks terdefinisi
- [ ] Tauri v2 version pinned di Cargo.toml
