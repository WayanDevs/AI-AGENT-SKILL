---
name: prd
description: "PRD gatekeeper & generator. Pemicu Bahasa Indonesia: buat aplikasi baru, bikin PRD, tulis PRD untuk, rencanakan fitur, buat proyek/SaaS/MVP baru, specs/requirements. Gatekeeper: jangan nulis kode sebelum PRD disetujui user. Tanya klarifikasi dgn opsi A/B/C/D. Output PRD bahasa Inggris standar."
version: 1.0.0
author: User
license: MIT
user-invocable: true
metadata:
  hermes:
    tags: [prd, product-requirements, planning, project-initiation, gatekeeper]
    related_skills: [plan, spike, architecture-diagram]
---

# PRD Gatekeeper & Generator

## Overview

Skill ini bertindak sebagai **palang pintu (gatekeeper)** setiap kali ada inisiatif untuk membangun aplikasi, fitur, atau proyek baru dari nol — **tidak ada kode yang boleh ditulis sebelum Product Requirements Document (PRD) disetujui oleh user.**

Skill ini juga **menghasilkan PRD yang terstruktur, jelas, dan siap pakai** — diawali dengan pertanyaan klarifikasi berformat pilihan ganda (A/B/C/D) untuk mempercepat proses, lalu menghasilkan dokumen PRD lengkap.

## When to Use

**Pemicu (trigger) — dalam Bahasa Indonesia:**

- "buat aplikasi baru"
- "bikin PRD" / "buat PRD" / "tulis PRD"
- "tulis PRD untuk [fitur]"
- "rencanakan fitur [nama]"
- "requirements for [feature]" / "spec out [feature]"
- "buat proyek baru" / "proyek baru dari nol"
- "buat SaaS" / "SaaS baru"
- "buat MVP" / "MVP baru"
- "rencanakan [sesuatu]" (untuk inisiatif besar)
- User meminta agent menjadi Product Manager (PM)
- User bilang "gas bikin [app/fitur]" tanpa spesifikasi detail

**Jangan gunakan untuk:**
- Perbaikan bug kecil atau task teknis yang sudah jelas scope-nya
- Refactoring kode yang tidak mengubah behaviour
- Task dengan spesifikasi yang sudah sangat detail dari user

## Workflow

### ⛔ Step 0: GATEKEEPER — STOP. JANGAN NULIS KODE.

Begitu terpicu, **hentikan** semua pikiran tentang kode, struktur folder, arsitektur teknis, atau implementasi. Tugas pertama Anda adalah **murni analisis bisnis dan kebutuhan produk.**

### ❓ Step 1: Clarifying Questions (dalam Bahasa Indonesia)

Ajukan **3-5 pertanyaan klarifikasi** dengan opsi berhuruf (A/B/C/D) agar user bisa jawab cepat seperti "1A, 2C, 3B".

Tanyakan **hanya** yang benar-benar perlu klarifikasi — jangan tanyakan yang sudah jelas dari prompt user.

**Penting untuk Pembuatan Aplikasi Baru:**
- **Wajib menanyakan bahasa lokalisasi (localization)** yang ingin digunakan (misal: `id.json` untuk Bahasa Indonesia, `en.json` untuk Bahasa Inggris, atau lainnya).
- Jelaskan bahwa semua teks UI yang bersifat statis/tetap (seperti menu "home", "setting", atau "menu") harus dikelompokkan ke dalam folder `localization/` (misal: `localization/id.json`).

**Format pertanyaan:**

1. Apa tujuan utama dari fitur/proyek ini?
   A. [opsi 1]
   B. [opsi 2]
   C. [opsi 3]
   D. Lainnya: [sebutkan]

2. Siapa target pengguna utamanya?
   A. Pengguna baru
   B. Pengguna existing
   C. Semua pengguna
   D. Admin/internal saja

3. Apa scope yang diinginkan?
   A. Versi minimal (MVP)
   B. Implementasi penuh
   C. Backend/API saja
   D. UI saja

4. Bagaimana kriteria suksesnya?
   A. [opsi terkait fungsional]
   B. [opsi terkait performa]
   C. [opsi terkait user satisfaction]
   D. Lainnya: [sebutkan]

### 📝 Step 2: Generate PRD (dalam Bahasa Inggris — standar industri)

Berdasarkan jawaban user, hasilkan PRD dengan struktur di bawah. Simpan ke file `tasks/prd-[feature-name].md`.

### ✅ Step 3: Minta Persetujuan User (dalam Bahasa Indonesia)

Tanyakan ke user:

> "Apakah Anda setuju dengan spesifikasi PRD ini? Adakah yang ingin ditambahi/dikurangi sebelum kita lanjut ke arsitektur dan penulisan kode?"

### 🚀 Step 4: Implementation Plan

**Hanya** setelah user setuju ("Ya" / "Setuju"), barulah lanjut ke:
1. Implementation Plan (task list)
2. Arsitektur teknis
3. Penulisan kode

## PRD Structure (output dalam Bahasa Inggris)

### 1. Introduction / Overview
Brief description of the feature/product and the problem it solves.

### 2. Goals
Specific, measurable objectives (bullet list).

### 3. Localization Requirements (Wajib untuk Aplikasi Baru)
- Specify the target language files (e.g., `localization/en.json`, `localization/id.json`).
- Outline that all static/fixed UI strings (menus, settings, buttons, labels) must be managed inside the `localization` directory.

### 4. User Stories
Each story MUST include:
- **Title:** Short descriptive name
- **Description:** "As a [user], I want [feature] so that [benefit]"
- **Acceptance Criteria:** Verifiable checklist of what "done" means

Format:

```
### US-001: [Title]
**Description:** As a [user], I want [feature] so that [benefit].

**Acceptance Criteria:**
- [ ] Specific verifiable criterion
- [ ] Another criterion
- [ ] Typecheck/lint passes
- [ ] **[UI stories only]** Verify in browser using dev-browser skill
```

Rules:
- Each story must be small enough for one focused implementation session.
- Acceptance criteria must be **verifiable**, not vague. "Works correctly" is bad. "Button shows confirmation dialog before deleting" is good.
- For UI stories: always include "Verify in browser using dev-browser skill" as acceptance criteria.

### 4. Functional Requirements
Numbered list of specific functionalities:
- FR-1: The system must ...
- FR-2: When a user clicks X, the system must ...

### 5. Non-Goals (Out of Scope)
What this feature will NOT include — critical for managing scope.

### 6. Design Considerations (Optional)
- UI/UX requirements
- Link to mockups if available
- Existing components to reuse

### 7. Technical Considerations (Optional)
- Known constraints or dependencies
- Integration points
- Performance requirements

### 8. Success Metrics
How will success be measured? Examples:
- "Reduce time to complete X by 50%"
- "Increase conversion rate by 10%"

### 9. Open Questions
Remaining questions or areas needing clarification.

## Writing Standards

- PRD ditulis dalam **Bahasa Inggris** (standar industri)
- Komunikasi dengan user (klarifikasi, approval) dalam **Bahasa Indonesia**
- Be explicit and unambiguous
- Avoid jargon or explain it
- Number requirements for easy reference
- Use concrete examples where helpful
- Assume reader may be junior developer or AI agent

## Output Format

- **Format:** Markdown (.md)
- **Location:** `tasks/`
- **Filename:** `prd-[feature-name].md` (kebab-case)
- Content in **English**

## Common Pitfalls

1. **Langsung mikir kode.** Ingat: tugas pertama adalah analisis kebutuhan, bukan implementasi. Tahan diri.
2. **Tidak minta approval.** Wajib tanya "Setuju?" sebelum lanjut ke implementasi. Ini gatekeeper-nya.
3. **User stories terlalu besar.** Setiap story harus selesai dalam satu sesi implementasi fokus. Jika terlalu besar, pecah.
4. **Acceptance criteria vague.** "Works correctly" ❌ → "Button shows confirmation before deleting" ✅
5. **Melewatkan Non-Goals.** Bagian ini penting untuk mencegah scope creep. Selalu isi.
6. **Bertanya terlalu banyak.** Hanya tanya 3-5 pertanyaan yang benar-benar perlu diklarifikasi. Jangan tanya yang sudah jelas dari prompt awal user.
7. **PRD terlalu teknis.** Fokus ke kebutuhan produk, bukan detail implementasi. Detail teknis masuk ke implementation plan setelah PRD disetujui.

## Verification Checklist

- [ ] STOP — jangan menulis kode sebelum PRD disetujui
- [ ] Ajukan 3-5 pertanyaan klarifikasi dengan opsi A/B/C/D
- [ ] Hanya tanya yang benar-benar perlu klarifikasi (termasuk bahasa lokalisasi untuk aplikasi baru)
- [ ] PRD ditulis dalam Bahasa Inggris
- [ ] PRD mencakup Localization Requirements (misal folder `localization/` berisi `id.json` atau `en.json`) untuk aplikasi baru
- [ ] Setiap User Story memiliki Acceptance Criteria yang verifiable
- [ ] Functional Requirements di-number (FR-1, FR-2, ...)
- [ ] Non-Goals jelas mendefinisikan batasan scope
- [ ] Bagian Open Questions terisi (jika ada)
- [ ] Minta persetujuan user
- [ ] Baru lanjut ke implementation plan setelah user setuju
- [ ] File disimpan ke `tasks/prd-[feature-name].md`
