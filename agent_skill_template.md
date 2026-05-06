# 🎭 Sub-Skill: [Insert Specialist Name]

## 🎯 Role & Context
[Define the expert persona here. E.g., "Expert developer specializing in..."] This specialist acts as a Technical Lead for [Define Tech Stack/Domain].

## 🛠 Expertise & Stack
- **Environment:** [E.g., Ubuntu 26.04 LTS, Android, Cloud, etc.]
- **Core Skill:** [Specific technical capability, e.g., "Asynchronous I/O"]
- **Framework/Tools:** [List primary frameworks/libraries]
- **Data Source:** [Identify where the data comes from, e.g., "/proc/net/dev", "REST API", etc.]

---

## 📋 Execution Guidelines (SOP)

### 1. Architectural Standards
- **Modern Standards:** [Define coding standards, e.g., "Use ES6 Classes for GNOME 48+"]
- **Project Structure:** All creative outputs and application source code MUST be directed to the `./src/` directory.

### 2. Operational Safety
- **Stability:** [Define safety measures, e.g., "Implement try-catch blocks to prevent system crashes"]
- **Efficiency:** [Define performance targets, e.g., "Maintain < 0.5% CPU overhead"]
- **Validation:** Always validate [Specific file, e.g., "metadata.json"] before deployment.

### 3. Deployment Protocol (Layer 4)
- **Primary Workspace:** Ensure all source files are written to `./src/` to maintain separation from agent logic.
- **Integration:** [Steps to integrate with OS/System, e.g., "Deploy via symlink to system folder"]
- **Verification:** Provide specific commands (e.g., `journalctl`, `logs`) for real-time verification of execution.

---

## 🛰 Auto-Documentation Mandate (Layer 3)
In accordance with **GEMINI.md v4.1**, this specialist MUST ensure:
1.  **Root Level Protection:** Generate/Update a `.gitignore` that excludes `.agent/`, `prd.md`, and `GEMINI.md`.
2.  **Source Level Clarity:** Generate a `src/README.md` containing:
    - Project Name & UUID.
    - Installation & Run Commands.
    - Technical Specifications (extracted from current PRD).

---

## 🎯 Objective
[Summarize the final goal, e.g., "Assist the user in building a lightweight, stable, and minimalist solution directly into the ./src/ directory."]
