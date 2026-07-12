---
name: obsidian
description: Read, search, create, and edit notes in the Obsidian vault.
platforms: [linux, macos, windows]
---

# Obsidian Vault

Use this skill for filesystem-first Obsidian vault work: reading notes, listing notes, searching note files, creating notes, appending content, and adding wikilinks.

## Vault path

Use a known or resolved vault path before calling file tools.

The documented vault-path convention is the `OBSIDIAN_VAULT_PATH` environment variable, for example from `${HERMES_HOME:-~/.hermes}/.env`. If it is unset, use `~/Documents/Obsidian Vault`.

File tools do not expand shell variables. Do not pass paths containing `$OBSIDIAN_VAULT_PATH` to `read_file`, `write_file`, `patch`, or `search_files`; resolve the vault path first and pass a concrete absolute path. Vault paths may contain spaces, which is another reason to prefer file tools over shell commands.

If the vault path is unknown, `terminal` is acceptable for resolving `OBSIDIAN_VAULT_PATH` or checking whether the fallback path exists. Once the path is known, switch back to file tools.

## Read a note

Use `read_file` with the resolved absolute path to the note. Prefer this over `cat` because it provides line numbers and pagination.

## List notes

Use `search_files` with `target: "files"` and the resolved vault path. Prefer this over `find` or `ls`.

- To list all markdown notes, use `pattern: "*.md"` under the vault path.
- To list a subfolder, search under that subfolder's absolute path.

## Search

Use `search_files` for both filename and content searches. Prefer this over `grep`, `find`, or `ls`.

- For filenames, use `search_files` with `target: "files"` and a filename `pattern`.
- For note contents, use `search_files` with `target: "content"`, the content regex as `pattern`, and `file_glob: "*.md"` when you want to restrict matches to markdown notes.

## Create a note

Use `write_file` with the resolved absolute path and the full markdown content. Prefer this over shell heredocs or `echo` because it avoids shell quoting issues and returns structured results.

### Note structure conventions

When creating notes for this vault, follow these conventions:

1. **Folder organization**: Organize notes in category-based folders (e.g., `Ubuntu/`, `Hermes Desktop/`, `Python/`). Do not create notes at vault root unless they are index/MOC files.

2. **Frontmatter**: Every note should include YAML frontmatter with:
   ```yaml
   ---
   tags: [tag1, tag2, tag3]
   category: Category Name
   date: YYYY-MM-DD
   status: (in-progress|solved|archived)
   ---
   ```

3. **Wikilinks**: Connect related notes using `[[Note Name]]` syntax. When creating a note, identify and link to related existing notes in the vault.

4. **Visual structure**: Use emoji for section headers to improve scannability:
   - 🔍 for "Symptoms" or "Problem" sections
   - 🛠️ for "Root Cause" or "Analysis" sections  
   - ✅ for "Solution" or "Results" sections
   - 🧪 for "Verification" or "Testing" sections
   - 📚 for "Lessons" or "Learning" sections
   - 📁 for "Files Modified" sections
   - 🔗 for "Related" or "References" sections

5. **Hashtags**: Include relevant hashtags at the bottom of notes for additional discoverability (e.g., `#ubuntu #troubleshooting #desktop`).

## Append to a note

Prefer a native file-tool workflow when it is not awkward:

- Read the target note with `read_file`.
- Use `patch` for an anchored append when there is stable context, such as adding a section after an existing heading or appending before a known trailing block.
- Use `write_file` when rewriting the whole note is clearer than constructing a fragile patch.

For an anchored append with `patch`, replace the anchor with the anchor plus the new content.

For a simple append with no stable context, `terminal` is acceptable if it is the clearest safe option.

## Targeted edits

Use `patch` for focused note changes when the current content gives you stable context. Prefer this over shell text rewriting.

## Wikilinks

Obsidian links notes with `[[Note Name]]` syntax. When creating notes, use these to link related content.

## Import external documentation

When importing or scraping external documentation repositories (e.g., Tauri, Astro, or Docusaurus) to save into the Obsidian vault, use the workflow described in `references/import-external-docs.md`. This details cloning, MDX-to-MD cleaning, sanitizing custom JSX elements, restructuring frontmatter to meet vault guidelines, and committing to git.

## System installation documentation

When documenting system package installations, configurations, or setup processes, use the template and pattern in `references/system-installation-docs.md`. This ensures consistent structure for system administration notes.
