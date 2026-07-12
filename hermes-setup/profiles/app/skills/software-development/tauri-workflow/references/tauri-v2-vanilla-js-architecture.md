# Tauri v2 + Vanilla JS + TailwindCSS v4 Architecture Reference

Proven architecture from YibYib Video Downloader MVP (July 2025). All patterns verified with `cargo check` + `npm run build` passing cleanly.

## Backend Module Layout (src-tauri/src/)

```
src-tauri/src/
├── lib.rs              # Entry: Builder, setup(), generate_handler![]
├── main.rs             # #[cfg_attr] entry point (unchanged from template)
├── constants.rs        # App-wide constants (paths, defaults, limits)
├── commands/
│   ├── mod.rs           # Barrel re-exports (pub mod + pub use)
│   ├── download_commands.rs
│   ├── queue_commands.rs
│   ├── converter_commands.rs
│   ├── history_commands.rs
│   ├── settings_commands.rs
│   ├── log_commands.rs
│   └── health_commands.rs
├── db/
│   ├── mod.rs
│   ├── schema.rs        # Migration runner (include_str! SQL files)
│   ├── queries.rs       # All CRUD functions
│   └── migrations/
│       └── v001_initial.sql
├── engine/
│   ├── mod.rs
│   ├── ytdlp.rs         # External binary wrapper (tokio::process::Command)
│   ├── url_parser.rs    # URL validation & platform detection
│   ├── task_engine.rs   # Task lifecycle (create → download → complete)
│   └── queue_manager.rs # Background daemon (tauri::async_runtime::spawn)
├── converter/
│   ├── mod.rs
│   └── ffmpeg.rs        # FFmpeg async wrapper
├── errors/
│   ├── mod.rs
│   └── app_error.rs     # thiserror + custom Serialize
├── events/
│   ├── mod.rs
│   ├── download_events.rs
│   ├── queue_events.rs
│   └── converter_events.rs
├── logging/
│   ├── mod.rs
│   └── setup.rs         # tracing-subscriber (file + stdout)
├── models/
│   ├── mod.rs
│   ├── task.rs
│   ├── queue_item.rs
│   ├── history_entry.rs
│   ├── media_info.rs
│   └── settings.rs
├── state/
│   ├── mod.rs
│   └── app_state.rs     # Mutex<Connection> + AtomicUsize + Mutex<PathBuf>
└── plugins/
    └── mod.rs           # Placeholder for future plugin system
```

## Frontend Module Layout (src/)

```
src/
├── main.js              # Bootstrap orchestrator (12 steps)
├── config/index.js      # Runtime config loader (reads from backend settings)
├── constants/           # App, routes, events, states, defaults
├── components/
│   ├── sidebar.js       # Navigation + live health status footer
│   ├── toast.js         # Toast/snackbar notification system
│   └── splash-screen.js # Loading splash with fade-out
├── hooks/index.js       # Lifecycle hooks (beforeMount, afterMount)
├── layouts/main-layout.js # 2-column flex (sidebar + content)
├── locales/en.json      # i18n strings
├── pages/               # 8 SPA pages (render + mount + unmount)
│   ├── download.js      # URL input → analyze → download with progress
│   ├── queue.js         # Queue list with status badges
│   ├── converter.js     # File drop zone → FFmpeg conversion
│   ├── history.js       # Download history table
│   ├── library.js       # Media grid cards with play/open
│   ├── log-monitor.js   # Terminal-style log viewer (3s poll)
│   ├── plugins.js       # "Coming Soon" placeholder
│   └── settings.js      # 8-tab settings (General→Advanced)
├── router/index.js      # Hash-based SPA router
├── services/            # invoke() wrappers (1 service per command group)
├── styles/main.css      # @import "tailwindcss" + custom scrollbar + keyframes
└── utils/
    ├── error-handler.js # Global window.onerror + unhandledrejection
    ├── i18n.js          # Dot-notation translator with fallback
    ├── logger.js        # Console + bridge to backend log_from_frontend
    └── shortcuts.js     # Global keyboard shortcuts (Ctrl+, Ctrl+L, etc.)
```

## Key Patterns

### generate_handler! Registration
Always use FULL module path, not re-exported names:
```rust
.invoke_handler(tauri::generate_handler![
    commands::download_commands::analyze_url,  // ✅ Full path
    commands::queue_commands::get_queue,        // ✅ Full path
    // commands::analyze_url,                  // ❌ Re-export breaks macro
])
```

### Background Daemon in setup()
```rust
.setup(|app| {
    let handle = app.handle().clone();
    tauri::async_runtime::spawn(async move {  // NOT tokio::spawn!
        loop {
            tokio::time::sleep(Duration::from_secs(3)).await;
            // ... poll queue, process tasks
        }
    });
    Ok(())
})
```

### SPA Page Contract (Vanilla JS)
Each page exports 3 functions:
```js
export function renderMyPage() { return `<div>...</div>`; }    // Returns HTML string
export async function mountMyPage() { /* bind events */ }       // Called after DOM insert
export function unmountMyPage() { /* cleanup listeners */ }     // Called before navigation away
```

### Settings with Live Reload
When user changes `concurrent_downloads` in Settings:
1. Frontend calls `setSetting('concurrent_downloads', '3')`
2. Backend `set_setting` command writes to SQLite
3. Backend also updates `AppState.concurrent_downloads.store(3, Ordering::SeqCst)`
4. Queue manager daemon reads the new value on next tick

### External Binary Health Check
```rust
fn check_binary(bin: &str, args: &[&str]) -> (bool, String) {
    match Command::new(bin).args(args).output() {
        Ok(out) => (true, String::from_utf8_lossy(&out.stdout)
            .lines().next().unwrap_or("unknown").trim().to_string()),
        Err(_) => (false, "Not found".to_string()),
    }
}
```
