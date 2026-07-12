---
name: token-saver
description: >-
  8 rules to reduce token usage across Input, Output, and Context layers.
  Adaptasi dari LG-token-saver v3.0 oleh LiaoGong/CC杰 untuk Hermes Agent.
  Mode: lite (~65%), full (~80%), ultra (~87% savings).
tags: [tokens, optimization, cost-saving, efficiency]
category: software-development
version: 1.0
author: Adapted from LG-token-saver v3.0
---

# Token Saver 🪙

> **87% average token savings** — 8 rules, zero dependencies, Hermes Agent.

## Mode Switching

Sebut salah satu di chat untuk ganti mode:

| Command | Savings | Role |
|---------|---------|------|
| `token-saver lite` | ~65% | Minimal intervention |
| `token-saver full` | ~80% | Balanced, **default** |
| `token-saver ultra` | ~87% | Maximum compression |
| `token-saver off` | — | Disable |

---

## The 8 Rules

### 🟢 INPUT: Jangan load yang tidak perlu

**R1 — SubAgent untuk Eksplorasi (>3 files)**
Saat mencari/eksplorasi lintas banyak file (3+), selalu kirim `delegate_task` untuk eksplorasi. Session utama hanya terima ringkasan akhir saja.
- ✅ Gunakan `delegate_task(goal="...", context="...", toolsets=["terminal", "file"])`
- ✅ Include di context: *"Report findings in under 200 words. Show only file paths and key conclusions. Omit raw data."*
- ❌ Jangan baca 10+ file satu per satu di session utama

**R2 — Grep Before Read**
File >300 baris: `search_files` dulu untuk target symbol/function, baru `read_file` dengan offset & limit hanya baris relevan (≤30).
- ✅ `search_files(pattern="function_name", path="src/")` → `read_file(path, offset=N, limit=30)`
- ✅ File <300 baris dengan konteks jelas → langsung baca
- ❌ Jangan `read_file` file 1000 baris tanpa filter dulu

**R3 — Batch Independent Calls**
Kirim 2+ tool calls independen dalam 1 pesan. Jangan serialisasi tugas yang bisa paralel.
- ✅ `search_files` + `web_search` + `terminal` dalam 1 langkah
- ✅ `delegate_task(tasks=[...])` untuk batch paralel (max 3 concurrent)
- ❌ Jangan nunggu hasil A dulu baru mulai B kalau gak dependen

**R4 — Never Search the Same Thing Twice**
Mental note: sudah search apa saja di sesi ini. Langsung refer, jangan search ulang.
- ✅ Kalau sudah dapat file path dari search sebelumnya, langsung `read_file` saja
- ❌ Jangan `search_files` lagi untuk pattern yang sama

### 🔵 OUTPUT: Jangan bilang yang tidak penting

**R5 — Filter Bash Output**
Pipe perintah verbose:
- `npm install`, `cargo build`, `make`, `pip install` → `2>&1 | tail -20`
- `pytest`, `go test` → `2>&1 | grep -E "PASSED|FAILED|ERROR|FAIL"` atau `tail -20`
- Success: last **5 lines**. Failure: last **50 lines**.
- `git log`, `git diff --stat` → `| head -30`
- ❌ Jangan biarkan output 500 baris mentah masuk context

**R6 — Terse Replies (≤200 words)**
Setiap reply default ≤200 kata. Lead with jawaban, bukan penjelasan.
- ✅ Langsung ke inti
- ✅ Code blocks **tidak dihitung** dalam word limit
- ❌ No pleasantries: *"Tentu, dengan senang hati!"*, *"Baik, saya akan bantu..."*
- ❌ No summaries of what you just did before showing the result
- ℹ️ Berbeda dengan user yang minta detailed explanation — pakai judgment

### 🟠 CONTEXT: Bersihkan sebelum keburu

**R7 — Compact After Large Reads**
Jika `read_file` atau output terminal >300 baris atau >30KB, kompakkan segera:
- Jangan simpan data mentah di pikiran
- Ekstrak hanya baris/esensi yang relevan untuk langkah berikutnya
- ❌ Jangan teruskan beban konteks yang tidak perlu

**R8 — Limit SubAgent Output**
Setiap `delegate_task` harus include di context:
- *"Report findings in under 200 words. Show only file paths and key conclusions. Omit raw data."*
- SubAgent adalah alat, bukan diarist — jangan minta narasi panjang

---

## Quick Reference Card

```
R1  SubAgent eksplorasi (>3 files)      → delegate_task
R2  Grep sebelum baca (>300 lines)       → search_files → read_file(limit=30)
R3  Batch independent calls              → parallel tool calls
R4  Jangan search ulang                  → mental note
R5  Filter bash output                   → tail/grep/head
R6  Terse replies ≤200 words             → langsung ke inti
R7  Compact setelah large reads          → ekstrak esensi saja
R8  Limit SubAgent output               → "≤200 words, key findings only"
```

## Usage

Skill ini aktif otomatis setelah di-load dengan `skill_view(name='token-saver')`. 
Ganti mode kapan saja dengan mention di chat: `token-saver ultra`.
