---
name: odysseus-agent
description: "Configure, use, and extend Odysseus AI Agent \u2014 skill management, model routing, session handling, and troubleshooting."
version: 1.0.0
category: general
status: published
confidence: 0.9
source: learned
owner: wayan
created: "2026-07-04T11:57:32Z"
---

# Odysseus Agent

Odysseus adalah AI agent framework buatan Wayan yang berjalan sebagai sistem agen otonom untuk coding, task execution, dan workflow automation. Odysseus memiliki sistem skill yang memungkinkan agen belajar dan menyimpan prosedur reusable untuk tugas-tugas berulang.

- **Skill-based learning** — Odysseus belajar dari pengalaman dengan menyimpan prosedur sebagai skill. Skill terakumulasi seiring waktu, membuat agen semakin baik untuk tugas spesifik.
- **Persistent memory** — mengingat preferensi user, detail environment, dan pelajaran dari sesi sebelumnya.
- **Multi-model support** — bekerja dengan berbagai LLM provider (OpenRouter, Anthropic, OpenAI, DeepSeek, Ollama, LM Studio, dll).
- **Extensible skill system** — skill bisa dibuat, diupdate, dan diorganisir dalam kategori.

---

```
odysseus/
├── data/
│   ├── skills/
│   │   ├── general/          # Skill kategori umum
│   │   │   ├── <nama-skill>/
│   │   │   │   └── SKILL.md  # Definisi skill
│   │   │   └── ...
│   │   └── imported/         # Skill yang diimpor
│   ├── logs/
│   │   └── app.log           # Log aplikasi
│   └── app.db                # Database utama (SQLite)
└── odysseus.log              # Log utama
```

---

### Format Skill

Setiap skill disimpan di `data/skills/general/<nama-skill>/SKILL.md` dengan format:

```yaml
---
name: nama-skill              # Lowercase, hyphens (wajib)
description: "Deskripsi skill" # Penjelasan singkat (wajib)
version: 1.0.0                # Semantic versioning (wajib)
category: general              # Kategori skill (wajib)
status: published                  # draft | active | archived (wajib)
confidence: 0.9                # 0.0-1.0 tingkat kepercayaan (wajib)
source: learned                # learned | manual | imported (wajib)
owner: wayan                   # Pemilik skill (wajib)
created: "2026-07-04T..."      # ISO 8601 timestamp (wajib)
---

# Judul Skill

Konten markdown skill...
```

### Frontmatter Fields

| Field | Type | Required | Keterangan |
|-------|------|----------|------------|
| `name` | string | ✅ | Nama unik skill (lowercase, pakai hyphen) |
| `description` | string | ✅ | Deskripsi singkat fungsi skill |
| `version` | string | ✅ | Versi semantic (major.minor.patch) |
| `category` | string | ✅ | Kategori: `general`, `coding`, `devops`, dll |
| `status` | string | ✅ | Status: `draft`, `active`, `archived` |
| `confidence` | float | ✅ | Tingkat kepercayaan: 0.0 - 1.0 |
| `source` | string | ✅ | Asal: `learned`, `manual`, `imported` |
| `owner` | string | ✅ | Pemilik/pembuat skill |
| `created` | string | ✅ | Tanggal dibuat (ISO 8601) |

### Membuat Skill Baru

1. Buat folder di `data/skills/general/<nama-skill>/`
2. Buat file `SKILL.md` di dalam folder tersebut
3. Tambahkan frontmatter YAML dengan semua field wajib
4. Tulis konten skill dalam markdown

```bash
# Contoh membuat skill baru
mkdir -p /home/yan/odysseus/data/skills/general/nama-skill
# Lalu buat SKILL.md dengan frontmatter + konten
```

### Mengorganisir Skill

- **Gunakan nama yang deskriptif** — `senior-fullstack` bukan `sf`
- **Satu skill per folder** — setiap skill punya folder sendiri
- **Status tracking** — gunakan `status: draft` untuk skill baru, `active` untuk yang sudah teruji, `archived` untuk yang tidak dipakai
- **Confidence scoring** — `0.8` default, naikkan ke `0.9-1.0` setelah skill terbukti efektif

---

Odysseus memiliki koleksi skill di `data/skills/general/`:

| Skill | Deskripsi |
|-------|-----------|
| `senior-fullstack` | Toolkit lengkap untuk fullstack developer senior |
| `prd-architect` | PRD generation dan architecture planning |
| `tauri-expert` | Tauri v2 app development |
| `rust-programming-expert` | Expert Rust programming |
| `python-programming-expert` | Expert Python programming |
| `ui-ux-pro-max` | UI/UX design specialist |
| `tailwind-expert` | Tailwind CSS mastery |
| `supabase-security-expert` | Supabase security hardening |
| `supabase-migration` | Supabase database migration |
| `firebase-security-expert` | Firebase security rules |
| `bun-runtime-expert` | Bun runtime development |
| `tanstack-query-expert` | TanStack Query patterns |
| `seo` | SEO optimization |
| `seo-geo` | Geo-targeted SEO |
| `seo-aeo-landing-page-writer` | SEO/AEO landing page content |
| `e2e-testing-expert` | End-to-end testing |
| `secure-fuzz-testing` | Security fuzz testing |
| `saas-mvp-launcher` | SaaS MVP rapid launch |
| `saas-billing` | SaaS billing integration |
| `saas-multi-tenant` | SaaS multi-tenant architecture |
| `saas-transformer` | SaaS transformation patterns |
| `production-ready-hardener` | Production hardening checklist |
| `scalability-clean-code` | Scalable clean code principles |
| `auto-doc-updater` | Auto-dokumentasi perubahan |
| `brainstorming` | Brainstorming dan ideation |
| `asisten-ramah` | Asisten ramah berbahasa Indonesia |
| `coderabbit` | Code review ala CodeRabbit |
| `token-saver` | Token usage optimization |
| `app-analyzer-optimizer` | App analysis dan optimization |
| `hig` | Human Interface Guidelines |
| `fullstack-expert` | Fullstack expertise |
| `senior-frontend` | Senior frontend development |

---

```
/home/yan/odysseus/                           # Root proyek
/home/yan/odysseus/data/                      # Data directory
/home/yan/odysseus/data/skills/general/       # Skill aktif
/home/yan/odysseus/data/skills/imported/      # Skill yang diimpor
/home/yan/odysseus/data/app.db                # Database SQLite
/home/yan/odysseus/data/logs/app.log          # Application log
/home/yan/odysseus/odysseus.log               # Main log
```

---

### Menulis Skill yang Baik

1. **Trigger conditions** — kapan skill ini harus digunakan
2. **Langkah-langkah bernomor** — dengan command yang exact
3. **Pitfalls section** — kesalahan umum dan cara menghindarinya
4. **Verification steps** — cara memverifikasi hasil
5. **Bilingual** — tulis dalam Bahasa Indonesia dan/atau English

### Konvensi Penamaan

- **Folder skill:** lowercase, gunakan hyphen (`senior-fullstack`, bukan `SeniorFullstack`)
- **File skill:** selalu `SKILL.md` (uppercase)
- **Deskripsi:** singkat tapi informatif, bisa bilingual (ID/EN)

### Maintenance

- Update `version` saat ada perubahan signifikan
- Ubah `status` dari `draft` → `active` setelah skill terbukti efektif
- Naikkan `confidence` berdasarkan keberhasilan penggunaan
- Archive skill yang sudah tidak relevan (`status: archived`)

---

### Skill tidak terbaca
1. Pastikan file bernama `SKILL.md` (case-sensitive)
2. Pastikan frontmatter valid (buka-tutup `---`)
3. Pastikan semua field wajib terisi
4. Cek tidak ada karakter invalid di YAML

### Database issues
1. Cek file `data/app.db` ada dan tidak corrupt
2. Lihat log di `data/logs/app.log` untuk error detail

### Log debugging
```bash
# Lihat log terbaru
tail -50 /home/yan/odysseus/data/logs/app.log

# Cari error spesifik
grep -i "error\|failed" /home/yan/odysseus/odysseus.log | tail -20
```
