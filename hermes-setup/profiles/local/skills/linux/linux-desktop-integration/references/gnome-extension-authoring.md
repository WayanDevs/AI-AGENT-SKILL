# GNOME Shell Extension Authoring

> Umbrella: `linux-desktop-integration` — Membuat custom extension untuk GNOME Shell.

## Struktur Folder

```
~/.local/share/gnome-shell/extensions/<uuid>/
├── extension.js        # Kode utama extension
├── metadata.json       # Nama, versi, kompatibilitas
└── schemas/            # Settings schema (opsional)
    ├── org.gnome.shell.extensions.<name>.gschema.xml
    └── gschemas.compiled
```

## Full Working Example: Network Speed Monitor (GNOME 50)

Extension lengkap untuk menampilkan ↓ download / ↑ upload speed di top panel, update tiap 2 detik, dengan long-press toggle bits/bytes.

### extension.js

```javascript
'use strict';

import GObject from 'gi://GObject';
import St from 'gi://St';
import GLib from 'gi://GLib';
import Clutter from 'gi://Clutter';
import Gio from 'gi://Gio';

import { Extension } from 'resource:///org/gnome/shell/extensions/extension.js';
import * as Main from 'resource:///org/gnome/shell/ui/main.js';
import * as PanelMenu from 'resource:///org/gnome/shell/ui/panelMenu.js';

const UPDATE_INTERVAL = 2;
const IGNORED_INTERFACES = ['lo', 'vir', 'vbox', 'docker', 'br-', 'lxc', 'veth'];

const Indicator = GObject.registerClass(
class NetSpeedIndicator extends PanelMenu.Button {
    _init(settings) {
        super._init(0.0, 'Net Speed', false);

        this._settings = settings;
        this._prevRx = 0;
        this._prevTx = 0;
        this._timer = null;

        this._label = new St.Label({
            text: '↓ 0 KB/s ↑ 0 KB/s',
            y_align: Clutter.ActorAlign.CENTER,
            style_class: 'panel-button',
        });
        this.add_child(this._label);

        this._netDevFile = Gio.File.new_for_path('/proc/net/dev');

        // Long-press toggle bits/bytes
        this._clickTime = 0;
        this.connect('button-press-event', (actor, event) => {
            this._clickTime = GLib.get_monotonic_time();
            return Clutter.EVENT_PROPAGATE;
        });
        this.connect('button-release-event', (actor, event) => {
            const elapsed = (GLib.get_monotonic_time() - this._clickTime) / 1000;
            if (elapsed >= 800) {
                const cur = this._settings.get_boolean('use-bits');
                this._settings.set_boolean('use-bits', !cur);
                this._refresh();
            }
            return Clutter.EVENT_PROPAGATE;
        });
    }

    _formatSpeed(bytesPerSec) {
        const useBits = this._settings.get_boolean('use-bits');
        let val = useBits ? bytesPerSec * 8 : bytesPerSec;
        const base = useBits ? 1000 : 1024;
        const units = useBits
            ? ['bps', 'Kbps', 'Mbps', 'Gbps']
            : ['B/s', 'KB/s', 'MB/s', 'GB/s'];
        let i = 0;
        while (val >= base && i < units.length - 1) { val /= base; i++; }
        return `${val.toFixed(1)} ${units[i]}`;
    }

    _ignored(iface) {
        return IGNORED_INTERFACES.some(p => iface.startsWith(p));
    }

    _readStats() {
        try {
            const [ok, contents] = this._netDevFile.load_contents(null);
            if (!ok) return null;

            const lines = new TextDecoder().decode(contents).split('\n');
            let rx = 0, tx = 0;

            for (let i = 2; i < lines.length; i++) {
                const line = lines[i].trim();
                if (!line) continue;
                const parts = line.split(':');
                if (parts.length < 2) continue;
                if (this._ignored(parts[0].trim())) continue;

                const nums = parts[1].trim().split(/\s+/).map(n => parseInt(n, 10));
                rx += nums[0] || 0;   // RX bytes = kolom 2
                tx += nums[8] || 0;   // TX bytes = kolom 10
            }
            return { rx, tx };
        } catch (e) {
            log('NetSpeed: error reading stats: ' + e);
            return null;
        }
    }

    _refresh() {
        const stats = this._readStats();
        if (!stats) return;

        if (this._prevRx === 0 && this._prevTx === 0) {
            this._prevRx = stats.rx;
            this._prevTx = stats.tx;
            return;
        }

        const down = Math.max(0, (stats.rx - this._prevRx) / UPDATE_INTERVAL);
        const up   = Math.max(0, (stats.tx - this._prevTx) / UPDATE_INTERVAL);

        this._label.text = `↓ ${this._formatSpeed(down)} ↑ ${this._formatSpeed(up)}`;

        this._prevRx = stats.rx;
        this._prevTx = stats.tx;
    }

    start() {
        this._refresh();
        this._timer = GLib.timeout_add_seconds(
            GLib.PRIORITY_DEFAULT,
            UPDATE_INTERVAL,
            () => { this._refresh(); return GLib.SOURCE_CONTINUE; }
        );
    }

    stop() {
        if (this._timer) {
            GLib.source_remove(this._timer);
            this._timer = null;
        }
    }
});

export default class NetSpeedExtension extends Extension {
    enable() {
        this._settings = this.getSettings();
        this._indicator = new Indicator(this._settings);
        Main.panel.addToStatusArea('net-speed', this._indicator, 0, 'right');
        this._indicator.start();
    }

    disable() {
        if (this._indicator) {
            this._indicator.stop();
            this._indicator.destroy();
            this._indicator = null;
        }
        this._settings = null;
    }
}
```

### metadata.json

```json
{
  "name": "Network Speed",
  "description": "Real-time network speed in top panel",
  "uuid": "netspeed@custom",
  "shell-version": ["45", "46", "47", "48", "49", "50"],
  "version": 1,
  "settings-schema": "org.gnome.shell.extensions.netspeed"
}
```

### Settings Schema

`schemas/org.gnome.shell.extensions.netspeed.gschema.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<schemalist>
  <schema path="/org/gnome/shell/extensions/netspeed/"
          id="org.gnome.shell.extensions.netspeed">
    <key type="b" name="use-bits">
      <default>false</default>
      <summary>Display in bits</summary>
      <description>Show speed in bits per second instead of bytes.</description>
    </key>
  </schema>
</schemalist>
```

### Install

```bash
# 1. Buat folder
mkdir -p ~/.local/share/gnome-shell/extensions/netspeed@custom/schemas

# 2. Copy extension.js + metadata.json + schema XML

# 3. Compile schema
glib-compile-schemas ~/.local/share/gnome-shell/extensions/netspeed@custom/schemas/

# 4. Restart GNOME Shell (Alt+F2 → r → Enter)
# 5. Enable via gnome-extensions enable netspeed@custom
```

---

## Template Minimum — Skeleton

Extension ini menampilkan ↓ download / ↑ upload speed di top panel.

### 1. metadata.json

```json
{
  "name": "My Extension Name",
  "description": "Deskripsi singkat",
  "uuid": "my-extension@username",
  "shell-version": [
    "45", "46", "47", "48", "49", "50"
  ],
  "url": "",
  "version": 1,
  "settings-schema": "org.gnome.shell.extensions.my-extension"
}
```

### 2. extension.js

Template minimal yang bisa dipakai untuk berbagai keperluan:

```javascript
'use strict';

import GObject from 'gi://GObject';
import St from 'gi://St';
import GLib from 'gi://GLib';
import Clutter from 'gi://Clutter';
import Gio from 'gi://Gio';

import { Extension } from 'resource:///org/gnome/shell/extensions/extension.js';
import * as Main from 'resource:///org/gnome/shell/ui/main.js';
import * as PanelMenu from 'resource:///org/gnome/shell/ui/panelMenu.js';

// === KUSTOMISASI DI SINI ===
const UPDATE_INTERVAL = 2; // detik
// ==========================

const Indicator = GObject.registerClass(
class MyIndicator extends PanelMenu.Button {
    _init(settings) {
        super._init(0.0, 'My Indicator', false);
        this._settings = settings;
        this._timer = null;

        this._label = new St.Label({
            text: 'Loading...',
            y_align: Clutter.ActorAlign.CENTER,
            style_class: 'panel-button',
        });
        this.add_child(this._label);
    }

    // === LOGIKA UPDATE DI SINI ===
    _refresh() {
        // Baca data, update this._label.text
        this._label.text = '↓ 10 MB/s ↑ 2 MB/s';
    }
    // ==============================

    start() {
        this._refresh();
        this._timer = GLib.timeout_add_seconds(
            GLib.PRIORITY_DEFAULT,
            UPDATE_INTERVAL,
            () => {
                this._refresh();
                return GLib.SOURCE_CONTINUE;
            }
        );
    }

    stop() {
        if (this._timer) {
            GLib.source_remove(this._timer);
            this._timer = null;
        }
    }
});

export default class MyExtension extends Extension {
    enable() {
        this._settings = this.getSettings();
        this._indicator = new Indicator(this._settings);
        Main.panel.addToStatusArea('my-indicator', this._indicator, 0, 'right');
        this._indicator.start();
    }

    disable() {
        if (this._indicator) {
            this._indicator.stop();
            this._indicator.destroy();
            this._indicator = null;
        }
        this._settings = null;
    }
}
```

### 3. Settings Schema (opsional)

`schemas/org.gnome.shell.extensions.my-extension.gschema.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<schemalist>
  <schema path="/org/gnome/shell/extensions/my-extension/" id="org.gnome.shell.extensions.my-extension">
    <key type="b" name="use-bits">
      <default>false</default>
      <summary>Display speed in bits</summary>
      <description>Whether to display in bits per second instead of bytes.</description>
    </key>
  </schema>
</schemalist>
```

### 4. Install & Aktifkan

```bash
# Compile schema (wajib jika pakai settings)
glib-compile-schemas ~/.local/share/gnome-shell/extensions/<uuid>/schemas/

# Enable via CLI (gagal? berarti perlu restart shell dulu)
gnome-extensions enable <uuid>

# Atau enable via dconf
dconf write /org/gnome/shell/enabled-extensions "$(dconf read /org/gnome/shell/enabled-extensions | sed 's/.$//')'<uuid>']"
```

### 5. Restart GNOME Shell

User harus lakukan sendiri:
- `Alt+F2`, ketik `r`, Enter
- Atau logout/login

## API Penting untuk GNOME 45+

| Import | Path | Kegunaan |
|--------|------|----------|
| `Extension` | `resource:///org/gnome/shell/extensions/extension.js` | Base class extension |
| `Main.panel` | `resource:///org/gnome/shell/ui/main.js` | Panel utama |
| `PanelMenu.Button` | `resource:///org/gnome/shell/ui/panelMenu.js` | Button di panel |
| `St.Label` | `gi://St` | Widget teks |
| `Gio.File` | `gi://Gio` | Baca file (/proc/net/dev, dll) |
| `GLib.timeout_add_seconds` | `gi://GLib` | Timer periodik |

## Pitfalls

- **Gagal enable = butuh restart shell.** Setelah copy manual extension, `gnome-extensions enable` sering gagal karena cache belum di-refresh. Restart shell dulu.
- **PanelMenu.Button vs St.Label.** `PanelMenu.Button` adalah cara resmi menambah item ke panel. Jangan pakai `Main.panel._rightBox.insert_child_at_index` karena private API bisa berubah.
- **`addToStatusArea` param:** nama unik, widget, posisi (0 = kiri), region ('right' = kanan). Pastikan nama unik antar extension.
- **`getSettings()`** otomatis membaca schema dari `settings-schema` di metadata.json. Pastikan schema sudah di-compile.
- **`Gio.File.load_contents()`** adalah synchronous — untuk file kecil seperti /proc/net/dev ini OK. Untuk file besar, pakai `load_contents_async()`.
