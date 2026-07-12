---
name: token-saver
description: "Skill penghematan token komprehensif \\u2014 8 rules sistematis untuk mengurangi penggunaan token di Input, Output, dan Context. 3 mode: lite (~65%), full (~80%), ultra (~87% savings). Bilingual ID/EN."
version: 2.0.0
category: general
status: published
confidence: 0.9
source: learned
owner: wayan
created: "2026-07-04T11:57:32Z"
---

# Token Saver 🪙

> **87% average token savings** — 8 rules, zero dependencies.

Sebut salah satu di chat untuk ganti mode:

| Command | Savings | Keterangan |
|---------|---------|------------|
| `token-saver lite` | ~65% | Intervensi minimal — reply agak ringkas, grep before read |
| `token-saver full` | ~80% | Balanced, **default** — semua 8 rules aktif |
| `token-saver ultra` | ~87% | Maximum compression — ultra terse, aggressive delegation |
| `token-saver off` | — | Nonaktifkan |

---

### 🟢 INPUT: Jangan load yang tidak perlu

**R1 — SubAgent untuk Eksplorasi (>3 files)**
Saat mencari/eksplorasi lintas banyak file (3+), delegasikan ke subagent. Session utama hanya terima ringkasan akhir.
- ✅ Kirim subagent dengan instruksi: *"Report findings in under 200 words. Show only file paths and key conclusions."*
- ❌ Jangan baca 10+ file satu per satu di session utama

**R2 — Grep Before Read**
File >300 baris: cari dulu target symbol/function, baru baca dengan offset & limit hanya baris relevan (≤30).
- ✅ Search pattern dulu → baca file dengan offset & limit=30
- ✅ File <300 baris dengan konteks jelas → langsung baca
- ❌ Jangan baca file 1000 baris tanpa filter dulu

**R3 — Batch Independent Calls**
Kirim 2+ tool calls independen dalam 1 pesan. Jangan serialisasi tugas yang bisa paralel.
- ✅ Search + web fetch + terminal dalam 1 langkah kalau tidak dependen
- ❌ Jangan nunggu hasil A dulu baru mulai B kalau tidak ada dependensi

**R4 — Never Search the Same Thing Twice**
Mental note: sudah search apa saja di sesi ini. Langsung refer, jangan search ulang.
- ✅ Kalau sudah dapat file path dari search sebelumnya, langsung baca saja
- ❌ Jangan search lagi untuk pattern yang sama

### 🔵 OUTPUT: Jangan bilang yang tidak penting

**R5 — Filter Bash Output**
Pipe perintah verbose:
- `npm install`, `cargo build`, `pip install` → `2>&1 | tail -20`
- `pytest`, `go test` → `2>&1 | grep -E "PASSED|FAILED|ERROR"` atau `tail -20`
- Success: last **5 lines**. Failure: last **50 lines**.
- `git log`, `git diff --stat` → `| head -30`
- ❌ Jangan biarkan output 500 baris mentah masuk context

**R6 — Terse Replies (≤200 words)**
Setiap reply default ≤200 kata. Lead with jawaban, bukan penjelasan.
- ✅ Langsung ke inti
- ✅ Code blocks **tidak dihitung** dalam word limit
- ❌ No pleasantries: *"Tentu, dengan senang hati!"*, *"Baik, saya akan bantu..."*
- ❌ No summaries of what you just did before showing the result
- ℹ️ Kalau user minta penjelasan detail → pakai judgment, boleh panjang

### 🟠 CONTEXT: Bersihkan sebelum keburu

**R7 — Compact After Large Reads**
Jika baca file atau output terminal >300 baris atau >30KB, kompakkan segera:
- Jangan simpan data mentah di pikiran
- Ekstrak hanya baris/esensi yang relevan untuk langkah berikutnya
- ❌ Jangan teruskan beban konteks yang tidak perlu

**R8 — Limit SubAgent Output**
Setiap delegasi ke subagent harus include instruksi:
- *"Report findings in under 200 words. Show only file paths and key conclusions. Omit raw data."*
- SubAgent adalah alat, bukan narrator — jangan minta narasi panjang

---

```
R1  SubAgent eksplorasi (>3 files)       → delegate/subagent
R2  Grep sebelum baca (>300 lines)       → search → read(limit=30)
R3  Batch independent calls              → parallel tool calls
R4  Jangan search ulang                  → mental note
R5  Filter bash output                   → tail/grep/head
R6  Terse replies ≤200 words             → langsung ke inti
R7  Compact setelah large reads          → ekstrak esensi saja
R8  Limit SubAgent output               → "≤200 words, key findings only"
```

- **JANGAN** tulis ulang (rewrite) seluruh file jika hanya perubahan kecil.
- Prioritaskan editing presisi (patch/replace) alih-alih tulis ulang file.
- Saat share cuplikan kode, berikan **hanya** blok yang berubah — hindari menulis ulang kode yang tidak berubah.

Aktifkan skill ini ketika:
- User meminta "hemat token", "ringkas", "concise", "save tokens"
- Konteks token sudah hampir penuh
- Melakukan refactoring kecil di banyak file secara beruntun
- Project besar dengan banyak file

Skill aktif otomatis setelah di-load. Ganti mode kapan saja dengan mention di chat: `token-saver ultra`.
