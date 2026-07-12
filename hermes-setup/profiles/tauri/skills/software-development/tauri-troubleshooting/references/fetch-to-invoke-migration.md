# fetch() → invoke() Migration Patterns

Concrete examples from migrating a video downloader app (Express → Tauri v2).

## Rust Backend: Command Signatures

```rust
// Link analysis command — returns structured data
#[tauri::command]
fn cek_link(
    app: AppHandle,          // for emitting events
    state: State<'_, AppState>,  // shared app state
    url: String,             // frontend passes this
) -> Result<LinkCheckResult, String> { ... }

// Download command — spawns async background work
#[tauri::command]
fn add_download(
    app: AppHandle,
    state: State<'_, AppState>,
    url: String,
    title: String,           // add_download now takes title from cek_link result
    file_type: String,
    resolution: String,
) -> Result<DownloadItem, String> { ... }
```

## Frontend: Matching invoke() Calls

```ts
// Analyze link — note: camelCase JS params auto-map to snake_case Rust params
const result = await invoke('cek_link', { url: urlValue });

// Start download — pass all params the Rust fn expects
await invoke('add_download', {
    url,
    title,        // ← enriched from cek_link result
    fileType: 'mp4',
    resolution: '1080p'
});

// Control download
await invoke('control_download', { id: downloadId, action: 'pause' });

// Get data (no params)
const logs: SystemLog[] = await invoke('get_system_logs');
```

## Event-Driven Log Updates

Pattern: backend emits `log-updated` event after each push_log(), frontend listens and refreshes.

```rust
// Rust: emit after logging
push_log(&state.logs, "INFO", "[cek-link] Platform terdeteksi: YouTube");
let _ = app.emit("log-updated", ());
```

```ts
// TS: listen + refresh
listen('log-updated', () => { pollLogsOnce(); });

async function pollLogsOnce() {
    const logs = await invoke('get_system_logs');
    renderAllLogs(logs);
}
```

## Two-Phase UX: Cek Link → Download

Better UX than direct download: user clicks "Cek Link" first to preview metadata, then confirms download.

1. `invoke('cek_link', { url })` → returns title, thumbnail, duration, playlist tracks
2. Frontend renders preview (single video card or playlist track list)
3. User clicks Download → `invoke('add_download', { url, title, ... })` with enriched title

This avoids the "Proses Analisis Tautan..." placeholder title problem.

## Playlist Detection

```rust
let is_playlist = url.contains("list=") || url.contains("/playlist");
```

When playlist detected, return `link_type: "playlist"` with items array.
Frontend renders checkbox list of tracks, user selects which to download.
Each selected track gets its own `add_download()` call.
