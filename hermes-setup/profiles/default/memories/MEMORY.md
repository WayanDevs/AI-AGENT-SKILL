Obsidian vault: /home/yan/obsidian/YanVault — gunakan untuk dokumentasi proyek, catatan debugging, dan referensi teknis.
§
Obsidian vault /home/yan/obsidian/YanVault → git@github.com:WayanDevs/obsidian-vault.git via SSH. Commit & push setiap habis buat/sunting catatan.
§
Dokumentasi Obsidian vault: semua catatan di /home/yan/obsidian/YanVault harus diorganisir dalam folder berdasarkan kategori (ubuntu, hermes, dll). Setiap catatan harus memiliki frontmatter dengan tags, category, date, status. Gunakan [[wikilink]] untuk menghubungkan catatan yang terkait. Format markdown standar dengan emoji emoji untuk struktur visual (🔍, 🛠️, ✅).
§
Obsidian image resize syntax: `![Description|400](path/to/image.jpg)` bukan `{width=600}`. Format: `![alt-text|width-px](image-path)`. Width dalam pixel, cocok dengan Local Image Plus plugin.
§
Jangan pernah menjalankan perintah logout, reboot, shutdown, turnoff, restart, systemctl reboot, atau sejenisnya. Jika ada perubahan yang memerlukan restart GNOME Shell, logout, atau reboot sistem — beritahu user untuk melakukannya sendiri secara manual. Jangan eksekusi perintah tersebut.
§
User tidak suka respons yang dipecah jadi beberapa tahap (misal: "Baik, saya lakukan X" lalu di pesan berikutnya "Sekarang saya lakukan Y"). Lebih suka eksekusi langsung dalam satu respons utuh tanpa narasi berlebihan. Langsung kerjakan, jangan staging/pre-announce.
§
LM Studio Qwen3.5 empty stream: reasoning mode tidak bisa di-disable. Solusi terbaik: gunakan Ollama (ollama pull qwen2.5-coder:7b, config: provider=ollama, base_url=http://localhost:11434/v1). Ollama lebih stabil, no reasoning issue, auto-start systemd. LM Studio base_url harus http://127.0.0.1:3000/v1.
§
Odysseus AI agent at /home/yan/odysseus/. Skills: data/skills/general/<name>/SKILL.md (9 required frontmatter fields). Use skill `odysseus-skill-authoring` for format details. prd-architect = global PRD, prd-tauri-v2 = Tauri PRD.