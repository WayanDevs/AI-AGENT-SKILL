---
name: prd-architect
description: "Global PRD gatekeeper & generator untuk SEMUA jenis aplikasi (Flutter, Next.js, Django, Laravel, React Native, Electron, dll). Pemicu: buat aplikasi baru, bikin PRD, desktop app, mobile app, web app, SaaS, MVP. Enforces mandatory standards: logging, settings, docs folder, localization, constants, services layer, error handling, responsive layout. Tanya tech stack di awal. PRD bahasa Inggris, komunikasi Bahasa Indonesia."
version: 2.0.0
category: general
status: published
confidence: 0.9
source: learned
owner: wayan
created: "2026-07-04T11:57:32Z"
---

# PRD Architect — Global PRD Generator

## Overview

Skill ini adalah **palang pintu utama (gatekeeper)** untuk pembuatan aplikasi baru dari nol — **tidak ada kode yang boleh ditulis sebelum PRD disetujui user.**

Skill ini bersifat **universal** — mendukung semua tech stack dan platform:
- **Desktop:** Electron, .NET MAUI, Qt
- **Mobile:** Flutter, React Native, Kotlin, Swift
- **Web:** Next.js, Nuxt.js, SvelteKit, Django, Laravel, Rails, FastAPI, Express
- **Hybrid/Cross-platform:** Flutter, React Native, Ionic, Capacitor
- **Backend-only:** Go, Rust, Node.js, Python, Java/Spring

> ⚠️ **Untuk Tauri v2 apps** → gunakan skill `prd-tauri-v2` yang memiliki 33 mandatory standards khusus Tauri.

## When to Use

**Pemicu (trigger):**

- "buat aplikasi baru" / "bikin app baru"
- "bikin PRD" / "buat PRD" / "tulis PRD untuk..."
- "buat proyek baru" / "proyek baru dari nol"
- "buat SaaS" / "SaaS baru" / "buat MVP"
- "desktop app baru" (yang BUKAN Tauri)
- "mobile app baru" / "buat app Flutter" / "React Native app"
- "web app baru" / "Next.js project"
- "backend API baru"
- User bilang "gas bikin [app/fitur]" tanpa spesifikasi detail
- User minta agent menjadi Product Manager (PM)

**Redirect ke skill lain:**

- "buat app Tauri" / "Tauri v2 project" → **gunakan `prd-tauri-v2`**
- Bug fix / refactoring → **tidak perlu PRD**
- Task dengan spesifikasi yang sudah sangat detail → **langsung kerjakan**

---

## Mandatory Standards (M1-M25)

Setiap aplikasi yang dibuat melalui skill ini **WAJIB** memiliki item berikut. Adaptasikan sesuai tech stack yang dipilih.

### Core Requirements

| # | Mandatory | Detail |
|---|-----------|--------|
| M1 | **Error Logging System** | Sistem logging yang terstruktur: `info`, `warn`, `error`, `debug`. Semua operasi penting harus ter-log. Pilih library sesuai stack (Winston/Pino untuk Node.js, logging untuk Python, logback untuk Java, dll). |
| M2 | **Settings/Preferences** | Halaman atau mekanisme settings: theme, language, notifikasi, app info. Persisted ke database atau local storage. |
| M3 | **Tech Stack Documentation** | Tech stack harus terdokumentasi jelas di PRD dan README. Versi library/framework di-pin. |
| M4 | **CHANGELOG.md** | File `CHANGELOG.md` di root project. Format: [Keep a Changelog](https://keepachangelog.com/). |
| M5 | **Localization** | Folder `localization/` (atau `i18n/`, `l10n/` sesuai konvensi stack) dengan file bahasa default. Semua teks UI statis harus dari file localization, bukan hardcoded. **Wajib tanyakan** bahasa default ke user. |
| M6 | **Target Platform** | **Wajib ditanyakan**: OS/platform target. Mempengaruhi build target, installer, dan platform-specific config. |
| M7 | **Database** | Pilihan database terdokumentasi (SQLite, PostgreSQL, MySQL, Supabase, Firebase, dll). Schema terdokumentasi di `docs/database.md`. |
| M8 | **Separated Architecture** | Separation of concerns yang jelas — frontend/backend terpisah, atau layer yang terdefinisi (MVC, MVVM, Clean Architecture, dll). |
| M9 | **README.md** | Wajib ada di root. Isi: deskripsi app, screenshot placeholder, prerequisites, install, dev setup, build, license. |
| M10 | **Docs Folder** | Folder `docs/` dengan file-file dokumentasi — sesuaikan jumlah dengan kompleksitas project. Minimal: `api.md`, `database.md`, `setup.md`, `structure.md`. |
| M11 | **Global Error Handler** | Error handler global — uncaught exceptions, unhandled rejections, crash reporting. Error harus ter-log DAN user harus dapat feedback visual. |
| M12 | **Environment Config** | File `.env.example` sebagai template. `.env` di-gitignore. Semua config sensitif lewat env vars. |
| M13 | **Constants/Config** | File constants terpusat — app name, version, defaults, route paths, API endpoints. Tidak boleh hardcode nilai yang bisa berubah. |
| M14 | **Services/Repository Layer** | Abstraksi antara UI ↔ data source. UI tidak boleh langsung panggil API/database. Harus lewat service/repository layer. |
| M15 | **Loading/Splash State** | Loading indicator saat startup. User tidak boleh lihat blank screen. |
| M16 | **Error Feedback UI** | Toast, snackbar, alert, atau mekanisme feedback visual ke user. Error bukan cuma masuk log — user harus tahu ada masalah. |
| M17 | **Responsive/Adaptive Layout** | Layout harus handle berbagai ukuran layar sesuai platform (mobile: portrait/landscape, desktop: resize, web: responsive breakpoints). |
| M18 | **Navigation** | Sistem navigasi yang jelas — sidebar, tab bar, drawer, bottom nav, atau routing sesuai platform. |
| M19 | **Input Validation** | Semua user input WAJIB divalidasi — client-side DAN server-side. Sanitize sebelum proses. |
| M20 | **Health Check** | Endpoint atau command untuk verifikasi: DB connected, services reachable, disk space OK. |
| M21 | **Version Pinning** | Semua dependency di-lock/pin versinya. Lockfile wajib di-commit (package-lock.json, Cargo.lock, pubspec.lock, dll). |
| M22 | **Folder Structure** | Struktur folder terdokumentasi di PRD dan `docs/structure.md`. Konsisten dan scalable. |
| M23 | **State Management** | Strategi state management terdefinisi sesuai stack (Redux, Zustand, Riverpod, BLoC, Vuex, dll). |
| M24 | **Testing Strategy** | Minimal: unit tests untuk business logic. Dokumentasikan di PRD. Framework test sesuai stack. |
| M25 | **CI/CD Awareness** | Dokumentasikan di PRD bagaimana app akan di-build dan deploy, meskipun belum diimplementasi di v1. |

---

## Workflow

### ⛔ Step 0: GATEKEEPER — STOP. JANGAN NULIS KODE.

Begitu terpicu, **hentikan** semua pikiran tentang kode. Tugas pertama adalah **analisis kebutuhan produk** melalui klarifikasi.

> Jika user minta buat **Tauri v2 app** → redirect ke skill `prd-tauri-v2`.

### ❓ Step 1: Clarifying Questions (Bahasa Indonesia)

Ajukan **5-7 pertanyaan klarifikasi** dengan opsi A/B/C/D. User bisa jawab cepat seperti "1A, 2C, 3B, 4D".

**Pertanyaan WAJIB (urutan ini):**

1. **Jenis aplikasi & tech stack** — pertanyaan ini WAJIB di posisi pertama:
   A. Web app (Next.js / Nuxt / SvelteKit / Django / Laravel / lainnya)
   B. Mobile app (Flutter / React Native / Kotlin / Swift / lainnya)
   C. Desktop app (Electron / .NET MAUI / lainnya)
   D. Backend/API only (Express / FastAPI / Go / Spring / lainnya)
   E. Lainnya: [sebutkan]

   > Jika jawaban = **Tauri v2** → STOP, redirect ke skill `prd-tauri-v2`.

2. **Target platform/OS:**
   A. Web (semua browser modern)
   B. Android + iOS
   C. Ubuntu/Linux
   D. Windows
   E. macOS
   F. Cross-platform (sebutkan)

3. **Tujuan utama aplikasi** — apa yang dilakukan app ini?
   A. [opsi sesuai konteks]
   B. [opsi sesuai konteks]
   C. [opsi sesuai konteks]
   D. Lainnya: [sebutkan]

4. **Target pengguna:**
   A. Personal use
   B. Tim/internal
   C. Publik/end-user
   D. Lainnya

5. **Scope:**
   A. MVP (fitur minimal dulu)
   B. Versi lengkap
   C. Prototype/experiment
   D. Lainnya

6. **Bahasa localization default:**
   A. English (`en.json`)
   B. Bahasa Indonesia (`id.json`)
   C. Keduanya
   D. Lainnya: [sebutkan]

**Pertanyaan opsional (tanyakan jika relevan):**

- Database preference? (SQLite / PostgreSQL / Supabase / Firebase / MongoDB)
- Perlu authentication? (email/password, OAuth, SSO)
- Perlu real-time features? (WebSocket, SSE, push notification)
- Hosting/deployment target? (Vercel, AWS, VPS, self-hosted)
- Dark/light theme?
- Offline support?

### 📝 Step 2: Generate PRD (Bahasa Inggris)

Berdasarkan jawaban, generate PRD dengan struktur standar. **Semua 25 mandatory items HARUS tercakup** — adaptasikan sesuai tech stack yang dipilih.

**PRD Structure:**

```markdown
# PRD: [App Name]

## 1. Introduction / Overview
Brief description + problem it solves.

## 2. Goals
Specific, measurable objectives.

## 3. Target Platform
- Primary: [Web / Android+iOS / Ubuntu / Windows / Cross-platform]
- Build targets: [sesuai platform]

## 4. Tech Stack
- Frontend: [framework + version]
- Backend: [framework + version]
- Database: [engine + version]
- Build tool: [tool]
- Styling: [framework/approach]
- State management: [library]
- Testing: [framework]

## 5. Localization
- Default: [bahasa + filename]
- Location: [folder path]
- All static UI text from localization file, never hardcoded

## 6. User Stories
US-001 format with acceptance criteria.

## 7. Functional Requirements
FR-1 numbered list.

## 8. Mandatory Features Checklist
Checklist of all M1-M25 with status and stack-specific implementation.

## 9. Non-Goals (Out of Scope)

## 10. Folder Structure
Full tree sesuai tech stack + konvensi framework.

## 11. Documentation Requirements
List docs/ files sesuai kompleksitas project.

## 12. Design Considerations
UI/UX, navigation pattern, layout, theme.

## 13. Technical Considerations
Architecture, API design, auth, error handling, performance.

## 14. Testing Strategy
Unit tests, integration tests, E2E tests — framework + scope.

## 15. Deployment & CI/CD
Build, deploy target, CI pipeline awareness.

## 16. Success Metrics

## 17. Open Questions
```

Simpan ke: `tasks/prd-[app-name].md`

### ✅ Step 3: Minta Persetujuan (Bahasa Indonesia)

> "Apakah Anda setuju dengan PRD ini? Ada yang mau ditambah/kurangi sebelum lanjut ke implementasi?"

### 🚀 Step 4: Implementation Plan

**Hanya** setelah user setuju, lanjut ke:
1. Implementation plan (task list per user story)
2. Arsitektur teknis detail
3. Penulisan kode — mulai dari folder structure + constants + boilerplate

---

## Docs Structure (Minimum 4, Sesuaikan Kompleksitas)

| File | Isi Wajib | Kapan Wajib |
|------|-----------|-------------|
| `docs/api.md` | API endpoints / commands: nama, input, output, error codes | Selalu (jika ada API/backend) |
| `docs/database.md` | Schema: tabel, kolom, tipe data, relasi, index, migrations | Selalu (jika ada database) |
| `docs/setup.md` | Step-by-step setup dev environment | Selalu |
| `docs/structure.md` | Penjelasan folder & file, konvensi penamaan | Selalu |
| `docs/changelog.md` | Detail perubahan per versi | Untuk project besar |
| `docs/layout.md` | Layout UI per halaman, wireframe, navigation flow | Jika ada UI |
| `docs/events.md` | Event system documentation (emit/listen, payload) | Jika pakai event system |
| `docs/permissions.md` | Permissions, roles, access control | Jika ada auth/permissions |
| `docs/shortcuts.md` | Keyboard shortcuts | Jika desktop app |
| `docs/deployment.md` | Deployment guide, CI/CD config | Untuk production apps |

---

## Stack-Specific Folder Structures

### Flutter App

```
app-name/
├── lib/
│   ├── core/                  # Constants, themes, utils, extensions
│   ├── features/              # Feature-based modules
│   │   └── feature_name/
│   │       ├── data/          # Repositories, data sources, models
│   │       ├── domain/        # Entities, use cases
│   │       └── presentation/  # Screens, widgets, controllers
│   ├── l10n/                  # Localization (ARB files)
│   ├── services/              # App-wide services (auth, storage, api)
│   ├── router/                # Navigation/routing (GoRouter/AutoRoute)
│   └── main.dart
├── test/                      # Unit & widget tests
├── integration_test/          # Integration tests
├── assets/                    # Images, fonts, etc
├── docs/
├── CHANGELOG.md
├── README.md
└── pubspec.yaml
```

### Next.js / React Web App

```
app-name/
├── src/
│   ├── app/                   # App Router (Next.js 14+) / pages
│   ├── components/            # Reusable UI components
│   ├── lib/                   # Utilities, helpers, constants
│   ├── services/              # API calls, data fetching
│   ├── hooks/                 # Custom React hooks
│   ├── stores/                # State management (Zustand/Redux)
│   ├── types/                 # TypeScript types/interfaces
│   ├── styles/                # Global styles, Tailwind config
│   └── i18n/                  # Localization files
├── public/                    # Static assets
├── tests/                     # Tests
├── docs/
├── CHANGELOG.md
├── README.md
└── package.json
```

### Django / Python Backend

```
app-name/
├── app_name/                  # Main Django project
│   ├── settings/              # Settings (base, dev, prod)
│   ├── urls.py
│   └── wsgi.py
├── apps/                      # Django apps (feature modules)
│   └── feature_name/
│       ├── models.py
│       ├── views.py
│       ├── serializers.py
│       ├── urls.py
│       ├── services.py        # Business logic layer
│       └── tests/
├── core/                      # Shared: constants, utils, exceptions
├── locale/                    # Localization
├── docs/
├── CHANGELOG.md
├── README.md
├── requirements.txt
└── manage.py
```

### FastAPI / Express Backend

```
app-name/
├── src/
│   ├── routes/                # API route handlers
│   ├── services/              # Business logic
│   ├── models/                # Data models / schemas
│   ├── middleware/             # Auth, logging, error handling
│   ├── config/                # App config, constants, env loader
│   ├── utils/                 # Helpers
│   └── main.py / index.ts     # Entry point
├── tests/
├── docs/
├── CHANGELOG.md
├── README.md
└── .env.example
```

> **Note:** Jika tech stack tidak ada di atas, buat folder structure yang mengikuti konvensi komunitas framework tersebut. Selalu pastikan ada separation of concerns yang jelas.

---

## Constants Convention (Sesuaikan per Stack)

Prinsip utama: **JANGAN hardcode nilai yang bisa berubah.** Semua harus dari constants/config.

Yang harus masuk constants:
- App name, version, description
- Route paths / named routes
- API endpoints / base URL
- Default settings (theme, language, log level)
- Feature flags
- Breakpoints (jika responsive)
- Error messages / status codes

---

## Common Pitfalls

1. **Langsung nulis kode.** STOP. PRD dulu, approval dulu, baru kode.
2. **Tidak tanya tech stack.** Pertanyaan pertama WAJIB menanyakan jenis app + tech stack.
3. **Skip mandatory items.** Semua M1-M25 wajib ada di PRD, diadaptasi sesuai stack.
4. **Hardcode teks UI.** Semua teks statis harus dari localization file.
5. **Hardcode app name/version.** Harus dari constants.
6. **Tidak tanya bahasa localization.** Wajib tanyakan bahasa default (en/id/lainnya).
7. **Tidak buat docs/.** Minimal 4 file docs (api, database, setup, structure). Bukan opsional.
8. **Error tidak ter-log.** Setiap error harus masuk logging system DAN user dapat feedback visual.
9. **Tidak ada services layer.** UI langsung panggil API/DB → sulit maintain. Wajib lewat service.
10. **Tidak ada loading state.** App startup tanpa loading → user lihat blank screen.
11. **Constants tidak sinkron.** Version di constants, package.json/pubspec.yaml, dan config harus sama.
12. **Tidak ada input validation.** Semua input user WAJIB divalidasi — client-side DAN server-side.
13. **Layout tidak responsive.** Harus handle berbagai ukuran layar sesuai platform.
14. **User stories terlalu besar.** Setiap story harus selesai dalam satu sesi. Pecah kalau besar.
15. **Acceptance criteria vague.** "Works correctly" ❌ → "Button shows confirmation before deleting" ✅
16. **Non-goals kosong.** Bagian ini penting untuk mencegah scope creep. Selalu isi.
17. **PRD terlalu teknis.** Fokus ke kebutuhan produk. Detail teknis masuk implementation plan.
18. **Tidak tanya database.** Jika app perlu persistence, tanyakan database preference.
19. **Folder structure tidak sesuai konvensi.** Ikuti konvensi komunitas framework yang dipilih.
20. **Testing strategy diabaikan.** Minimal unit tests untuk business logic harus terdokumentasi.

---

## Verification Checklist

- [ ] STOP — tidak ada kode sebelum PRD disetujui
- [ ] Tech stack & jenis app ditanyakan di pertanyaan pertama
- [ ] Jika Tauri v2 → redirect ke `prd-tauri-v2`
- [ ] Target platform/OS ditanyakan
- [ ] Bahasa localization default ditanyakan
- [ ] Semua M1-M25 tercakup di PRD (diadaptasi sesuai stack)
- [ ] PRD ditulis dalam Bahasa Inggris
- [ ] Komunikasi dengan user dalam Bahasa Indonesia
- [ ] Folder structure sesuai konvensi framework
- [ ] Docs folder tercantum (minimal 4 file)
- [ ] Constants convention tercantum
- [ ] Localization terdefinisi
- [ ] User stories punya acceptance criteria yang verifiable
- [ ] Functional requirements di-number (FR-1, FR-2, ...)
- [ ] Non-goals jelas
- [ ] Testing strategy terdokumentasi
- [ ] State management terdefinisi (jika ada UI)
- [ ] File disimpan ke `tasks/prd-[app-name].md`
- [ ] Minta persetujuan user sebelum implementasi
- [ ] Services layer ada (UI tidak langsung panggil API/DB)
- [ ] Input validation terdefinisi (client + server)
- [ ] Error feedback UI ada (toast/snackbar/alert)
- [ ] Loading state saat startup
- [ ] Responsive/adaptive layout
- [ ] Health check terdefinisi
- [ ] Version pinning / lockfile
