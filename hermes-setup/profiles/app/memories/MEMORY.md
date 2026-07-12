Jangan pernah menjalankan perintah logout, reboot, shutdown, turnoff, restart, systemctl reboot, atau sejenisnya. Jika ada perubahan yang memerlukan restart GNOME Shell, logout, atau reboot sistem — beritahu user untuk melakukannya sendiri secara manual. Jangan eksekusi perintah tersebut.
§
User tidak suka respons yang dipecah jadi beberapa tahap (misal: "Baik, saya lakukan X" lalu di pesan berikutnya "Sekarang saya lakukan Y"). Lebih suka eksekusi langsung dalam satu respons utuh tanpa narasi berlebihan. Langsung kerjakan, jangan staging/pre-announce.
§
LM Studio Qwen3.5 empty stream: reasoning mode tidak bisa di-disable. Solusi terbaik: gunakan Ollama (ollama pull qwen2.5-coder:7b, config: provider=ollama, base_url=http://localhost:11434/v1). Ollama lebih stabil, no reasoning issue, auto-start systemd. LM Studio base_url harus http://127.0.0.1:3000/v1.
§
Odysseus AI agent at /home/yan/odysseus/. Skills: data/skills/general/<name>/SKILL.md (9 required frontmatter fields). Use skill `odysseus-skill-authoring` for format details. prd-architect = global PRD, prd-tauri-v2 = Tauri PRD.
§
Obsidian vault at /home/yan/obsidian/YanVault (git@github.com:WayanDevs/obsidian-vault.git). Notes require frontmatter (tags, category, date, status) in category folders, [[wikilinks]], and emojis. Image resize format: `![alt|width](path)`. Commit/push on every edit.
§
Tauri projects at /home/yan/Dev/ use Vanilla JS, TailwindCSS, Vite, SQLite, and tauri-plugin-shell sidecars (yt-dlp, ffmpeg) in binaries/ target triple filenames.