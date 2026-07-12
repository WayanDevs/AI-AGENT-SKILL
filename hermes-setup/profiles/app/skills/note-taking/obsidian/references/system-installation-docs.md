# System Installation Documentation Pattern

Pattern for documenting system installations and configurations in Obsidian vault.

## Structure

**Target folder**: Organize by OS/system category (e.g., `ubuntu/`, `debian/`, `arch/`)

**File naming**: `<Package Name> Installation.md`

## Template

```markdown
---
tags: [os-name, package-name, package-manager, installation]
category: os-name
date: YYYY-MM-DD
status: completed
---

# 📦 Package Name Installation

## 🔍 Overview
Brief description of what the package does and why it's useful.

## 🛠️ Installation Process

### System Requirements
- OS requirements
- Dependencies
- Other prerequisites

### Installation Steps

**1. Install Package**
```bash
# Installation commands with exact syntax
```

**Packages Installed:**
- `package-name` (version)
- `dependency-1` (version)
- `dependency-2` (version)

**2. Configuration (if needed)**
```bash
# Config commands
```

**3. Verify Installation**
```bash
# Verification commands
```

## ✅ Installation Status

- **Version:** X.Y.Z
- **Installation Date:** YYYY-MM-DD
- **Status:** ✅ Successfully installed and configured

## 📚 Usage Commands

### Common Command 1
```bash
command-example
```

### Common Command 2
```bash
command-example
```

## 🎯 Examples (if applicable)

```bash
# Real-world usage examples
```

## 🔗 Related Notes
- [[Related Note 1]]
- [[Related Note 2]]

## 📝 Notes
- Important notes
- Known issues
- Tips and tricks
```

## Key Points

1. **Complete versions**: Always capture exact version numbers from installation output
2. **Real commands**: Include the actual commands used, not generic examples
3. **Verification**: Always include and run verification commands
4. **Status tracking**: Use status field (completed/in-progress/failed)
5. **Cross-linking**: Link to related system setup or troubleshooting notes

## When to Use

- After successfully installing system packages (apt, dnf, pacman, flatpak, snap)
- After configuring system services
- After setting up development tools
- When documenting reproducible system setup steps
