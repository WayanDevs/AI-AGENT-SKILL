# 📋 PROJECT PRD: [Project Name]

@version: 1.0
@status: draft/final
@priority: p1
@tech_stack: [e.g., Flutter Desktop / GNOME GJS]

---

## 🎯 1. EXECUTIVE SUMMARY
**Goal**: [Jelaskan dalam 1 kalimat apa yang ingin dicapai aplikasi ini].
**Audience**: [Siapa penggunanya? e.g., Ubuntu 26.04 Power Users].
**Value**: [Kenapa aplikasi ini dibuat? e.g., Low-latency system monitoring].

---

## 🛠 2. CORE SPECIFICATIONS (The "What")
*   **Feature A**: [Deskripsi singkat].
    *   *Requirement*: [e.g., Harus update tiap 1 detik].
*   **Feature B**: [Deskripsi singkat].
    *   *Constraint*: [e.g., Tidak boleh menggunakan lebih dari 10MB RAM].

---

## 🏗 3. TECHNICAL CONSTRAINTS (Layer 2 Constraints)
*   **Workspace**: All source code MUST reside in `./src/`.
*   **Security**: [e.g., No root access required].
*   **Performance**: [e.g., Non-blocking asynchronous I/O only].

---

## 🎨 4. UI/UX REQUIREMENTS
*   **Layout**: [e.g., GNOME Top Bar / Sidebar / Floating Window].
*   **Theming**: [e.g., Adwaita Dark / Custom CSS].
*   **Interactions**: [e.g., Click to toggle, Right-click for settings].

---

## 💾 5. DATA & INTERFACES
*   **Data Source**: [e.g., /proc/net/dev, Web API, Local Database].
*   **External Dependencies**: [e.g., GLib, libnm, Flutter Provider].

---

## 🚀 6. DEFINITION OF DONE (DoD)
- [ ] [Kriteria 1: e.g., Berhasil di-build tanpa error].
- [ ] [Kriteria 2: e.g., Muncul di Top Bar dengan data real-time].
- [ ] [Kriteria 3: e.g., Lolos validasi memory leak selama 1 jam].

---

## ⚠️ 7. RISK & MITIGATION
*   **Risk**: [e.g., GNOME Shell updates might break API].
*   **Mitigation**: [e.g., Use stable Libadwaita components].
