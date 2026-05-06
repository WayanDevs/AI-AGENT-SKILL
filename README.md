# AI-AGENT-SKILL
my personal ai agent skills


# 🚀 AI-AGENT-SKILL

Welcome to the **AI-AGENT-SKILL** repository by [@WayanDevs](https://github.com/WayanDevs)!

This repository serves as the central brain and skill-set hub for an advanced AI orchestration agent. It is designed around the **GEMINI (v1)** architecture, employing a structured 5-layer hierarchy to translate human intent into deterministic, safe, and effective execution.

## 🧱 The 5-Layer Architecture

The logic within this repository follows a strict execution hierarchy:

1. **Layer 0 — Guardrails (The Gatekeeper)**: Enforces safety, requiring clarification for ambiguous tasks and ensuring destructive commands are blocked.
2. **Layer 1 — Skill Selection (Discovery)**: Prioritizes local `.agent/skills.md` to map the appropriate skill to the current intent.
3. **Layer 2 — Directive (SOP)**: Natural language Standard Operating Procedures that define *what* needs to happen (Goal, Inputs, Edge Cases).
4. **Layer 3 — Orchestration**: Manages workspace context, auto-documents changes, and ensures execution occurs purely inside a designated `./src/` environment.
5. **Layer 4 — Execution**: Executes deterministic, repeatable tool commands without unwanted side effects.

## 📂 Repository Structure

This repository includes the core configuration and template files required to define the AI Agent's behavior:

- **`GEMINI.md`**: The global orchestration rulebook. It defines the system role, operating principles, and constraints the AI must follow globally.
- **`skills.md`**: The active index or registry of skills available for the agent to use. 
- **`agent_skill_template.md`**: A blueprint/template to create new, standardized agent skills.
- **`prd_template.md`**: A Product Requirements Document (PRD) template tailored for AI understanding, helping define clear technical constraints, goals, and definitions of done (DoD) before execution begins.

## 🛠️ Operating Principles

- **The Pragmatic Colleague**: The agent will not ask redundant questions if answers exist in the `prd_template.md` or directives.
- **Clean Repository Mandate**: The agent isolates logic from source code. Agent configuration files stay in the root or `.agent/` directory, while all project source code is pushed to `./src/`.
- **Self-Annealing Loop**: Identifies errors in tools or logic and adapts directives accordingly to prevent future failures.

## 🚀 Getting Started

1. Place the `GEMINI.md` file in the appropriate global context for your AI Agent (e.g., as global instructions or system prompt rules).
2. Start a new task by defining the requirements using `prd_template.md`.
3. If the agent needs new capabilities, create them based on the `agent_skill_template.md` and add them to `skills.md`.
4. Let the agent build, test, and document your projects!

---

## Example root for antigravity at my linux ubuntu

gemini.md at /home/username/.gemini/GEMINI.md

agent_skill_template.md at /home/usernamr/Public/.agent/skills/linux_system_specialist.md

prd_templae at /home/username/Public/.linux/app/network_speed_meter/prd.md

skills.md at /home/username/Public/.linux/app/network_speed_meter/.agent/skills.md


*For more information, please visit [WayanDevs/AI-AGENT-SKILL](https://github.com/WayanDevs/AI-AGENT-SKILL/)*
