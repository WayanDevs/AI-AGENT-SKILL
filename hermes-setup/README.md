# Hermes Agent Setup & Profiles

Backup konfigurasi Hermes Agent — profiles, memories, skills, dan settings.

## Struktur

```
hermes-setup/
├── profiles/
│   ├── default/          # Profile utama (home)
│   │   ├── config.yaml   # Konfigurasi utama
│   │   ├── SOUL.md       # System prompt
│   │   └── memories/     # MEMORY.md & USER.md
│   ├── app/              # Profile untuk app development
│   │   ├── config.yaml
│   │   ├── SOUL.md
│   │   ├── memories/
│   │   └── skills/       # 102 skills
│   ├── lmstudio/         # Profile untuk LM Studio
│   │   ├── SOUL.md
│   │   └── skills/       # 75 skills
│   ├── local/            # Profile untuk local LLM (Ollama)
│   │   ├── config.yaml
│   │   ├── SOUL.md
│   │   ├── memories/
│   │   └── skills/       # 84 skills
│   └── tauri/            # Profile untuk Tauri development
│       ├── config.yaml
│       ├── SOUL.md
│       ├── memories/
│       └── skills/       # 18 skills
```

## Catatan

- File `config.yaml` sudah di-sanitize (tidak mengandung API keys)
- Skills default profile ada di `hermes-skills/` (root repo)
- Setiap profile punya konfigurasi model & provider yang berbeda
