---
name: ubuntu-gnome-dev
description: "Panduan development dan kustomisasi desktop pada Ubuntu 26.04 LTS dengan GNOME & Wayland. PC: Lenovo IdeaPad Gaming 3 (Ryzen 5 5600H, GPU Radeon Cezanne/Vega, 12GB RAM). Workspace dev di /home/yan/Dev/. Integrasi Obsidian YanVault untuk ringkasan/catatan debugging."
version: 1.0.0
category: general
status: published
confidence: 0.9
source: learned
owner: wayan
created: "2026-07-04T11:57:32Z"
---

# Ubuntu GNOME Dev Environment

Skill ini mengatur standar environment dan setup proyek di sistem operasi **Ubuntu 26.04 LTS (Resolute Raccoon)** dengan desktop environment **GNOME** dan display server **Wayland** pada hardware **Lenovo IdeaPad Gaming 3 15ACH6** (AMD Ryzen 5 5600H, AMD Radeon Vega/Cezanne Graphics, 12GB RAM, 35GB Swap).

## Core Principles

1. **Workspace Development**
   - Semua aplikasi, kode, repositori, dan proyek development baru **WAJIB** dibuat di dalam folder `/home/yan/Dev/`.
   - Contoh: `/home/yan/Dev/my-new-app/`.
   
2. **Dokumentasi & Ringkasan Obsidian (YanVault)**
   - Setiap kali menyelesaikan task coding, debugging, setup sistem, troubleshooting, atau konfigurasi desktop, buat ringkasan (summary) dalam bentuk catatan Obsidian Markdown (.md).
   - Catatan disimpan di `/home/yan/obsidian/YanVault/` di subfolder yang sesuai (misal: `/home/yan/obsidian/YanVault/ubuntu/` atau `/home/yan/obsidian/YanVault/hermes/`).
   - Setiap selesai membuat/menyunting catatan, **WAJIB** lakukan commit dan push pada repositori git vault:
     ```bash
     cd /home/yan/obsidian/YanVault
     git add .
     git commit -m "docs: sync note <nama-note>"
     git push origin master
     ```

---

## obsidian-vault: Standard Formatting

Setiap catatan Obsidian yang dibuat harus mengikuti format frontmatter dan struktur di bawah:

```yaml
---
tags: [tag1, tag2, category]   # Huruf kecil, comma-separated
category: Nama Kategori        # Contoh: Ubuntu Desktop, Hermes, Rust
date: YYYY-MM-DD              # Tanggal pembuatan
status: solved                # solved | open | progress
---

# Judul Catatan

**Problem:** Deskripsi singkat masalah/tujuan  
**Status:** ✅ SOLVED / 🛠️ IN PROGRESS  
**Related:** [[CatatanTerkait1]], [[CatatanTerkait2]]

---

## 🔍 Gejala
- Detail apa yang terjadi (output log, perilaku GUI)

## 🛠️ Akar Masalah
- Analisis teknis mengapa error/kebutuhan itu muncul

## ✅ Solusi
- Step-by-step penyelesaian (command exact, konfigurasi file)

## 🧪 Verifikasi
- Cara memastikan solusi berhasil berjalan

## 📚 Pelajaran Penting
- Pola/pelajaran yang didapat agar tidak diulang di masa depan

## 📁 File yang Dimodifikasi
- Daftar file yang diedit/dihapus/dibuat beserta keterangannya

#tag1 #tag2 #category
```

### Aturan Visual Catatan
- Gunakan link wikilink `[[Nama Catatan]]` untuk menghubungkan catatan yang relevan.
- Syntax resize gambar: `![Deskripsi|400](path/to/image.jpg)` (format pixel width, cocok dengan Local Image Plus).
- Selalu gunakan emoji visual untuk struktur (🔍, 🛠️, ✅, 🧪, 📚, 📁).

---

## Spesifikasi System & Hardware

Gunakan informasi spesifikasi ini saat memecahkan masalah performa, driver, rendering, display, atau swap space:

- **OS:** Ubuntu 26.04 LTS (Resolute Raccoon) - Codename: `resolute`
- **CPU:** AMD Ryzen 5 5600H (6 Cores, 12 Threads)
- **RAM:** 12 GiB DDR4
- **Swap:** 35 GiB (Sangat cukup untuk caching/compilation besar)
- **Graphics:** AMD Radeon Vega / Cezanne Mobile Series (integrated/APU)
- **Display Server:** Wayland (default Ubuntu 26.04 GNOME)

---

## GNOME & Wayland Development Conventions

### 1. GUI Apps PATH Fix (GNOME Show Applications)
Jika aplikasi GUI gagal terbuka lewat GNOME launcher tetapi jalan di terminal (karena runtime manager seperti NVM, Pyenv, dll), ikuti perbaikan standard:
- Buat shell wrapper di `~/.local/bin/<app-name>` untuk mengekspor PATH runtime secara hardcoded (jangan pakai ekspansi shell `~` karena desktop manager tidak membacanya).
- Set `StartupWMClass=<Name>` di `.desktop` file agar GNOME mencocokkan window dengan ikon dock secara presisi.
- Refresh database desktop: `update-desktop-database ~/.local/share/applications`

### 2. Wayland Compatibility
- Aplikasi desktop GUI baru (Tauri, Electron) harus di-launch dengan parameter pendukung Wayland jika diperlukan (misal `--ozone-platform-hint=auto` pada Chromium-based apps) untuk menghindari blurry screen pada scaling UI.
- Tauri v2 apps: Capability permissions harus dikonfigurasi dengan tepat di folder `capabilities/`.

### 3. GNOME Extensions & Dock
- Dock didesain Windows-style (full width, black solid background, centered icons):
  - `extend-height=true`
  - `background-color='rgba(0,0,0,1)'`
  - `transparency-mode=FIXED`
  - `always-center-icons=true`
- Jangan jalankan perintah shutdown/reboot dari terminal agen. Jika sistem perlu restart (misal setelah instalasi driver/kernel), instruksikan user untuk melakukannya secara manual.

---

## Langkah Membuat Project Baru di `/home/yan/Dev`

1. Buat folder project di `/home/yan/Dev/<project-name>`:
   ```bash
   mkdir -p /home/yan/Dev/my-app
   cd /home/yan/Dev/my-app
   ```
2. Inisiasi git repository secara lokal.
3. Buat file `.env.example` jika membutuhkan environment variables.
4. Gunakan `prd-architect` untuk membuat PRD global sebelum menulis kode apa pun.
5. Selesai pengerjaan project, jika ada konfigurasi khusus Ubuntu/GNOME yang diterapkan, catat ringkasannya ke Obsidian YanVault dan push perubahan git.
