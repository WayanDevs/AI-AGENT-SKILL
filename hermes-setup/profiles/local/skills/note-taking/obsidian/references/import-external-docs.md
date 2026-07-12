# Importing External Git/Doc Repositories to Obsidian

This reference guide outlines the workflow for scraping, converting, and importing external documentation repositories (e.g., Docusaurus, Starlight, Astro-based MDX docs) into an Obsidian vault.

## Workflow Steps

### 1. Clone the Source Repository
Clone the repository to a temporary directory using a shallow clone to save time and bandwidth:
```bash
git clone --depth 1 https://github.com/org/repo-docs.git /tmp/repo-docs
```

### 2. Identify the Documentation Root and Excluded Paths
Explore the cloned repository to find where markdown files are stored. Common locations include:
- `docs/`
- `src/content/docs/` (Astro/Starlight)
- `website/docs/` (Docusaurus)

Also identify non-English folders (e.g., `zh-cn`, `ko`, `ja`, `es`) or fragment directories to exclude them if importing a single language.

### 3. Parse and Clean the Documentation
Use a Python script (e.g., in `/tmp/`) to iterate over the files. The cleaning process should handle:
- **Extensions**: Convert `.mdx` to `.md` for native Obsidian compatibility.
- **Frontmatter conversion**: Convert existing frontmatter to match the Obsidian vault format:
  ```yaml
  ---
  tags: [tag1, tag2]
  category: CategoryName
  date: YYYY-MM-DD
  status: solved
  title: Title
  ---
  ```
- **React/JSX Component Stripping**: Remove MDX imports (e.g., `import X from '...'`) and React-like inline tags (e.g., `<Steps>`, `<CardGrid>`, `<Cta />`, `<Badge>`) which cause rendering issues in Obsidian.
- **Title Headers**: Standardize titles using H1 headers: `# 📚 Title`.

#### Clean & Convert Script Template
```python
import os
import re
import yaml
from datetime import datetime

src_dir = "/tmp/repo-docs/src/content/docs"
dest_dir = "/home/yan/obsidian/YanVault/docs/repo-name"
ignored_dirs = ["zh-cn", "ko", "ja", "_fragments"]

os.makedirs(dest_dir, exist_ok=True)

def process_file(src_path, dest_path):
    with open(src_path, 'r', encoding='utf-8') as f:
        content = f.read()

    frontmatter = {}
    body = content
    match = re.match(r'^---\r?\n(.*?)\r?\n---\r?\n(.*)', content, re.DOTALL)
    if match:
        fm_text = match.group(1)
        body = match.group(2)
        try:
            frontmatter = yaml.safe_load(fm_text) or {}
        except Exception as e:
            print(f"Error parsing frontmatter: {e}")

    # Build Obsidian Frontmatter
    tags = ["imported-docs", "tech-stack-name"]
    category = "Category/Name"
    title = frontmatter.get('title', "Fallback Title")

    new_fm = {
        'tags': tags,
        'category': category,
        'date': datetime.today().strftime('%Y-%m-%d'),
        'status': 'solved',
        'title': title
    }

    # Remove MDX imports and tags
    body = re.sub(r'^import\s+.*?\s+from\s+[\'"].*?[\'"];?\s*$', '', body, flags=re.MULTILINE)
    body = re.sub(r'</?Steps>', '', body, flags=re.IGNORECASE)
    body = re.sub(r'</?CardGrid.*?>', '', body, flags=re.IGNORECASE)
    body = re.sub(r'<Card\s+.*?>', '', body, flags=re.IGNORECASE)
    body = re.sub(r'</?Card>', '', body, flags=re.IGNORECASE)
    body = re.sub(r'<Badge\s+.*?>', '', body, flags=re.IGNORECASE)
    body = re.sub(r'</?Badge>', '', body, flags=re.IGNORECASE)

    # Format Output
    fm_block = "---\n" + yaml.dump(new_fm, default_flow_style=False) + "---\n\n"
    header_block = f"# 📚 {title}\n\n"
    final_content = fm_block + header_block + body.strip() + "\n\n#tech-stack #docs"

    os.makedirs(os.path.dirname(dest_path), exist_ok=True)
    with open(dest_path, 'w', encoding='utf-8') as f:
        f.write(final_content)

for root, dirs, files in os.walk(src_dir):
    dirs[:] = [d for d in dirs if d not in ignored_dirs]
    for file in files:
        if file.endswith(('.md', '.mdx')):
            src_path = os.path.join(root, file)
            rel_path = os.path.relpath(src_path, src_dir)
            dest_file = rel_path.replace('.mdx', '.md')
            dest_path = os.path.join(dest_dir, dest_file)
            process_file(src_path, dest_path)
```

## Handling Timeouts and Large Repositories

For massive repositories or platforms where direct `git clone` commands time out or fail (e.g. timeout limits inside tool execution environments):

1. **Avoid full `git clone`**: If the repo is large or network speed is slow, avoid cloning the entire repository.
2. **Download specific subdirectories/files**: Write a downloader script using the raw GitHub URL endpoint (`https://raw.githubusercontent.com/<owner>/<repo>/<branch>/<path>`) to fetch index listings or single pages sequentially.
3. **Use Local Toolchain Documentation**: If the tech stack is installed locally (like Rust's `rustup` HTML documentation), prioritize extracting and converting from local directories over downloading from remote servers. Convert the HTML documents using packages like `html2text` or `beautifulsoup4` into clean Markdown formats.
4. **Git Batching**: When committing thousands of generated Markdown files (e.g., Rust documentation containing 1,800+ pages), use standard `git add docs/` and commit them as a single clean batch to avoid repository history clutter.

## RAG Metadata Requirements

When importing external documents as a Knowledge Base source for RAG (Retrieval-Augmented Generation) systems, verify that every file contains standard metadata structure at the top:

```yaml
---
title: "Document Title"
source: "https://example.com/source-url"
category: "technology-category"
version: "v1.2.3"
updated: "YYYY-MM-DD"
---
```

Use a recursive verification script (e.g., Python parsing frontmatter with `pyyaml`) to scan all imported files, add missing fields automatically, and rewrite the headers uniformly before committing to git.

