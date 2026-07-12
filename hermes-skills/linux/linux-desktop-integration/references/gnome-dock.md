# GNOME Dock Customization (Ubuntu Dock / Dash to Dock)

> Umbrella: `linux-desktop-integration` — GNOME desktop environment tweaks.

Gunakan `gsettings` pada schema `org.gnome.shell.extensions.dash-to-dock` (Ubuntu Dock memakai schema yang sama dengan Dash to Dock).

## macOS-style centered dock

```bash
# Pindah ke bawah
gsettings set org.gnome.shell.extensions.dash-to-dock dock-position BOTTOM

# Center icons
gsettings set org.gnome.shell.extensions.dash-to-dock always-center-icons true

# Dock mengecil ngikutin jumlah icon
gsettings set org.gnome.shell.extensions.dash-to-dock extend-height false

# Show apps button ikut di tengah
gsettings set org.gnome.shell.extensions.dash-to-dock show-apps-always-in-the-edge false
```

## Windows 11-style taskbar (full width, icons center, solid black)

```bash
# Full width kiri-kanan
gsettings set org.gnome.shell.extensions.dash-to-dock extend-height true

# Icons di tengah
gsettings set org.gnome.shell.extensions.dash-to-dock always-center-icons true

# Solid background (hitam)
gsettings set org.gnome.shell.extensions.dash-to-dock transparency-mode FIXED
gsettings set org.gnome.shell.extensions.dash-to-dock background-color 'rgba(0,0,0,1)'

# Show apps button tetap di ujung kanan (seperti Windows start)
gsettings set org.gnome.shell.extensions.dash-to-dock show-apps-always-in-the-edge true
gsettings set org.gnome.shell.extensions.dash-to-dock show-show-apps-button true
```

## Other useful toggles

| Setting | Description |
|---------|-------------|
| `autohide` | Auto-hide dock when not hovering |
| `intellihide` | Hide dock when a window overlaps |
| `dock-position` | `LEFT` / `RIGHT` / `BOTTOM` |
| `dash-max-icon-size` | Max icon size in pixels |
| `show-running` | Show only running apps (vs all favorites) |
| `show-favorites` | Show pinned favorites |
| `click-action` | `CYCLE`/`MINIMIZE`/`PREVIEW` on click |

## Verifikasi

```bash
gsettings list-recursively org.gnome.shell.extensions.dash-to-dock | grep -E 'extend-height|always-center-icons|transparency-mode|background-color|show-apps'
```

## Pitfalls

- **GNOME Shell restart:** Perubahan langsung terasa, tidak perlu restart shell.
- **`background-color`** menggunakan format `'rgba(r,g,b,a)'`, bukan hex `#000000`.
- **Wayland vs X11:** Semua perintah di atas work di kedua session.
- **Schema name** `dash-to-dock` dipakai juga oleh Ubuntu Dock meski ekstensinya bernama `ubuntu-dock@ubuntu.com`.
