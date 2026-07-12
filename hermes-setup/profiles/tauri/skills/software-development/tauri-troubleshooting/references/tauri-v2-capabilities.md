# Tauri v2 Capabilities & Plugin Permission Mapping

## Core (no extra plugin needed)
- `core:default` — always available, covers invoke handler, window management basics

## Plugins (require crate in Cargo.toml + plugin registration in main.rs)

| Permission | Plugin Crate | Cargo.toml |
|---|---|---|
| `shell:allow-spawn`, `shell:allow-execute` | `tauri-plugin-shell` | `tauri-plugin-shell = "2"` |
| `dialog:all`, `dialog:allow-open`, `dialog:allow-save` | `tauri-plugin-dialog` | `tauri-plugin-dialog = "2"` |
| `fs:allow-read`, `fs:allow-write`, `fs:default` | `tauri-plugin-fs` | `tauri-plugin-fs = "2"` |
| `notification:default` | `tauri-plugin-notification` | `tauri-plugin-notification = "2"` |
| `clipboard-manager:allow-read`, `clipboard-manager:allow-write` | `tauri-plugin-clipboard-manager` | `tauri-plugin-clipboard-manager = "2"` |
| `process:default` | `tauri-plugin-process` | `tauri-plugin-process = "2"` |
| `os:default` | `tauri-plugin-os` | `tauri-plugin-os = "2"` |

## Plugin Registration Pattern (main.rs)
```rust
fn main() {
    tauri::Builder::default()
        .plugin(tauri_plugin_shell::init())    // if using shell:*
        .plugin(tauri_plugin_dialog::init())   // if using dialog:*
        .plugin(tauri_plugin_fs::init())       // if using fs:*
        // ... other plugins
        .invoke_handler(tauri::generate_handler![...])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

## Common Mistake
Adding permissions to `capabilities/default.json` WITHOUT:
1. Adding the plugin crate to `src-tauri/Cargo.toml`
2. Registering the plugin in `main.rs` via `.plugin(...)`

This causes: `Permission X not found, expected one of core:default...`
