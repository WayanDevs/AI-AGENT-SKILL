# Tauri v2 + Vanilla JS + Vite: Post-Scaffold Checklist

Verified working pattern from real project build (July 2026, Tauri 2.x, Vite 6.x, TailwindCSS 4.x).

## After `npm create tauri-app@latest`

### 1. Install missing deps
```bash
npm install @tauri-apps/api
npm install -D vite tailwindcss @tailwindcss/postcss postcss autoprefixer
```

### 2. Move index.html to root
```bash
mv src/index.html index.html
```
Update paths inside: `/main.js` → `/src/main.js`, CSS → `/src/styles/main.css`

### 3. package.json scripts
```json
"scripts": {
  "dev": "vite",
  "build": "vite build",
  "preview": "vite preview",
  "tauri": "tauri"
}
```

### 4. vite.config.js
```js
import { defineConfig } from 'vite';
export default defineConfig({
  clearScreen: false,
  server: { port: 5173, strictPort: true, host: true },
  build: { target: ['es2021', 'chrome100', 'safari13'], minify: !process.env.TAURI_DEBUG, sourcemap: !!process.env.TAURI_DEBUG },
  envPrefix: ['VITE_', 'TAURI_'],
});
```

### 5. tauri.conf.json build section
```json
"build": {
  "beforeDevCommand": "npm run dev",
  "beforeBuildCommand": "npm run build",
  "frontendDist": "../dist",
  "devUrl": "http://localhost:5173"
}
```

### 6. postcss.config.js (TailwindCSS v4 — NO tailwind.config.js!)
```js
export default {
  plugins: { '@tailwindcss/postcss': {}, autoprefixer: {} }
};
```

### 7. src/styles/main.css
```css
@import "tailwindcss";
```

### 8. Rust lib.rs — mandatory imports
```rust
use tauri::Manager;  // CRITICAL — needed for .path(), .manage(), .emit()
```

### 9. AppError pattern (serializable for Tauri commands)
```rust
use serde::{Serialize, Serializer};
use thiserror::Error;

#[derive(Error, Debug)]
pub enum AppError {
    #[error("Database error: {0}")]
    Database(#[from] rusqlite::Error),
    #[error("I/O error: {0}")]
    Io(#[from] std::io::Error),
    #[error("{0}")]
    Custom(String),
}

impl Serialize for AppError {
    fn serialize<S>(&self, serializer: S) -> Result<S::Ok, S::Error>
    where S: Serializer {
        serializer.serialize_str(&self.to_string())
    }
}
```

### 10. Recommended Cargo.toml deps for full-stack Tauri app
```toml
[dependencies]
tauri = { version = "2", features = [] }
tauri-plugin-opener = "2"
serde = { version = "1", features = ["derive"] }
serde_json = "1"
rusqlite = { version = "0.31", features = ["bundled"] }
tokio = { version = "1", features = ["full"] }
uuid = { version = "1", features = ["v4"] }
chrono = { version = "0.4", features = ["serde"] }
thiserror = "1"
tracing = "0.1"
tracing-subscriber = { version = "0.3", features = ["env-filter", "fmt"] }
```

## Known Pitfalls
- `rusqlite` with `features = ["bundled"]` compiles SQLite from C source — first build takes ~2 min
- Old `.db` file + new schema = migration errors. Delete stale DB during dev.
- `cargo check` after adding many crates: first run downloads + compiles ~500 crates, ~5 min
- `cargo fmt` after writing Rust files cleans all rustfmt warnings in one shot
