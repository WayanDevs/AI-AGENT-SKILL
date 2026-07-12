---
name: rust-best-practices
description: "Panduan komprehensif menulis Rust idiomatic berdasarkan Apollo GraphQL's Best Practices Handbook. 9 chapter: coding styles, clippy, performance, error handling, testing, generics, type state, documentation, pointers. Index skill ke semua chapter individual."
version: 1.0.0
category: general
status: published
confidence: 0.9
source: learned
owner: wayan
created: "2026-07-04T11:57:32Z"
---

# Rust Best Practices

Panduan menulis Rust idiomatic berdasarkan Apollo GraphQL's [Rust Best Practices Handbook](https://github.com/apollographql/rust-best-practices).

Gunakan skill ini saat:
1. Menulis Rust code baru
2. Review atau refactoring Rust code yang ada
3. Memilih antara borrowing vs cloning vs ownership
4. Implementasi error handling dengan Result types
5. Optimasi performa Rust code
6. Menulis tests atau dokumentasi untuk Rust projects

Setiap chapter tersedia sebagai **skill individual** yang bisa di-load terpisah:

| Chapter | Skill Name | Topik |
|---------|-----------|-------|
| 1 | `rust-coding-styles-idioms` | Borrowing vs cloning, Copy trait, Option/Result handling, iterators, comments |
| 2 | `rust-clippy-linting` | Clippy configuration, important lints, workspace lint setup |
| 3 | `rust-performance-mindset` | Profiling, avoiding redundant clones, stack vs heap, zero-cost abstractions |
| 4 | `rust-error-handling` | Result vs panic, thiserror vs anyhow, error hierarchies |
| 5 | `rust-automated-testing` | Test naming, one assertion per test, snapshot testing, doc tests |
| 6 | `rust-generics-dispatch` | Static vs dynamic dispatch, trait objects, generics |
| 7 | `rust-type-state-pattern` | Compile-time state safety, PhantomData, when to use it |
| 8 | `rust-comments-documentation` | When to comment, doc comments, rustdoc, TODO conventions |
| 9 | `rust-understanding-pointers` | Thread safety, Send/Sync, pointer types, Arc/Rc/Box |

### Borrowing & Ownership
- Prefer `&T` over `.clone()` unless ownership transfer is required
- Use `&str` over `String`, `&[T]` over `Vec<T>` in function parameters
- Small `Copy` types (≤24 bytes) can be passed by value
- Use `Cow<'_, T>` when ownership is ambiguous

### Error Handling
- Return `Result<T, E>` for fallible operations; avoid `panic!` in production
- Never use `unwrap()`/`expect()` outside tests
- Use `thiserror` for library errors, `anyhow` for binaries only
- Prefer `?` operator over match chains for error propagation

### Performance
- Always benchmark with `--release` flag
- Run `cargo clippy -- -D clippy::perf` for performance hints
- Avoid cloning in loops; use `.iter()` instead of `.into_iter()` for Copy types
- Prefer iterators over manual loops; avoid intermediate `.collect()` calls

### Linting
Run regularly: `cargo clippy --all-targets --all-features --locked -- -D warnings`

Key lints to watch:
- `redundant_clone` — unnecessary cloning
- `large_enum_variant` — oversized variants (consider boxing)
- `needless_collect` — premature collection

Use `#[expect(clippy::lint)]` over `#[allow(...)]` with justification comment.

### Testing
- Name tests descriptively: `process_should_return_error_when_input_empty()`
- One assertion per test when possible
- Use doc tests (`///`) for public API examples
- Consider `cargo insta` for snapshot testing generated output

### Generics & Dispatch
- Prefer generics (static dispatch) for performance-critical code
- Use `dyn Trait` only when heterogeneous collections are needed
- Box at API boundaries, not internally

### Type State Pattern
Encode valid states in the type system to catch invalid operations at compile time:
```rust
struct Connection<State> { /* ... */ _state: PhantomData<State> }
struct Disconnected;
struct Connected;

impl Connection<Connected> {
    fn send(&self, data: &[u8]) { /* only connected can send */ }
}
```

### Documentation
- `//` comments explain *why* (safety, workarounds, design rationale)
- `///` doc comments explain *what* and *how* for public APIs
- Every `TODO` needs a linked issue: `// TODO(#42): ...`
- Enable `#![deny(missing_docs)]` for libraries

---

> **Source:** Apollo GraphQL's [Rust Best Practices Handbook](https://github.com/apollographql/rust-best-practices)
> **Related skills:** `rust-programming-expert`, `rust-coding-styles-idioms`, `rust-clippy-linting`, `rust-performance-mindset`, `rust-error-handling`, `rust-automated-testing`, `rust-generics-dispatch`, `rust-type-state-pattern`, `rust-comments-documentation`, `rust-understanding-pointers`
