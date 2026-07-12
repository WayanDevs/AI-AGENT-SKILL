---
name: token-budget-advisor
description: "Intercept response flow untuk menawarkan pilihan kedalaman jawaban (25/50/75/100%) SEBELUM menjawab. User kontrol seberapa detail respons yang diterima. Heuristic token estimation, depth levels, session persistence."
version: 1.0.0
category: general
status: published
confidence: 0.9
source: learned
owner: wayan
created: "2026-07-04T11:57:32Z"
---

## When to Use

- User ingin kontrol seberapa panjang/detail respons
- User sebut: tokens, budget, depth, response length
- User bilang: "versi singkat", "tldr", "ringkas", "detail", "lengkap", "short version", "brief", "al 25%", "exhaustive"
- User ingin pilih level kedalaman jawaban di awal

**Jangan trigger ketika:**
- User sudah set level di sesi ini (pertahankan diam-diam)
- Jawaban trivial satu baris
- "Token" merujuk ke auth/session/payment token, bukan ukuran respons

# Token Budget Advisor (TBA)

Intercept response flow untuk menawarkan pilihan kedalaman jawaban **sebelum** menjawab.

### Step 1 — Estimasi input tokens

Heuristic estimation (tanpa tokenizer):

- Prosa: `words × 1.3`
- Kode/campuran kode: `chars / 4`

Untuk konten campuran, gunakan tipe konten dominan.

### Step 2 — Estimasi ukuran respons berdasarkan kompleksitas

| Complexity | Multiplier | Contoh prompt |
|------------|-----------|---------------|
| Simple | 3× – 8× | "Apa itu X?", ya/tidak, satu fakta |
| Medium | 8× – 20× | "Bagaimana X bekerja?" |
| Medium-High | 10× – 25× | Request kode dengan konteks |
| Complex | 15× – 40× | Analisis multi-bagian, perbandingan, arsitektur |
| Creative | 10× – 30× | Cerita, essay, narasi |

Response window = `input_tokens × mult_min` sampai `input_tokens × mult_max` (tidak melebihi output-token limit model).

### Step 3 — Presentasikan pilihan depth

Tampilkan blok ini **sebelum** menjawab, dengan angka estimasi aktual:

```
Menganalisis prompt Anda...

Input: ~[N] tokens  |  Tipe: [type]  |  Kompleksitas: [level]

Pilih level kedalaman:

[1] Essential   (25%)  ->  ~[tokens]   Jawaban langsung saja, tanpa basa-basi
[2] Moderate    (50%)  ->  ~[tokens]   Jawaban + konteks + 1 contoh
[3] Detailed    (75%)  ->  ~[tokens]   Jawaban lengkap dengan alternatif
[4] Exhaustive (100%)  ->  ~[tokens]   Semua hal, tanpa batasan

Level berapa? (1-4 atau bilang "25%", "50%", "75%", "100%")

Presisi: estimasi heuristik ~85-90% akurasi (±15%).
```

Estimasi token per level (dalam response window):
- 25% → `min + (max - min) × 0.25`
- 50% → `min + (max - min) × 0.50`
- 75% → `min + (max - min) × 0.75`
- 100% → `max`

### Step 4 — Jawab sesuai level yang dipilih

| Level | Target panjang | Include | Omit |
|-------|---------------|---------|------|
| 25% Essential | 2-4 kalimat maks | Jawaban langsung, kesimpulan utama | Konteks, contoh, nuansa, alternatif |
| 50% Moderate | 1-3 paragraf | Jawaban + konteks + 1 contoh | Analisis mendalam, edge cases, referensi |
| 75% Detailed | Respons terstruktur | Banyak contoh, pro/kontra, alternatif | Extreme edge cases, referensi exhaustive |
| 100% Exhaustive | Tanpa batasan | Semuanya — analisis penuh, semua kode, semua perspektif | Tidak ada |

Jika user sudah sinyal level, langsung jawab di level itu tanpa bertanya:

| Yang user bilang | Level |
|------------------|-------|
| "1" / "25%" / "versi singkat" / "ringkas" / "tldr" / "brief" | 25% |
| "2" / "50%" / "moderate" / "balanced" / "seimbang" | 50% |
| "3" / "75%" / "detail" / "detailed" / "thorough" / "lengkap" | 75% |
| "4" / "100%" / "exhaustive" / "full" / "semua" / "tanpa batas" | 100% |

Jika user sudah set level sebelumnya di sesi ini, **pertahankan diam-diam** untuk respons berikutnya kecuali user ganti.

Skill ini menggunakan estimasi heuristik — tanpa tokenizer asli. Akurasi ~85-90%, variansi ±15%. Selalu tampilkan disclaimer.

### Triggers ✅

- "Kasih versi singkat dulu."
- "Berapa token yang akan kamu pakai?"
- "Jawab di 50% depth."
- "Saya mau jawaban exhaustive, bukan summary."
- "Give me the short version first."
- "Respond at 75% depth."

### Does Not Trigger ❌

- "Apa itu JWT token?"
- "Checkout flow pakai payment token."
- "Apakah ini normal?"
- "Lanjutkan refactoring."
- Follow-up questions setelah user sudah pilih depth untuk sesi ini

- **`token-saver`** → mengatur CARA agen bekerja (hemat input/output/context)
- **`token-budget-advisor`** → mengatur SEBERAPA DALAM jawaban yang diberikan (depth level)
- Keduanya bisa aktif bersamaan — `token-saver` hemat proses internal, `token-budget-advisor` kontrol kedalaman output untuk user.
