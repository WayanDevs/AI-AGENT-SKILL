# Linux Desktop App Crash Diagnosis

When a Tauri (or any Rust/GTK) app closes instantly without visible error:

## 1. systemd user journal (BEST source)
```bash
journalctl --user -b -0 --no-pager | grep -i <app-name>
```
Shows: Rust panics, SIGABRT stack traces, process lifecycle (start/stop/crash).
GNOME launcher captures app stdout/stderr here automatically.

## 2. Apport crash files
```bash
ls -la /var/crash/*<app-name>*
# Read header fields:
grep -E "Signal:|Date:|Title:" /var/crash/_usr_bin_<name>.*.crash
```
- Signal 6 = SIGABRT = Rust panic / abort()
- Signal 11 = SIGSEGV = segfault

## 3. Run from terminal with backtrace
```bash
# If the app needs a display (Wayland/X11):
DISPLAY=:0 RUST_BACKTRACE=1 /usr/bin/<binary> 2>&1 | tee /tmp/crash.log
```
Find display: `ps -u $USER -F | grep -E "wayland|Xorg|Xwayland"`

## 4. dmesg (requires root)
```bash
sudo dmesg -T | grep -i <app-name>
# or
sudo dmesg -T | tail -20
```
Shows kernel-level signals (OOM kills, segfaults with instruction pointer).

## Common Tauri crash patterns found via journalctl

| Pattern | Meaning |
|---------|---------|
| `panicked at src/main.rs:NNN` + `there is no reactor running` | `tokio::spawn` inside sync `fn` command → fix: make it `async fn` |
| `panicked at ... unwrap() on Err/None` | Unhandled error in unwrap → add proper error handling |
| `thread caused non-unwinding panic. aborting.` | Double-panic or panic crossing FFI boundary |
| `Signal: 6` in crash file | SIGABRT from Rust panic |
| Process starts and stops within 1-5 seconds | Crash during initialization or first user action |
