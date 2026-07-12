---
name: auto-doc-updater
description: Otomatis mendokumentasikan setiap perubahan fitur atau perbaikan bug yang berhasil di-build ke CHANGELOG.md dan BLUEPRINT.md.
author: Roedy Rustam
tags: [documentation, changelog, blueprint, automation]
---

# Auto Documentation Updater 📝

> Dokumentasi proyek selalu sinkron dengan kode — tanpa diminta.

## Kondisi Pemicu

Skill ini aktif **secara otomatis** setelah:
- Berhasil melakukan perubahan kode (fitur baru, bug fix, refactor)
- Build berhasil diverifikasi (misal `npm run build`, `cargo build` sukses tanpa error)

---

## Protokol

Setiap siklus perubahan kode berhasil diverifikasi:

### Langkah 1 — Identifikasi Perubahan
Rangkum perubahan secara teknis dan fungsional:
- Fitur/modul apa yang diubah/ditambah
- Issue apa yang diperbaiki
- Breaking changes (jika ada)

### Langkah 2 — Update CHANGELOG.md

Format:

```markdown
# Changelog

## [vX.Y.Z] - YYYY-MM-DD

### Added
- Fitur baru A

### Changed
- Perubahan pada B

### Fixed
- Bug fix untuk C

### Removed
- Penghapusan D
```

- Jika versi untuk hari ini sudah ada, tambahkan poin di kategori yang sesuai
- Jika belum ada, buat blok baru (increment `patch` untuk bug fix, `minor` untuk fitur baru, `major` untuk breaking changes)
- Gunakan semantic versioning (`vMAJOR.MINOR.PATCH`)

### Langkah 3 — Update BLUEPRINT.md

Update metadata di frontmatter:

```yaml
---
title: Project Blueprint
version: vX.Y.Z          # ← sinkron dengan CHANGELOG
last_updated: YYYY-MM-DD # ← hari ini
---
```

Tambahkan poin ke bagian terkait:
- **Feature Modules** — jika ada penambahan modul/fitur besar
- **Architecture** — jika ada perubahan arsitektur

### Langkah 4 — Gunakan patch
Gunakan `patch()` untuk update CHANGELOG.md dan BLUEPRINT.md — jangan overwrite file penuh.

### Langkah 5 — Konfirmasi
Sampaikan ke user: "CHANGELOG dan BLUEPRINT telah diperbarui secara otomatis."

---

## Aturan

- ✅ Hanya dokumentasikan **hasil sukses** (build sukses, test passing)
- ✅ Gunakan bahasa profesional deskriptif layaknya Release Notes
- ❌ Jangan dokumentasikan error/sedang dalam proses perbaikan
- ❌ Jangan merusak struktur file yang sudah ada
- ✅ Selalu cek apakah file sudah ada sebelum membuat baru

---

## File yang Dikelola

| File | Format | Wajib? |
|------|--------|--------|
| `CHANGELOG.md` | Markdown, reverse-chronological | Ya |
| `BLUEPRINT.md` | Markdown + YAML frontmatter | Jika ada |

---

## Contoh Output

```markdown
# Changelog

## [v1.2.0] - 2026-06-13

### Added
- Fitur autentikasi OAuth2 dengan Google dan GitHub
- Upload file drag-and-drop pada dashboard

### Fixed
- [SECURITY] XSS vulnerability pada input nama pengguna
- Memory leak pada WebSocket connection pool
```
