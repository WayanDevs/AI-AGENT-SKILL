# Hermes Desktop Launcher PATH Fix (GNOME/Wayland)

## Symptom
- Hermes Desktop launches from terminal: `hermes-desktop` works
- Hermes Desktop launches via `gio launch` or `gtk-launch`
- **FAILS** when launched from GNOME Show Applications menu
- No visible error, just loading spinner then disappears

## Root Cause
**PATH mismatch**: Terminal shell has NVM/pyenv/conda paths, GNOME launcher does not.

When Hermes Desktop checks for `npm`, it fails silently because:
```
Terminal PATH:     /home/user/.nvm/versions/node/vXX.XX.X/bin:...
GNOME Launcher PATH:  /usr/bin:/bin  (minimal system PATH)
```

Error in logs:
```
Desktop GUI requires Node.js/npm, but npm was not found on PATH.
```

## Debugging
Add logging to launcher script:
```bash
#!/usr/bin/env bash
{
    echo "===== $(date) ====="
    echo "USER=$USER"
    echo "PATH=$PATH"
    echo "NODE=$(which node 2>&1)"
    echo "NPM=$(which npm 2>&1)"
} >> /tmp/hermes-launch.log

# ... rest of script
```

Check logs after launch attempt:
```bash
cat /tmp/hermes-launch.log
```

## Fix

### 1. Update launcher script
File: `~/.local/bin/hermes-desktop`

```bash
#!/usr/bin/env bash

# Hardcode PATH with NVM (adjust version as needed)
export PATH="$HOME/.nvm/versions/node/v24.16.0/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

cd "$HOME" || exit 1

exec /home/user/.hermes/hermes-agent/venv/bin/hermes desktop
```

Make executable:
```bash
chmod +x ~/.local/bin/hermes-desktop
```

### 2. Update desktop entry
File: `~/.local/share/applications/com.hermes.desktop`

```ini
[Desktop Entry]
Type=Application
Name=Hermes Desktop
Exec=/home/user/.local/bin/hermes-desktop
Icon=/home/user/.hermes/hermes-agent/apps/desktop/public/hermes.png
Terminal=false
Categories=Development;
StartupNotify=false
StartupWMClass=Hermes
```

Key fields:
- `StartupWMClass=Hermes` - Fixes icon showing as gear/settings
- `StartupNotify=false` - Smoother launch experience
- Rename to `com.hermes.desktop` (reverse-DNS convention)

### 3. Refresh desktop database
```bash
update-desktop-database ~/.local/share/applications
```

## Verification
```bash
# Test all launch methods
hermes-desktop                                    # Terminal
gio launch ~/.local/share/applications/com.hermes.desktop   # GIO
gtk-launch com.hermes                            # GTK
# Click "Hermes Desktop" in Show Applications     # GUI
```

## General Pattern
**Any app that works in terminal but fails from GUI launcher**: PATH/environment mismatch.

Common culprits:
- NVM (Node.js)
- pyenv (Python)
- asdf (version manager)
- Conda (data science)
- SDKMAN (Java)
- rbenv (Ruby)

Solution: Hardcode required PATHs in the launcher script, not in shell rc files.

## Alternative: System-wide Node
Install Node.js system-wide to avoid PATH issues:
```bash
sudo apt install nodejs npm
```

But this loses NVM's version management.
