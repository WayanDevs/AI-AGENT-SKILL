---
name: code-review
description: "Pre-commit code review pipeline: security scan, quality gates, independent reviewer, auto-fix loop. Verifikasi otomatis sebelum commit/push \u2014 static scan, baseline test comparison, checklist keamanan, dan auto-fix maksimal 2 siklus."
version: 1.0.0
category: general
status: published
confidence: 0.9
source: learned
owner: wayan
created: "2026-07-04T11:57:32Z"
---

## When to Use

- Setelah implementasi fitur atau bug fix, sebelum `git commit` atau `git push`
- Ketika user bilang "commit", "push", "ship", "done", "verify", "review"
- Setelah menyelesaikan task dengan 2+ file edits di git repo
- Sebagai quality gate setelah setiap task

**Skip untuk:** perubahan dokumentasi saja, config tweak, atau user bilang "skip verification".

## Pitfalls

- **Empty diff** — cek `git status`, beritahu user tidak ada yang perlu diverifikasi
- **Bukan git repo** — skip dan beritahu user
- **Diff besar (>15k chars)** — split per file, review masing-masing
- **Reviewer return non-JSON** — retry sekali dengan prompt lebih ketat, lalu treat sebagai FAIL
- **False positives** — jika reviewer flag sesuatu yang intentional, catat di fix prompt
- **Test framework tidak ditemukan** — skip regression check, reviewer verdict tetap jalan
- **Lint tools tidak terinstal** — skip check itu diam-diam, jangan fail
- **Auto-fix memperkenalkan issue baru** — dihitung sebagai failure baru, cycle berlanjut

# Pre-Commit Code Review

Pipeline verifikasi otomatis sebelum kode di-commit. Static scan, quality gates baseline-aware, independent reviewer, dan auto-fix loop.

**Prinsip utama:** Jangan verifikasi pekerjaanmu sendiri. Context baru menemukan apa yang kamu lewatkan.

```bash
git diff --cached
```

Jika kosong, coba `git diff` lalu `git diff HEAD~1 HEAD`.

Jika `git diff --cached` kosong tapi `git diff` ada perubahan, minta user `git add <files>` dulu.

Jika diff melebihi 15.000 karakter, split per file:
```bash
git diff --name-only
git diff HEAD -- specific_file.py
```

Scan hanya baris yang ditambahkan. Setiap match = security concern.

```bash
# Hardcoded secrets
git diff --cached | grep "^+" | grep -iE "(api_key|secret|password|token|passwd)\s*=\s*['\"][^'\"]{6,}['\"]"

# Shell injection
git diff --cached | grep "^+" | grep -E "os\.system\(|subprocess.*shell=True"

# Dangerous eval/exec
git diff --cached | grep "^+" | grep -E "\beval\(|\bexec\("

# Unsafe deserialization
git diff --cached | grep "^+" | grep -E "pickle\.loads?\("

# SQL injection (string formatting in queries)
git diff --cached | grep "^+" | grep -E "execute\(f\"|\.format\(.*SELECT|\.format\(.*INSERT"
```

Deteksi bahasa project lalu jalankan tools yang sesuai. Capture failure count **SEBELUM** perubahan sebagai **baseline_failures** (stash changes, run, pop). Hanya failure BARU yang blocking.

**Test frameworks** (auto-detect):
```bash
# Python (pytest)
python -m pytest --tb=no -q 2>&1 | tail -5

# Node (npm test)
npm test -- --passWithNoTests 2>&1 | tail -5

# Rust
cargo test 2>&1 | tail -5

# Go
go test ./... 2>&1 | tail -5
```

**Linting dan type checking** (jalankan hanya jika terinstal):
```bash
# Python
which ruff && ruff check . 2>&1 | tail -10
which mypy && mypy . --ignore-missing-imports 2>&1 | tail -10

# Node
which npx && npx eslint . 2>&1 | tail -10
which npx && npx tsc --noEmit 2>&1 | tail -10

# Rust
cargo clippy -- -D warnings 2>&1 | tail -10

# Go
which go && go vet ./... 2>&1 | tail -10
```

**Baseline comparison:** Jika baseline bersih dan perubahanmu memperkenalkan failure → regresi. Jika baseline sudah ada failure, hitung hanya yang BARU.

Quick scan sebelum dispatch reviewer:

- [ ] Tidak ada hardcoded secrets, API keys, atau credentials
- [ ] Input validation pada user-provided data
- [ ] SQL queries pakai parameterized statements
- [ ] File operations validate paths (no traversal)
- [ ] External calls ada error handling (try/catch)
- [ ] Tidak ada debug print/console.log tertinggal
- [ ] Tidak ada commented-out code
- [ ] Kode baru ada tests-nya (jika test suite ada)

Reviewer mendapat **HANYA** diff dan hasil static scan. Tidak ada shared context dengan implementer. Fail-closed: response tidak bisa di-parse = fail.

Reviewer harus evaluasi:

**SECURITY (auto-FAIL):** hardcoded secrets, backdoors, data exfiltration, shell injection, SQL injection, path traversal, eval()/exec() dengan user input, pickle.loads(), obfuscated commands.

**LOGIC ERRORS (auto-FAIL):** wrong conditional logic, missing error handling untuk I/O/network/DB, off-by-one errors, race conditions, kode bertentangan dengan intent.

**SUGGESTIONS (non-blocking):** missing tests, style, performance, naming.

Output reviewer harus berupa:
```json
{
  "passed": true/false,
  "security_concerns": [],
  "logic_errors": [],
  "suggestions": [],
  "summary": "one sentence verdict"
}
```

Gabungkan hasil dari Steps 2, 3, dan 5.

**Semua passed:** Lanjut ke Step 8 (commit).

**Ada failures:** Laporkan apa yang gagal, lalu lanjut ke Step 7 (auto-fix).

```
VERIFICATION FAILED

Security issues: [list dari static scan + reviewer]
Logic errors: [list dari reviewer]
Regressions: [new test failures vs baseline]
New lint errors: [details]
Suggestions (non-blocking): [list]
```

**Maksimal 2 siklus fix-and-reverify.**

Gunakan konteks TERPISAH (bukan implementer, bukan reviewer) untuk fix. Fix agent hanya memperbaiki issues yang dilaporkan — tidak boleh refactor, rename, atau tambah fitur.

Setelah fix selesai, jalankan ulang Steps 1-6 (full verification cycle):
- Passed → lanjut ke Step 8
- Failed dan attempts < 2 → ulangi Step 7
- Failed setelah 2 attempts → eskalasi ke user dengan remaining issues

Jika verifikasi passed:

```bash
git add -A && git commit -m "[verified] <description>"
```

Prefix `[verified]` menandakan independent reviewer sudah approve.

### Python
```python
# Bad: SQL injection
cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")
# Good: parameterized
cursor.execute("SELECT * FROM users WHERE id = ?", (user_id,))

# Bad: shell injection
os.system(f"ls {user_input}")
# Good: safe subprocess
subprocess.run(["ls", user_input], check=True)
```

### JavaScript
```javascript
// Bad: XSS
element.innerHTML = userInput;
// Good: safe
element.textContent = userInput;
```

### Rust
```rust
// Bad: unwrap in production
let value = result.unwrap();
// Good: proper error handling
let value = result.map_err(|e| AppError::from(e))?;
```

- **`plan`** — validasi implementasi sesuai plan requirements
- **`rust-best-practices`** — referensi standar saat review Rust code
- **`rust-clippy-linting`** — linting detail khusus Rust
