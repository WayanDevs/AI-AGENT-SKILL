---
name: rust-programming-expert
description: Expert-level skill for Rust programming (Rust 2024 / v1.85+). Covers memory safety (ownership/lifetimes), async programming (Tokio), API backends (Axum, SQLx), CLI development (Clap, Serde), unsafe safety, optimization, and profiling.
author: Roedy Rustam
tags: [rust, async, axum, sqlx, tokio, clap, serde]
---

# Rust Programming Expert 🦀

> Expert-level guidance for high-performance, robust, memory-safe systems programming with Rust 2024 (v1.85+).

## Kondisi Pemicu

- Bootstrapping/memelihara crate, aplikasi, atau workspace Rust production
- Mendesain data model dengan lifetimes kompleks, smart pointers (`Arc`, `Rc`, `RefCell`), atau zero-copy abstractions (`Cow`)
- Membangun high-concurrency async services dengan Tokio + Axum
- Interaksi database dengan compile-time checked SQL (SQLx)
- CLI utilities modern (Clap + Serde)
- Migrasi Rust codebase ke Rust 2024 Edition
- Profiling, optimasi, debugging compile times atau memory footprint

---

## 1. Rust 2024 Edition Essentials

Rust 2024 (stabil di v1.85) — enhancement utama:

| Feature | Pattern Baru |
|---------|--------------|
| **Async Closures** | `let f = async \|x\| process(x).await;` |
| **RPIT Lifetimes** | RPIT capture semua lifetime in-scope. Gunakan `use<'a, T>` untuk restriksi |
| **Unsafe Extern** | `extern "C" { ... }` butuh `unsafe` keyword |
| **Prelude** | `Future` dan `IntoFuture` auto-import — stop `use std::future::Future` |
| **Temporary Scopes** | `if let` temporaries lebih cerdas |

---

## 2. Memory Safety & Ownership

### Ownership Mental Model
- **Setiap value punya satu owner**, value di-drop saat owner out-of-scope
- **Immutable references** (`&T`) boleh banyak — **mutable reference** (`&mut T`) cuma satu
- **Lifetimes** — reference tidak boleh outlive data yang ditunjuk

### Smart Pointers

```rust
// Multi-threaded state sharing
use std::sync::{Arc, RwLock};

struct AppState {
    inner: Arc<RwLock<SharedData>>,
}

#[derive(Debug)]
struct SharedData {
    connections: u32,
    users: Vec<String>,
}

impl AppState {
    pub fn add_user(&self, name: String) -> Result<(), &'static str> {
        let mut data = self.inner.write().map_err(|_| "poisoned")?;
        data.connections += 1;
        data.users.push(name);
        Ok(())
    }
}
```

- **`Arc<T>`** — Thread-safe shared reference
- **`Mutex<T>` / `RwLock<T>`** — Mutual exclusion
- **`Rc<T>` + `RefCell<T>`** — Single-threaded interior mutability saja

---

## 3. Error Handling

### Library: `thiserror`

```rust
use thiserror::Error;

#[derive(Error, Debug)]
pub enum DbError {
    #[error("Connection failed: {0}")]
    ConnectionFailed(String),
    #[error("{0} not found (id: {1})")]
    NotFound(&'static str, i64),
    #[error(transparent)]
    Sqlx(#[from] sqlx::Error),
}
```

### Application: `anyhow`

```rust
use anyhow::{Context, Result};

fn read_config(path: &str) -> Result<String> {
    let mut file = File::open(path)
        .with_context(|| format!("Buka {} gagal", path))?;
    let mut s = String::new();
    file.read_to_string(&mut s)?;
    Ok(s)
}
```

### Aturan
- Library → `thiserror` (error domain terstruktur)
- Application → `anyhow` (stack trace + error wrapping)
- Operator `?` untuk propagasi

---

## 4. Async Programming (Tokio)

### Best Practices

```rust
use tokio::task;

async fn process_job(job_id: u64) {
    // 1. I/O async — aman di reactor thread
    tokio::time::sleep(tokio::time::Duration::from_millis(50)).await;

    // 2. CPU heavy — offload ke blocking pool
    let result = task::spawn_blocking(move || {
        let mut sum = 0u64;
        for i in 0..10_000_000 {
            sum = sum.wrapping_add(i ^ job_id);
        }
        sum
    })
    .await
    .expect("Worker panicked");
}
```

### Aturan
- `tokio::spawn` untuk concurrent I/O tasks
- Jangan blocking async runtime dengan CPU-bound work → pakai `spawn_blocking`
- Async closures (Rust 2024): `let f = async || { ... };`

---

## 5. Production Web API: Axum + SQLx

```rust
use axum::{
    extract::{Path, State},
    http::StatusCode,
    routing::{get, post},
    Json, Router,
};
use serde::{Deserialize, Serialize};
use sqlx::postgres::{PgPool, PgPoolOptions};
use std::sync::Arc;

#[derive(Serialize, Deserialize)]
pub struct User {
    id: i32,
    username: String,
    email: String,
}

pub struct ApiState {
    db: PgPool,
}

#[tokio::main]
async fn main() -> Result<(), anyhow::Error> {
    let pool = PgPoolOptions::new()
        .max_connections(20)
        .connect(&dotenvy::var("DATABASE_URL")?)
        .await?;

    let app = Router::new()
        .route("/users/:id", get(get_user))
        .route("/users", post(create_user))
        .with_state(Arc::new(ApiState { db: pool }));

    let listener = tokio::net::TcpListener::bind("0.0.0.0:8080").await?;
    axum::serve(listener, app).await?;
    Ok(())
}

async fn get_user(
    State(state): State<Arc<ApiState>>,
    Path(id): Path<i32>,
) -> Result<Json<User>, (StatusCode, String)> {
    let user = sqlx::query_as!(
        User,
        "SELECT id, username, email FROM users WHERE id = $1",
        id
    )
    .fetch_optional(&state.db)
    .await
    .map_err(|e| (StatusCode::INTERNAL_SERVER_ERROR, e.to_string()))?
    .ok_or_else(|| (StatusCode::NOT_FOUND, "User not found".to_string()))?;

    Ok(Json(user))
}
```

### Compile-time SQL Checking
- `sqlx::query_as!()` — diverifikasi compiler saat `cargo build`
- Butuh `DATABASE_URL` env var untuk compile
- Atau `sqlx::query_as::<_, User>()` (runtime check)

---

## 6. CLI Development: Clap + Serde

```rust
use clap::{Parser, Subcommand};

#[derive(Parser, Debug)]
#[command(name = "mycli", version = "1.0", about = "CLI Tool")]
pub struct Cli {
    #[arg(short, long, global = true)]
    pub config: Option<String>,

    #[arg(short, long)]
    pub verbose: bool,

    #[command(subcommand)]
    pub command: Commands,
}

#[derive(Subcommand, Debug)]
pub enum Commands {
    #[command(about = "Analyze code")]
    Analyze {
        #[arg(short, long)]
        path: String,
        #[arg(long, default_value = "json")]
        format: String,
    },
    Init,
}
```

---

## 7. Optimization & Unsafe

### Cargo Release Profile

```toml
[profile.release]
opt-level = 3
lto = true
codegen-units = 1
panic = "abort"
strip = true
```

### Zero-cost Practices
- `&str` over `String` untuk read-only
- `Cow<'a, str>` untuk occasional mutation
- `&[T]` over `&Vec<T>` sebagai function args

### Unsafe Rules
- Hanya untuk C bindings, lockless data structures, atau raw pointers
- Wajib `// SAFETY:` block dokumentasi invariant

```rust
// SAFETY: ptr valid, aligned, initialized u32
unsafe {
    let val = std::ptr::read_volatile(ptr);
}
```

---

## 8. Testing & Quality

```rust
pub fn add(a: i32, b: i32) -> i32 { a + b }

#[cfg(test)]
mod tests {
    use super::*;
    #[test]
    fn test_add() {
        assert_eq!(add(2, 2), 4);
    }
}
```

```bash
cargo fmt --all -- --check
cargo clippy --all-targets --all-features -- -D warnings
cargo test --all-targets
```

---

## ⚠️ Pitfalls

| Masalah | Solusi |
|---------|--------|
| "borrowed value does not live long enough" | Return owned data (`String`, `Vec<T>`) atau adjust lifetime bounds |
| `Send` error di async | Variable across `.await` harus `Send`. Jangan hold `MutexGuard` std melewati `.await` — gunakan `tokio::sync::Mutex` |
| Cargo lock contention | `rm -f target/.rustc_info.json` atau kill zombie process |
| Compile lama | Set `CARGO_BUILD_JOBS=4`, gunakan `sccache` |
