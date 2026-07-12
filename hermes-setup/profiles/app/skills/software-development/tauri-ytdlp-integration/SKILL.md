---
name: tauri-ytdlp-integration
description: Best practices for integrating yt-dlp and FFmpeg into Tauri applications on Linux/Ubuntu and cross-platform targets.
author: Hermes Agent
tags: [tauri, rust, yt-dlp, ffmpeg, multimedia]
---

# Tauri yt-dlp & FFmpeg Integration Guidelines 🦀🎥

Panduan untuk mengintegrasikan command-line wrapper `yt-dlp` dan `FFmpeg` secara asinkron di backend Rust Tauri v2 dengan UI frontend.

## Kondisi Pemicu

Gunakan skill ini ketika:
- Mengimplementasikan proses download media menggunakan `yt-dlp` di Rust.
- Melakukan konversi audio/video menggunakan wrapper `FFmpeg`.
- Membangun progress listener real-time untuk status download dan transcoding.

---

## 1. Menghindari Stalled 0.0% Progress (yt-dlp Parser & Buffering)

Saat menggunakan `yt-dlp` di dalam subprocess Rust dengan stdio dialihkan ke pipe (`Stdio::piped()`), progress download seringkali tidak mengalir secara real-time (stuck di `0.0%` dan `0kbps` lalu tiba-tiba 100%). Hal ini disebabkan oleh buffering output stdout pada interpreter Python di dalam `yt-dlp`.

### Solusi Real-Time:
1. **Set Environment Variable:** Selalu set `PYTHONUNBUFFERED=1` pada `Command` subprocess Rust untuk memaksa Python men-flush stream output secara instan.
2. **Robust Progress Parsing:** `yt-dlp` sering kali mengabaikan `--progress-template` dalam mode pipe tertentu dan menggunakan format default (`[download]  1.5% of 10.0MiB at 310KiB/s ETA 00:26`). Parser kita harus bisa memproses format kustom dan format bawaan tersebut.

```rust
// Tambahkan env variable saat spawn
let mut child = Command::new("yt-dlp")
    .args(&args)
    .env("PYTHONUNBUFFERED", "1")
    .stdout(Stdio::piped())
    .stderr(Stdio::piped())
    .spawn()?;
```

```rust
fn parse_progress_line(line: &str) -> Option<DownloadProgress> {
    let trimmed = line.trim();
    
    let clean_line = if trimmed.starts_with("download:") {
        trimmed.strip_prefix("download:")?.trim()
    } else if trimmed.starts_with("[download]") {
        trimmed.strip_prefix("[download]")?.trim()
    } else {
        trimmed
    };
    
    let parts: Vec<&str> = clean_line.split_whitespace().collect();
    if parts.is_empty() {
        return None;
    }
    
    let first_part = parts[0];
    if !first_part.contains('.') && !first_part.contains('%') {
        return None;
    }
    
    let percent_str = first_part.replace('%', "").trim().to_string();
    let percent = percent_str.parse::<f64>().ok()?;
    if percent < 0.0 || percent > 100.0 {
        return None;
    }
    
    let mut speed = "0B/s".to_string();
    let mut eta = "--:--".to_string();
    
    if clean_line.contains("at") && clean_line.contains("ETA") {
        // Format bawaan yt-dlp: 1.5% of 10.0MiB at 310.49KiB/s ETA 00:26
        if let Some(at_idx) = parts.iter().position(|&x| x == "at") {
            if parts.len() > at_idx + 1 {
                speed = parts[at_idx + 1].to_string();
            }
        }
        if let Some(eta_idx) = parts.iter().position(|&x| x == "ETA") {
            if parts.len() > eta_idx + 1 {
                eta = parts[eta_idx + 1].to_string();
            }
        }
    } else {
        // Format template kustom: 10.5% 344.13KiB/s 00:24
        if parts.len() > 1 {
            speed = parts[1].to_string();
        }
        if parts.len() > 2 {
            eta = parts[2].to_string();
        }
    }
    
    Some(DownloadProgress {
        percent,
        speed,
        eta,
    })
}
```

---

## 2. Bundling Dependency dengan Tauri Sidecar (Offline-first & Standalone)

Untuk menghindari keharusan pengguna mendownload `yt-dlp` dan `ffmpeg` secara terpisah, kita dapat memanfaatkan fitur **Tauri Sidecar**. Ini membungkus file binary langsung ke dalam paket installer aplikasi.

### A. Struktur Folder & Penamaan File Binary
Binary eksternal diletakkan di dalam folder `src-tauri/binaries/` dengan format penamaan khusus berdasarkan *target triple* arsitektur mesin:
* **Linux (64-bit):**
  * `src-tauri/binaries/yt-dlp-x86_64-unknown-linux-gnu`
  * `src-tauri/binaries/ffmpeg-x86_64-unknown-linux-gnu`
* **Windows (64-bit):**
  * `src-tauri/binaries/yt-dlp-x86_64-pc-windows-msvc.exe`
  * `src-tauri/binaries/ffmpeg-x86_64-pc-windows-msvc.exe`

### B. Konfigurasi `tauri.conf.json`
Daftarkan sidecars tersebut di bawah blok `bundle`:
```json
"bundle": {
  "active": true,
  "targets": "all",
  "externalBin": [
    "binaries/yt-dlp",
    "binaries/ffmpeg"
  ]
}
```

### C. Konfigurasi `Cargo.toml` & Registrasi di `lib.rs`
1. Tambahkan `tauri-plugin-shell` ke dependensi `Cargo.toml`:
   ```toml
   [dependencies]
   tauri-plugin-shell = "2"
   ```
2. Daftarkan plugin shell pada inisialisasi Tauri di `src-tauri/src/lib.rs`:
   ```rust
   tauri::Builder::default()
       .plugin(tauri_plugin_opener::init())
       .plugin(tauri_plugin_shell::init())
   ```

### D. Pemanggilan Asinkron & Event Monitoring di Rust
Gunakan `tauri_plugin_shell::ShellExt` untuk memanggil sidecar secara asinkron. 

> ⚠️ **Kritis (Tauri v2 API):** 
> Objek `CommandChild` hasil dari `.spawn()` tidak memiliki method blocking `.wait()`. Pemantauan proses dan pendeteksian selesainya eksekusi (*exit code*) harus dilakukan sepenuhnya di dalam loop penangkap event channel `rx.recv()`.

```rust
use tauri::AppHandle;
use tauri_plugin_shell::ShellExt;

pub async fn download_media(
    app: &AppHandle,
    url: &str,
    output_path: &str,
    format_id: Option<&str>,
    merge_output_format: Option<&str>,
    progress_tx: tokio::sync::mpsc::Sender<DownloadProgress>,
) -> Result<String, AppError> {
    // 1. Load sidecar
    let sidecar = app.shell().sidecar("yt-dlp")
        .map_err(|e| AppError::External(format!("Failed to load sidecar: {e}")))?;

    // 2. Setup arguments
    let mut args = vec![
        "--newline".to_string(),
        "--progress".to_string(),
        "-o".to_string(),
        output_path.to_string(),
    ];
    if let Some(fmt) = format_id {
        args.push("-f".to_string());
        args.push(fmt.to_string());
    }
    args.push(url.to_string());

    // 3. Spawn process
    let (mut rx, _child) = sidecar
        .args(&args)
        .env("PYTHONUNBUFFERED", "1")
        .spawn()
        .map_err(|e| AppError::External(format!("Failed to spawn sidecar: {e}")))?;

    let mut exit_code = None;

    // 4. Capture events & output stream
    while let Some(event) = rx.recv().await {
        match event {
            tauri_plugin_shell::process::CommandEvent::Stdout(bytes) => {
                let line = String::from_utf8_lossy(&bytes);
                for sub_line in line.lines() {
                    if let Some(progress) = parse_progress_line(sub_line) {
                        let _ = progress_tx.send(progress).await;
                    }
                }
            }
            tauri_plugin_shell::process::CommandEvent::Stderr(bytes) => {
                let line = String::from_utf8_lossy(&bytes);
                for sub_line in line.lines() {
                    tracing::warn!("[stderr] {}", sub_line);
                }
            }
            tauri_plugin_shell::process::CommandEvent::Terminated(payload) => {
                // Simpan exit code saat proses berakhir
                exit_code = payload.code;
            }
            _ => {}
        }
    }

    // 5. Cek apakah proses sukses (exit_code == Some(0))
    if exit_code != Some(0) {
        return Err(AppError::Download(format!(
            "Process failed with exit code {:?}", exit_code
        )));
    }

    Ok(output_path.to_string())
}
```

---

## 3. Mengatasi Perubahan Ekstensi File oleh yt-dlp & Memaksa Output MP4

Jika Anda menentukan output path statis (misal `/path/file.mp4`), yt-dlp seringkali menggabungkan format audio-video ke ekstensi yang berbeda (seperti `.mkv` atau `.webm`) sesuai dengan kualitas stream aslinya di YouTube. Hal ini mengakibatkan file di disk tidak cocok dengan metadata database dan memicu crash "file not found" pada proses FFmpeg converter.

### Solusi 1: Minta Filename dari yt-dlp Terlebih Dahulu
Sebelum mengunduh, panggil yt-dlp dengan parameter `--get-filename` untuk menanyakan path absolut file yang sebenarnya akan dibuat:

```rust
pub async fn get_download_filename(
    app: &tauri::AppHandle,
    url: &str,
    output_dir: &str,
    format_id: Option<&str>,
    merge_output_format: Option<&str>,
) -> Result<String, AppError> {
    let sidecar = app.shell().sidecar("yt-dlp")
        .map_err(|e| AppError::External(format!("Failed to load sidecar: {e}")))?;

    let mut args = vec![
        "-o".to_string(),
        format!("{}/%(title)s.%(ext)s", output_dir),
        "--get-filename".to_string(),
    ];

    if let Some(fmt) = format_id {
        args.push("-f".to_string());
        args.push(fmt.to_string());
    }
    if let Some(mof) = merge_output_format {
        args.push("--merge-output-format".to_string());
        args.push(mof.to_string());
    }
    args.push(url.to_string());

    let output = sidecar.args(&args).output().await?;
    let resolved = String::from_utf8_lossy(&output.stdout).trim().to_string();
    Ok(resolved)
}
```

### Solusi 2: Memaksa Output MP4 dengan Remuxing/Merging
Guna mencegah yt-dlp menghasilkan container `.webm` saat mengunduh format video resolusi tinggi (seperti VP9/AV1), tambahkan argumen `--merge-output-format mp4` di `yt-dlp` command. Ini akan memicu FFmpeg untuk melakukan remuxing ke `.mp4` secara otomatis setelah download selesai.

---

## 3. Casing Parameter Tauri IPC (camelCase vs snake_case)

Di Tauri v2, key argument yang dikirim dari JS secara otomatis dipetakan menggunakan **camelCase** ke parameter backend Rust yang ditulis dengan **snake_case**.
Mengirim key snake_case langsung dari JavaScript (misal `{ folder_path: path }`) akan merusak pencocokan deserialisasi deserializer Tauri, sehingga memicu error runtime:
`missing required key folderPath`

### Aturan Emas:
Kirim parameter dari frontend JavaScript selalu menggunakan **camelCase**, dan terima di Rust menggunakan **snake_case**:
```javascript
// JS Frontend (Gunakan camelCase)
await invoke('open_folder', { folderPath: path });
```
```rust
// Rust Backend (Gunakan snake_case)
#[tauri::command]
pub fn open_folder(folder_path: String) -> Result<(), AppError> { ... }
```

---

## 4. Double-Table Cleanup saat Cancel Task

Ketika user menghapus atau membatalkan stuck active task, pastikan untuk menghapusnya dari tabel `tasks` DAN tabel `queue` secara bersamaan. Jika hanya dihapus dari `tasks`, scheduling thread `QueueManager` akan mengalami panic/error saat mencoba memproses ghost task ID yang masih ada di tabel antrean.

---

## 5. Mengatasi Loop Progress Bar (Progress Smoothing + Monotonic Guard)

Saat mengunduh video berkualitas tinggi, `yt-dlp` akan mengunduh stream video terlebih dahulu (`0-100%`), lalu beralih mengunduh stream audio (`0-100%`). Hal ini menyebabkan progress bar di UI melompat mundur dari 100% ke 0% dan berulang.

**Selain itu**, bahkan di DALAM satu tahap, yt-dlp bisa mengeluarkan output progress non-linear (misal: `30% → 0% → 40%`) saat melakukan retry fragment, berpindah CDN server, atau reconnect. Tanpa perlindungan tambahan, progress bar akan "berkedip mundur" berulang kali.

### Solusi Rust (Progress Smoothing + Monotonic Guard):
Terapkan **dua lapis perlindungan**:
1. **Stage Detection:** Membagi bobot kontribusi progress (video 0-80%, audio 80-100%).
2. **Monotonic Guard (KRITIS):** Progress yang dikirim ke UI **tidak pernah boleh turun** di bawah nilai tertinggi yang pernah di-emit. Ini mencegah semua jenis lompatan mundur.

### 🔄 UI-Side Progress Bouncing on Polling/Re-render (Kritis):
Jika halaman UI secara berkala memuat ulang (poll/refresh) daftar tugas aktif dari database SQLite, pastikan live progress tidak tertimpa oleh nilai lama dari database.
- **Masalah:** Saat `setInterval` mengambil data tugas aktif untuk merender ulang daftar HTML, data progress dari database (yang mungkin lebih lambat di-update/stale) akan menimpa progress live yang di-update oleh event handler `download-progress`. Ini membuat progress bar melompat bolak-balik antara progress live dan database.
- **Solusi Frontend:** Gunakan state map lokal `taskProgressMap = {}` di frontend.
  - Setiap kali event `download-progress` masuk, simpan persentase terbaru ke map: `taskProgressMap[payload.task_id] = payload.progress`.
  - Ketika merender ulang baris tugas di HTML, gunakan nilai dari map jika ada, sebelum menggunakan nilai fallback dari database: `const percent = taskProgressMap[task.id] !== undefined ? taskProgressMap[task.id] : task.progress;`.

### ⚠️ Pitfall:
- Jangan hanya track `max_progress` di level raw — track di level **smoothed/mapped** agar guard bekerja lintas stage transition.
- Tanpa monotonic guard, progress smoothing saja TIDAK cukup. User tetap akan melihat progress bar melompat mundur di dalam satu stage.

```rust
tokio::spawn(async move {
    let mut max_smoothed: f64 = 0.0;  // Track at smoothed level, not raw
    let mut raw_peak: f64 = 0.0;
    let mut second_stage = false;

    while let Some(progress) = rx.recv().await {
        let current = progress.percent;

        // Deteksi transisi ke tahap kedua (download audio)
        if !second_stage && raw_peak > 50.0 && current < 15.0 {
            second_stage = true;
        }

        if current > raw_peak {
            raw_peak = current;
        }

        // Petakan ke skala progress linier
        let mapped = if second_stage {
            80.0 + (current * 0.2) // Audio maps 80% to 100%
        } else {
            current * 0.8          // Video maps 0% to 80%
        };

        // MONOTONIC GUARD: progress hanya boleh naik, tidak pernah turun.
        // Clamp ke nilai tertinggi yang pernah di-emit.
        let smoothed = mapped.max(max_smoothed).min(100.0);
        max_smoothed = smoothed;

        emit_progress(smoothed);
    }
});
```

---

## 6. Pengelolaan Subfolder Dinamis & Kustom

User menginginkan fleksibilitas untuk memilih atau membuat subfolder di dalam folder unduhan utama (seperti `/home/yan/Downloads/YibYib_Media/lagu`) secara langsung dari panel analisis media, serta menyimpan subfolder kustom tersebut secara persisten.

### Solusi Desain:
1. **Database Persistence:** Simpan daftar path folder kustom sebagai JSON array di dalam tabel settings database SQLite (misal kunci: `"user_download_folders"`).
2. **Command Rust dengan Dynamic Output Override:** Ubah command download agar menerima opsional parameter path (`custom_output_dir`). Jika di-pass, buat folder tersebut secara rekursif (`std::fs::create_dir_all`) dan gunakan folder tersebut sebagai output path `yt-dlp`.

```rust
#[tauri::command]
pub async fn start_download(
    url: String,
    format_id: Option<String>,
    custom_output_dir: Option<String>,
) -> Result<String, AppError> {
    // Jika ada custom_output_dir, gunakan path tersebut untuk download
    let target_dir = custom_output_dir.unwrap_or_else(|| default_dir.clone());
    std::fs::create_dir_all(&target_dir)?;
    // ... jalankan download ...
}
```

3. **Scoping Rendering Helpers:** Saat mengimplementasikan fungsi pembantu HTML rendering (misal `renderFolderSelectorHTML()`), pastikan fungsi tersebut diletakkan pada **skup modul utama (global module scope)**, bukan bersarang di dalam fungsi pembantu view tertentu (seperti `renderSingleVideoPreview()`). Jika bersarang, view lain (seperti `renderPlaylistPreview()`) yang mencoba memanggil fungsi pembantu tersebut akan menghasilkan kesalahan `ReferenceError: Can't find variable`.

---

## 7. UX Toast Selectable & Kopierable untuk Error Logging

Dalam aplikasi desktop berbasis Tauri, sangat krusial bagi pengguna untuk dapat menyeleksi teks kesalahan (error) dan menyalinnya ke clipboard guna pelaporan masalah (debugging).

### Praktik Terbaik:
1. **Teks Selectable secara CSS:** Gunakan class Tailwind `select-text` (atau CSS `user-select: text`) pada elemen Toast agar kursor mouse dapat menyorot teks.
2. **Penyalinan Clipboard Satu-Klik:** Tambahkan tombol salin (📋) menggunakan `navigator.clipboard.writeText` di dalam toast.
3. **Durasi Bersyarat:** Atur agar toast dengan level `error` bertahan lebih lama (misalnya 10-12 detik alih-alih default 4 detik) dan sediakan tombol tutup manual (✕).
4. **Pencegahan Error Stringify Kosong (`{}`):**
   Di JavaScript, standard object `Error` tidak memiliki properti enumerable, sehingga `JSON.stringify(err)` akan mengembalikan `{}`. Pastikan untuk menyeleksi objek kesalahan dan memformatnya sebelum logging dikirim ke backend:
   ```javascript
   function formatArg(a) {
     if (a instanceof Error) {
       return `${a.message}\n${a.stack || ''}`;
     }
     if (typeof a === 'object' && a !== null) {
       try { return JSON.stringify(a); } catch { return String(a); }
     }
     return String(a);
   }
   ```

---

## 8. Pengelolaan Riwayat Unduhan & Manipulasi Berkas Fisik

Ketika mengimplementasikan tabel riwayat unduhan (download history) yang berinteraksi langsung dengan berkas fisik di dalam penyimpanan lokal pengguna, terapkan praktik terbaik berikut:

### A. Pencegahan Layout Overflow dengan Pemendekan Path (Truncated Paths)
Teks path absolut (`/home/user/Downloads/YibYib_Media/file.mp4`) seringkali sangat panjang dan dapat merusak tata letak (layout grid) tabel riwayat di resolusi layar standar (terutama mendesak kolom Action keluar dari layar).
- **Solusi:** Batasi lebar kolom tabel menggunakan CSS max-width dan gunakan properti Tailwind `truncate block` (atau `text-overflow: ellipsis`) pada pembungkus teks path. Contoh:
  ```html
  <td class="p-4 max-w-[200px] md:max-w-[320px]">
    <div class="font-semibold text-neutral-200 truncate" title="${item.title}">${item.title}</div>
    <div class="text-xxs text-neutral-500 truncate block mt-0.5" title="Folder: ${item.output_folder}/${item.filename}">
      ${item.output_folder}/${item.filename}
    </div>
  </td>
  ```

### B. Manipulasi Rename Berkas Fisik & Sinkronisasi SQLite
Menyediakan fitur ubah nama (Rename) pada riwayat memerlukan pembaruan ganda: perubahan nama berkas di disk penyimpanan lokal dan pembaruan kolom database (`title` & `filename`).
- **Solusi Langkah-Langkah Aman:**
  1. Ambil data path lama berkas dari database berdasarkan ID.
  2. Gunakan `std::fs::rename` untuk mengganti nama berkas fisik di dalam penyimpanan lokal.
  3. **Pitfall Guard:** Lakukan pengecekan apakah berkas fisik tersebut benar-benar ada di disk (`old_path.exists()`). Jika tidak ada (misalnya pengguna menghapusnya secara manual di luar aplikasi), lewati langkah penggantian nama berkas di disk (tulis warning log) dan lanjutkan ke pembaruan database agar status rekaman database tidak rusak/stuck.
  4. Lakukan query `UPDATE history` untuk menyinkronkan data judul dan nama berkas yang baru.
  
  ```rust
  #[tauri::command]
  pub fn rename_history_item(
      state: State<'_, AppState>,
      id: i64,
      new_title: String,
      new_filename: String,
  ) -> Result<(), AppError> {
      let conn = state.db.lock().unwrap();
      let entry = queries::get_history_entry_by_id(&conn, id)?;
      
      if let Some(item) = entry {
          let old_path = std::path::PathBuf::from(&item.output_folder).join(&item.filename);
          let new_path = std::path::PathBuf::from(&item.output_folder).join(&new_filename);
          
          if old_path.exists() {
              std::fs::rename(&old_path, &new_path)
                  .map_err(|e| AppError::External(format!("Failed to rename file: {e}")))?;
          }
          
          queries::update_history_item_filename(&conn, id, &new_title, &new_filename)?;
          Ok(())
      } else {
          Err(AppError::NotFound(format!("ID {id} not found")))
      }
  }
  ```

### C. Konsistensi Cursor Hover pada Tombol Kustom
Saat merender tombol aksi khusus (`Play`, `Folder`, `Rename`, `Delete`) di dalam web-view Tauri/GTK Linux, kursor mouse sering kali tidak berubah menjadi ikon tangan (`pointer`) secara bawaan. Selalu sertakan kelas `cursor-pointer` pada elemen-elemen tombol aksi interaktif untuk menjaga feel interaksi tetap terasa seperti aplikasi desktop native.

### D. Layout Tombol Aksi yang Compact, Badge Tipe File, & Badge Ukuran File
Untuk menghindari kolom tabel riwayat unduhan yang terlalu lebar atau terpotong pada layar yang lebih kecil, satukan tombol aksi, tipe file, dan ukuran file langsung di dalam baris informasi media utama:
1. **File Type Badge & File Size Badge:** Tampilkan ekstensi berkas secara eksplisit bersama ukuran berkas berdampingan di sebelah kiri judul media:
   - **Badge Tipe File (Warna Berbeda):**
     - **Biru** untuk berkas Video (seperti `mp4`, `webm`).
     - **Hijau** untuk berkas Audio/Lainnya (seperti `mp3`, `wav`, `m4a`).
   - **Badge Ukuran File & Badge Kualitas:** Menampilkan badge ukuran berkas (misal: `12.5 MB`) dan badge resolusi/bitrate (misal: `720p`, `320kbps`) menggunakan badge berwarna netral/gelap (seperti `bg-neutral-800`) langsung di sebelah kanan tipe berkas.
2. **Inline Action Sub-row:** Hilangkan kolom "Actions" dan "File Size" terpisah. Pindahkan seluruh tombol aksi (Play, Folder, Rename, Tags, Delete) tepat di bawah path berkas pada kolom detail media utama. Ini menyederhanakan struktur tabel dari 5 kolom menjadi hanya **3 kolom utama** (**Media Details**, **Platform**, **Date**), sangat ramah responsivitas, dan memberikan ruang optimal untuk judul berkas yang panjang.

### E. Menyimpan Metadata Kualitas Tanpa Migrasi Database SQLite (Kritis)
Ketika skema tabel `history` atau tabel SQLite lama tidak memiliki kolom khusus untuk menyimpan resolusi video (seperti `720p`) atau bitrate audio (seperti `320kbps`), dan Anda ingin menghindari migrasi skema database yang kompleks pada database pengguna yang sudah ada:
- **Pola Desain (Value-Encoding):** Enkode metadata tambahan ke dalam kolom status yang ada. Alih-alih menyimpan status `"completed"`, simpan sebagai `"completed|720p"` atau `"completed|320kbps"`.
- **Parsing di Frontend:** Pada saat memuat daftar riwayat, lakukan splitting string status:
  ```javascript
  const statusParts = (item.status || '').split('|');
  const quality = statusParts.length > 1 ? statusParts[1] : ''; // e.g. "720p" atau "320kbps"
  ```
- **Fallback untuk Legacy Records:** Jika record lama belum memiliki informasi ini (hanya berisi `"completed"`), berikan nilai fallback cerdas di frontend berdasarkan ekstensi berkas (misal: `.mp3` -> `320kbps`, `.mp4` -> `720p`).

### F. Opsi Default yang Seimbang untuk Pengunduhan
- **Video Mode:** Atur default ke **720p (HD)** pada container format **MP4** demi efisiensi bandwidth dan kecocokan putar di semua perangkat desktop/mobile.
- **Audio Mode:** Atur default ke **320 kbps (HQ)** pada container format **MP3** untuk kualitas suara terbaik.

---

## 9. Portabilitas Cross-Platform (Linux/Windows/macOS)

Untuk memastikan kode Tauri v2 dapat dicompile dan dijalankan di sistem operasi Windows dan macOS tanpa modifikasi besar, perhatikan abstraksi berikut:

### A. Pembuka Berkas/Folder OS-Specific
Alih-alih menggunakan pemanggilan langsung perintah Linux `xdg-open`, gunakan wrapper yang mendeteksi target OS saat kompilasi (`#[cfg(target_os = ...)]`) atau gunakan perintah shell yang sesuai dengan OS target:
```rust
use std::process::Command;

pub fn open_path_in_explorer(path: &str) -> Result<(), String> {
    #[cfg(target_os = "windows")]
    {
        Command::new("explorer")
            .arg(path.replace('/', "\\"))
            .spawn()
            .map_err(|e| e.to_string())?;
    }

    #[cfg(target_os = "macos")]
    {
        Command::new("open")
            .arg(path)
            .spawn()
            .map_err(|e| e.to_string())?;
    }

    #[cfg(target_os = "linux")]
    {
        Command::new("xdg-open")
            .arg(path)
            .spawn()
            .map_err(|e| e.to_string())?;
    }

    Ok(())
}
```

### B. Pemisah Path (Path Separator) dan Direktori Utama
- Hindari hardcoding karakter pemisah path `/` (slash) atau `\` (backslash). Gunakan library `std::path::PathBuf` yang secara otomatis menyesuaikan pemisah path sesuai dengan sistem operasi target.
- Gunakan API helper bawaan Tauri untuk mendapatkan direktori system default (misalnya `tauri::api::path` atau library `dirs` dari Rust crates) alih-alih hardcoding path Linux seperti `/home/user/Downloads/`.
- Saat mendownload atau mengonversi berkas, selalu konversi path ke representasi string yang kompatibel menggunakan `.to_string_lossy()` untuk memastikan stabilitas format string argument saat dikirim ke executor subprocess (`Command`).

---

## 10. Integrasi API Metadata Media (ID3 Tags, iTunes, & MusicBrainz)

Untuk memperkaya berkas unduhan audio `.mp3` dengan metadata ID3 yang akurat (Cover Art, Title, Artist, Album, Genre, Year) secara asinkron di Tauri:

### A. Multi-Source Metadata Pipeline (Waterfall Fallback)
Membangun pipeline pencarian metadata berantai (iTunes API → MusicBrainz → Local Regex Parser) agar pencarian tetap andal meskipun salah satu API tidak mengembalikan data yang lengkap (misalnya tag genre kosong).

*   **iTunes Search API:** Sangat andal untuk rilisan komersial. Gratis, tidak butuh API key, mendukung kueri pencarian fleksibel, dan menyediakan cover art resolusi tinggi (600x600px).
*   **MusicBrainz API:** Database open-source komunitas. Sangat lengkap untuk detail rilis album indie. Membutuhkan User-Agent kustom di header request agar tidak terblokir rate-limit.
*   **Spotify API (Opsional):** Dapat diaktifkan jika pengguna menyuplai Client ID & Secret Key sendiri.

### B. Implementasi Pencarian Metadata di Rust (reqwest)
Selalu gunakan parameter kueri bawaan dari builder `reqwest` untuk menghindari ketergantungan pada crate pemformatan URL pihak ketiga (seperti `urlencoding`).

```rust
// Contoh pencarian iTunes Search API di Rust
pub async fn lookup_itunes(query: &str) -> Result<Vec<MetadataMatch>, AppError> {
    let client = reqwest::Client::new();
    let response = client.get("https://itunes.apple.com/search")
        .query(&[("term", query), ("entity", "song"), ("limit", "5")])
        .send()
        .await?
        .json::<ITunesSearchResponse>()
        .await?;
    
    // Konversi hasil pencarian menjadi representasi seragam ...
    Ok(results)
}
```

### C. Implementasi Menulis Tag ID3 & Embed Cover Art (crate: `id3`)
Gunakan crate `id3` untuk memodifikasi metadata MP3 secara langsung di disk secara asinkron tanpa merusak bytes audio utama.

```rust
use id3::{Tag, TagLike, Frame, Version};
use id3::frame::{Content, Picture, PictureType};

pub async fn write_mp3_metadata(
    file_path: &str,
    title: &str,
    artist: &str,
    album: &str,
    genre: &str,
    year_str: &str,
    cover_url: Option<&str>,
) -> Result<(), AppError> {
    // 1. Baca atau buat tag ID3v2 baru
    let mut tag = Tag::read_from_path(file_path).unwrap_or_else(|_| Tag::new());
    
    tag.set_title(title);
    tag.set_artist(artist);
    tag.set_album(album);
    tag.set_genre(genre);
    
    if let Ok(year) = year_str.parse::<i32>() {
        tag.set_year(year);
    }

    // 2. Download dan Embed Gambar Album (Cover Art) jika ada URL
    if let Some(url) = cover_url {
        if url.starts_with("http") {
            if let Ok(response) = reqwest::get(url).await {
                if let Ok(bytes) = response.bytes().await {
                    // Cari MIME type gambar dari URL
                    let mime = if url.contains(".png") { "image/png" } else { "image/jpeg" };
                    
                    let picture = Picture {
                        mime_type: mime.to_string(),
                        picture_type: PictureType::CoverFront,
                        description: "Cover".to_string(),
                        data: bytes.to_vector(),
                    };
                    
                    // Hapus gambar lama dan pasang yang baru
                    tag.remove_all_pictures();
                    tag.add_picture(picture);
                }
            }
        }
    }

    // 3. Tulis perubahan kembali ke file MP3 secara fisik
    tag.write_to_path(file_path, Version::Id3v24)
        .map_err(|e| AppError::External(format!("Failed to write ID3 tags: {e}")))?;
        
    Ok(())
}
```

### D. Panduan Desain Modal Tag Editor di Frontend (JavaScript)
*   **Auto-Lookup & Auto-Fill:** Sediakan form pencarian dinamis di dalam modal. Tombol "Search Online" memicu IPC `lookup_metadata` dan menampilkan dropdown hasil kecocokan. Memilih salah satu opsi langsung mengisi otomatis (auto-fill) field Title, Artist, Album, Genre, dan memuat pratinjau thumbnail cover art secara instan.
*   **File Renaming Integration:** Setelah mendeteksi kecocokan metadata secara otomatis saat pengunduhan selesai, disarankan untuk mengganti nama file fisik di disk (via `std::fs::rename`) menjadi format yang rapi dan konsisten (misalnya `"Artis - Judul.mp3"`) agar penyimpanan lokal pengguna tetap teratur.

---


