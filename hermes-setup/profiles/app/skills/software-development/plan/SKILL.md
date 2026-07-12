---
name: plan
description: "Plan mode: tulis implementation plan yang actionable sebelum coding. Bite-sized tasks (2-5 min), exact file paths, complete code, TDD cycle, verification steps. Tidak boleh eksekusi \u2014 hanya planning."
version: 1.0.0
category: general
status: published
confidence: 0.9
source: learned
owner: wayan
created: "2026-07-04T11:57:32Z"
---

## When to Use

**Selalu gunakan sebelum:**
- Implementasi fitur multi-step
- Breakdown requirements kompleks
- Delegasi ke subagent

**Jangan skip meskipun:**
- Fitur terlihat sederhana (asumsi menyebabkan bug)
- Kamu yang akan implementasi sendiri (future you butuh panduan)
- Kerja sendiri (dokumentasi tetap penting)

# Plan Mode

Gunakan skill ini ketika user ingin **plan dulu sebelum eksekusi**.

Untuk turn ini, kamu **hanya merencanakan**.

- ❌ Jangan implementasi kode
- ❌ Jangan edit file project kecuali file plan itu sendiri
- ❌ Jangan jalankan mutating terminal commands, commit, push
- ✅ Boleh inspeksi repo/codebase dengan read-only commands
- ✅ Deliverable: markdown plan yang disimpan ke folder `tasks/` atau `plans/`

Tulis markdown plan yang konkret dan actionable. Include:
- **Goal** — satu kalimat apa yang dibangun
- **Current context / assumptions**
- **Proposed approach** — arsitektur, 2-3 kalimat
- **Step-by-step plan** — bite-sized tasks
- **Files likely to change** — exact paths
- **Tests / validation** — cara verifikasi
- **Risks, tradeoffs, open questions**

Simpan plan ke:
- `tasks/plan-<slug>.md` atau `plans/YYYY-MM-DD_HHMMSS-<slug>.md`

**Setiap task = 2-5 menit focused work.**

Setiap step adalah satu aksi:
- "Tulis failing test" — step
- "Jalankan test untuk pastikan fail" — step
- "Implementasi kode minimal agar test pass" — step
- "Jalankan test, pastikan pass" — step
- "Commit" — step

**Terlalu besar:**
```markdown
### Task 1: Build authentication system
[50 baris kode di 5 file]
```

**Ukuran tepat:**
```markdown
### Task 1: Create User model with email field
[10 baris, 1 file]

### Task 2: Add password hash field to User
[8 baris, 1 file]

### Task 3: Create password hashing utility
[15 baris, 1 file]
```

### Header (Wajib)

```markdown
# [Feature Name] Implementation Plan

**Goal:** [Satu kalimat apa yang dibangun]
**Architecture:** [2-3 kalimat tentang pendekatan]
**Tech Stack:** [Key technologies/libraries]

---
```

### Task Structure

````markdown
### Task N: [Descriptive Name]

**Objective:** Apa yang dicapai task ini (satu kalimat)

**Files:**
- Create: `exact/path/to/new_file.py`
- Modify: `exact/path/to/existing.py:45-67`
- Test: `tests/path/to/test_file.py`

**Step 1: Write failing test**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

**Step 2: Run test to verify failure**

Run: `pytest tests/path/test.py::test_specific_behavior -v`
Expected: FAIL — "function not defined"

**Step 3: Write minimal implementation**

```python
def function(input):
    return expected
```

**Step 4: Run test to verify pass**

Run: `pytest tests/path/test.py::test_specific_behavior -v`
Expected: PASS

**Step 5: Commit**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
````

### DRY (Don't Repeat Yourself)
- **Bad:** Copy-paste validasi di 3 tempat
- **Good:** Extract fungsi validasi, pakai di mana-mana

### YAGNI (You Aren't Gonna Need It)
- **Bad:** Tambah "fleksibilitas" untuk kebutuhan masa depan
- **Good:** Implementasi hanya yang dibutuhkan sekarang

### TDD (Test-Driven Development)
Setiap task yang menghasilkan kode harus include full TDD cycle:
1. Tulis failing test
2. Jalankan untuk verifikasi fail
3. Tulis kode minimal
4. Jalankan untuk verifikasi pass

### Frequent Commits
Commit setelah setiap task:
```bash
git add [files]
git commit -m "type: description"
```

| Mistake | Bad | Good |
|---------|-----|------|
| Vague tasks | "Add authentication" | "Create User model with email and password_hash fields" |
| Incomplete code | "Add validation function" | Code lengkap yang bisa di-copy-paste |
| Missing verification | "Test it works" | "Run `pytest tests/test_auth.py -v`, expected: 3 passed" |
| Missing file paths | "Create the model file" | "Create: `src/models/user.py`" |

- [ ] Tasks berurutan dan logis
- [ ] Setiap task bite-sized (2-5 menit)
- [ ] File paths exact
- [ ] Code examples complete (copy-pasteable)
- [ ] Commands exact dengan expected output
- [ ] Tidak ada missing context
- [ ] DRY, YAGNI, TDD principles applied
- [ ] Frequent commits setiap task

```
Bite-sized tasks (2-5 min each)
Exact file paths
Complete code (copy-pasteable)
Exact commands with expected output
Verification steps
DRY, YAGNI, TDD
Frequent commits
```

**A good plan makes implementation obvious.**
