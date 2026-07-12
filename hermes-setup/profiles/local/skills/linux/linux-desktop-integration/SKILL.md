---
name: linux-desktop-integration
description: Troubleshoot Linux desktop/GNOME application menu integration, GNOME extension management, dock/taskbar customization, and desktop environment configurations
category: linux
---

# Linux Desktop Integration

**Trigger:** A freshly installed app (Flatpak, native, AppImage) doesn't appear in the desktop application menu; GNOME extension setup or troubleshooting; dock/taskbar customization.

> 📎 **Supporting references:** `references/gnome-dock.md` — GNOME Dock/Ubuntu Dock customization via gsettings (center icons, full-width taskbar, background color).

## ⚠️ Safety Rules

**JANGAN pernah menjalankan perintah logout, reboot, shutdown, systemctl reboot, restart, turnoff, atau sejenisnya.** Jika perubahan memerlukan restart GNOME Shell (`Alt+F2` → `r` → Enter), logout, atau reboot — beritahu user untuk melakukannya sendiri secara manual. Jangan eksekusi sendiri.

| Perubahan | Perlu restart? | Cara |
|-----------|---------------|------|
| gsettings dock/panel | ❌ Biasanya langsung efek | — |
| Install/aktifkan extension baru (manual) | ✅ Ya, butuh restart shell | User tekan `Alt+F2`, ketik `r`, Enter |
| Nonaktifkan/ubah setting extension via gnome-extensions | ✅ Kadang perlu restart | User lakukan sendiri |

## General Approach

When an app can be launched from terminal but doesn't show in the desktop menu or has wrong icons, it's usually a **desktop file (.desktop)** or **icon resolution** issue. This pattern applies to Flatpak, Snap, AppImage, and manually installed apps.

---

## Flatpak Desktop Integration

Flatpak exports .desktop files and icons to `/var/lib/flatpak/exports/share/`, which is included in `XDG_DATA_DIRS`. When they don't appear correctly:

### Desktop Files Not Showing in Menu

1. Check if the .desktop file exists in the flatpak export:
   ```
   ls /var/lib/flatpak/exports/share/applications/ | grep <app-name>
   ```

2. Copy it to the user-local applications directory:
   ```
   cp /var/lib/flatpak/exports/share/applications/<app-id>.desktop ~/.local/share/applications/
   ```

3. **Wine-specific:** The Wine .desktop file has `NoDisplay=true` by default (it's meant to be a file handler, not a launcher). Change it:
   ```
   sed -i 's/NoDisplay=true/NoDisplay=false/' ~/.local/share/applications/org.winehq.Wine.desktop
   ```

4. Update the desktop database:
   ```
   update-desktop-database ~/.local/share/applications/
   ```

### Icons Showing as Generic/Settings Icon

1. Verify icons exist in the flatpak export:
   ```
   ls /var/lib/flatpak/exports/share/icons/hicolor/*/apps/<app-id>.*
   ```

2. Copy icons to user-local hicolor:
   ```bash
   for size in 16x16 24x24 32x32 48x48 64x64 128x128 256x256 scalable; do
     mkdir -p ~/.local/share/icons/hicolor/$size/apps/
     src="/var/lib/flatpak/exports/share/icons/hicolor/$size/apps/<app-id>.*"
     [ -f $src ] && cp -L $src ~/.local/share/icons/hicolor/$size/apps/
   done
   ```

3. Create an `index.theme` in `~/.local/share/icons/hicolor/` if missing:
   ```ini
   [Icon Theme]
   Name=Hicolor
   Comment=Fallback icon theme
   Directories=16x16/apps,24x24/apps,32x32/apps,48x48/apps,64x64/apps,128x128/apps,256x256/apps,scalable/apps
   
   [16x16/apps]
   Size=16
   Context=Applications
   Type=Fixed
   
   [24x24/apps]
   Size=24
   Context=Applications
   Type=Fixed
   
   [32x32/apps]
   Size=32
   Context=Applications
   Type=Fixed
   
   [48x48/apps]
   Size=48
   Context=Applications
   Type=Fixed
   
   [64x64/apps]
   Size=64
   Context=Applications
   Type=Fixed
   
   [128x128/apps]
   Size=128
   Context=Applications
   Type=Fixed
   
   [256x256/apps]
   Size=256
   Context=Applications
   Type=Fixed
   
   [scalable/apps]
   Size=48
   Context=Applications
   Type=Scalable
   MinSize=16
   MaxSize=512
   ```

4. Rebuild the icon cache:
   ```
   gtk-update-icon-cache ~/.local/share/icons/hicolor/ -f
   ```

5. **If still generic:** Use absolute paths in the `Icon=` field of the local .desktop file:
   ```diff
   -Icon=org.example.App
   +Icon=/home/user/.local/share/icons/hicolor/128x128/apps/org.example.App.png
   ```

6. Refresh the desktop: restart GNOME Shell (`Alt+F2`, type `r`, Enter) or logout/login.

## GNOME Extension Compatibility Patching

Setelah upgrade GNOME, banyak extension menjadi `OUT OF DATE` atau tidak muncul. Penyebab: array `shell-version` di `metadata.json` tidak mencakup versi GNOME baru.

> 📎 **Supporting references:** `references/gnome-extension-authoring.md` — panduan membuat extension sendiri + contoh lengkap Hermes Net Speed.

### Diagnosis

```bash
# Cek versi GNOME
gnome-shell --version

# Cek semua extension + status
gnome-extensions list

# Cek detail satu extension (lihat State: OUT OF DATE)
gnome-extensions info <uuid>
```

### Patch shell-version (metode cepat)

Edit `metadata.json` di folder extension, tambahkan versi GNOME yang baru:

```bash
# Lokasi extension user
~/.local/share/gnome-shell/extensions/<uuid>/metadata.json

# Atau system
/usr/share/gnome-shell/extensions/<uuid>/metadata.json
```

**Contoh perubahan:**
```json
"shell-version": [
    "47",
    "48",
    "49",
    "50"
]
```

**Setelah patch:** user perlu restart GNOME Shell (`Alt+F2` → `r` → Enter) atau logout/login.

> ⚠️ **Catatan:** Patching versi tidak menjamin extension berfungsi 100% — jika API GNOME berubah drastis, extension mungkin error. Tapi untuk banyak extension sederhana, ini sudah cukup.

### Triage: Cek Extension Existing Sebelum Buat Sendiri

Sebelum membuat extension dari nol, cek apakah sudah ada extension serupa yang tinggal di-patch:

```bash
# 1. List semua extension (system + user)
gnome-extensions list

# 2. Cari extension yang relevan
gnome-extensions list | grep -i net

# 3. Cek detail — perhatikan State:
#    - ACTIVE = berfungsi
#    - OUT OF DATE = tidak kompatibel dengan GNOME versi ini
gnome-extensions info <uuid>

# 4. Cek shell-version di metadata.json
cat ~/.local/share/gnome-shell/extensions/<uuid>/metadata.json
# atau
cat /usr/share/gnome-shell/extensions/<uuid>/metadata.json
```

**Skenario yang sering terjadi:** Extension sudah terinstall dari GNOME lama, tapi setelah upgrade GNOME jadi `OUT OF DATE`. Solusi: **patch shell-version** (lihat bagian sebelumnya). Seringkali extension masih jalan meski belum official support versi baru.

**Jika patch tidak work (error di console):** baru buat extension custom.

### Buat Extension Sendiri (jika pihak ketiga tidak ada yang work)

Lihat `references/gnome-extension-authoring.md` untuk **panduan lengkap + template yang bisa langsung dipakai**:
- **Template skeleton** — struktur dasar PanelMenu.Button + timer
- **Template Network Speed** — full working code (baca /proc/net/dev, format speed, toggle bits/bytes)
- **Template Settings** — GSchema XML + compile

Contoh use case: network speed monitor di top panel ketika extension yang ada tidak kompatibel dengan GNOME 50.

---

## GNOME Extension Management

### System Monitor (built-in)

Ubuntu 26.04 GNOME 50 ships with `system-monitor@gnome-shell-extensions.gcampax.github.com`.

```bash
# Cek status
gnome-extensions info system-monitor@gnome-shell-extensions.gcampax.github.com

# Enable/disable
gnome-extensions enable system-monitor@gnome-shell-extensions.gcampax.github.com
gnome-extensions disable system-monitor@gnome-shell-extensions.gcampax.github.com

# Atur tampilan
gsettings set org.gnome.shell.extensions.system-monitor show-download true
gsettings set org.gnome.shell.extensions.system-monitor show-upload true
gsettings set org.gnome.shell.extensions.system-monitor show-cpu true
gsettings set org.gnome.shell.extensions.system-monitor show-memory true
gsettings set org.gnome.shell.extensions.system-monitor show-swap true
```

**Catatan:** Extension ini nampilin ikon, bukan angka speed. Untuk angka download/upload numeric, gunakan extension pihak ketiga seperti `Network Speed Monitor` (lihat di bawah).

### Network Speed Monitor (pihak ketiga)

Extension ID: `netspeed-monitor@ajxv` — menampilkan ↓↑ numeric speed di top panel.

**Install manual:**
```bash
# Clone repo
git clone https://github.com/ajxv/netspeed-monitor.git /tmp/netspeed-monitor

# Copy ke folder extensions
mkdir -p ~/.local/share/gnome-shell/extensions/netspeed-monitor@ajxv
cp -r /tmp/netspeed-monitor/* ~/.local/share/gnome-shell/extensions/netspeed-monitor@ajxv/

# Compile schema
glib-compile-schemas ~/.local/share/gnome-shell/extensions/netspeed-monitor@ajxv/schemas/

# Patch metadata untuk kompatibilitas GNOME 50
# Edit ~/.local/share/gnome-shell/extensions/netspeed-monitor@ajxv/metadata.json
# Tambah "50" ke array shell-version
```

**Setelah install:** user perlu restart GNOME Shell (`Alt+F2` → `r` → Enter) atau logout/login.

**Toggle bits/bytes:** long-press (tekan lama) pada angka speed di panel.

### Extension Manager (aplikasi GUI)

Cara paling mudah browsing & install extension GNOME:

```bash
# Via apt
sudo apt install gnome-shell-extension-manager

# Atau Flatpak
flatpak install flathub com.mattjakeman.ExtensionManager
```

### Extension tidak muncul setelah manual install

1. Cek metadata.json — pastikan `shell-version` mencakup versi GNOME yang dipakai (`gnome-shell --version`).
2. Patch versi jika perlu: tambahkan versi GNOME ke array `shell-version`.
3. Restart GNOME Shell (`Alt+F2` → `r` → Enter).
4. Cek dengan `gnome-extensions list` dan `gnome-extensions info <uuid>`.

### CLI extension management

```bash
# List semua extension
gnome-extensions list

# Info detail
gnome-extensions info <uuid>

# Enable/disable
gnome-extensions enable <uuid>
gnome-extensions disable <uuid>

# Reset
gnome-extensions reset <uuid>
```

---

## Non-Flatpak Apps

For non-Flatpak apps (native `.deb`, AppImage, manual install):

### AppImages Requiring System-Level Setup (Sudo/Udev/Setcap)

If an AppImage (like Sunshine) requires root privileges to install system components (e.g., copying udev rules to `/etc/udev/rules.d/`, loading kernel modules, setting capabilities via `setcap`, or setting up systemd services):
1. **Avoid interactive/piped sudo:** Do not try to run `--install` or pipe passwords to `sudo -S` directly inside the agent session (this is blocked for security).
2. **Extract the AppImage:** Run `<app>.AppImage --appimage-extract` to inspect the internal files (such as `AppRun.wrapped`, configurations, and service files).
3. **Generate a helper script:** Create a shell script (e.g., `install-<app>.sh`) that automates the file copying, udev rules reloading, `setcap` capabilities configuration, user systemd unit creation, and desktop launcher entry creation.
4. **Instruct the user:** Prompt the user to run this helper script in their own local terminal using `sudo` to complete the installation.

### Desktop Integration for Standalone Binaries

1. Create a .desktop file manually in `~/.local/share/applications/`:
   ```ini
   [Desktop Entry]
   Name=App Name
   Exec=/path/to/binary
   Icon=/path/to/icon.png
   Terminal=false
   Type=Application
   Categories=Utility;
   ```

2. Make sure `Exec` is a full path or uses a command findable in `$PATH`.

3. For PATH issues with launchers started from GNOME (not terminal), hardcode the full PATH in a wrapper script (see memory: Hermes Desktop fix).

### Launcher Apps Utilizing Systemd User Services

Some modern applications (like Sunshine) use `.desktop` files that launch the application by starting a systemd user service (e.g., `Exec=systemctl --user start app-name`).

* **Common Failure:** If the app was previously run as a standalone binary or AppImage, there might be a stale custom service file overriding the package-installed systemd unit.
* **Symptom:** Clicking the desktop icon does nothing, but "Run in Terminal" or running the binary directly works. Checking systemd status (`systemctl --user status <service>`) shows execution failure (e.g., status 203/EXEC pointing to a non-existent path).
* **Resolution:**
  1. Check for local user-space service overrides: `ls -la ~/.config/systemd/user/`
  2. Remove the stale override file: `rm ~/.config/systemd/user/<service-name>.service`
  3. Reload the systemd daemon: `systemctl --user daemon-reload`
  4. Enable and start the official service:
     ```bash
     systemctl --user enable <service-name>
     systemctl --user start <service-name>
     ```

---

## Verification

After applying fixes:
```
# Check desktop files are picked up
ls -la ~/.local/share/applications/ | grep <app>

# Check icons exist
ls ~/.local/share/icons/hicolor/48x48/apps/<app-id>.*

# Verify icon cache
gtk-update-icon-cache ~/.local/share/icons/hicolor/ -f
```

## Pitfalls

- **GNOME Shell restart via dbus** may not work on Wayland sessions; use logout/login as the reliable fallback.
- **Icons in `/var/lib/flatpak/exports/share/`** are symlinks to the flatpak sandbox — copying with `cp -L` resolves them to actual files.
- **`NoDisplay=true`** is used by file-handler apps (Wine, etc.) — always check this before assuming the desktop file is missing.
- **Hicolor index.theme** is required for `gtk-update-icon-cache` to succeed; without it the cache won't be created.
