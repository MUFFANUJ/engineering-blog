# Engineering Blog

Source repository for engineering blog posts published on [openteams.com/blog](https://openteams.com/blog) under the **Engineering** category.

## Overview

This repo contains the source files for OpenTeams engineering articles. Posts are authored in Markdown (`.md`) or Quarto Markdown (`.qmd`) and published to the OpenTeams WordPress blog.

## Repository Structure

```text
engineering-blog/
├── posts/                          # Blog post directories
│   └── <post-slug>/
│       ├── index.md                # or index.qmd
│       └── images/                 # Post-specific images
├── scripts/
│   └── wordpress/
│       ├── publish.py              # Unified create/sync script
│       └── wordpress_utils.py      # Shared WordPress utilities
├── .env.example                    # WordPress credentials template
├── pyproject.toml                  # Python dependencies
└── README.md
```

Each post lives in its own directory under `posts/`, named with a URL-friendly slug (e.g., `posts/building-ml-pipelines/`).

## Setup

1. Copy `.env.example` to `.env` and fill in your WordPress credentials:
   ```
   WP_TOKEN=your-application-password
   WP_API_URL=https://openteams.com/wp-json/wp/v2
   USERNAME=your-wordpress-username
   ```

2. Install dependencies:
   ```bash
   uv sync
   ```

## Writing a Post

1. Create a new directory under `posts/` with a descriptive slug.
2. Add an `index.md` or `index.qmd` file with YAML frontmatter:

   ```yaml
   ---
   title: "Your Post Title"
   slug: your-post-slug
   author: wordpress-username
   categories:
     - Engineering
   tags:
     - python
     - data-engineering
   meta_description: "A short summary for SEO (150-160 chars)."
   focus_keyword: "main keyword"
   seo_keywords:
     - "keyword one"
     - "keyword two"
   ---
   ```

   **Required fields:** `title`, `slug`, `author`, `categories`

   `author` is the WordPress username of the post author. The post will be published under their name. The author must have an account on the OpenTeams WordPress site.

   **Optional fields:** `tags`, `meta_description`, `focus_keyword`, `seo_keywords`

   **Auto-added after publishing:** `wordpress_id`, `wordpress_url`, `last_synced`

3. Place any images in an `images/` subdirectory and reference them with relative paths:
   ```markdown
   ![diagram](images/architecture.png)
   ```

## File Formats

| Format | Extension | When to Use |
|--------|-----------|-------------|
| Markdown | `.md` | Standard prose, code snippets, conceptual articles |
| Quarto Markdown | `.qmd` | Posts with executable code, data visualizations, or reproducible analysis |

## Markdown Syntax Reference

The publish script converts markdown to HTML with support for Prism.js plugins. Use `#|` directives inside code blocks to control rendering.

### Standard Code Block

````markdown
```python
def hello():
    print("Hello, world!")
```
````

### Line Highlighting

Highlight specific lines to draw attention. Uses the [Prism.js Line Highlight](https://prismjs.com/plugins/line-highlight/) plugin.

````markdown
```python
#| highlight: 2-3, 5
import pandas as pd

df = pd.read_csv("data.csv")
df = df.dropna()
result = df.groupby("category").sum()
```
````

`#| highlight:` accepts single lines (`5`), ranges (`1-3`), and combinations (`1-3, 5, 9-12`).

### Command-Line Prompt

Show terminal prompts with the [Prism.js Command Line](https://prismjs.com/plugins/command-line/) plugin. Lines are rendered with `$` (or `#` for root) prompts.

````markdown
```bash
#| command-line
pip install pandas
python main.py
```
````

Customize the prompt with optional attributes:

````markdown
```bash
#| command-line data-user=root data-host=server
apt-get update
```
````

Use a fully custom prompt:

````markdown
```sql
#| command-line data-prompt="mysql>"
SELECT * FROM users;
```
````

### Command-Line with Output

Mark lines that are output (no prompt shown) with `data-output`:

````markdown
```bash
#| command-line
#| data-output: 2-4
ls -la
total 4
drwxr-xr-x 2 user user 4096 Jan 1 00:00 .
-rw-r--r-- 1 user user    0 Jan 1 00:00 file.txt
```
````

Or use `data-filter-output` to mark output lines with a prefix (stripped on render):

````markdown
```bash
#| command-line
#| data-filter-output: (out)
echo "hello"
(out)hello
```
````

### Mermaid Diagrams

Rendered automatically with dark theme:

````markdown
```mermaid
graph LR
    A[Start] --> B[End]
```
````

### Other Supported Elements

| Element | Syntax |
|---------|--------|
| Bold | `**text**` |
| Italic | `*text*` |
| Inline code | `` `code` `` |
| Link | `[text](url)` |
| Image | `![alt](images/file.png)` |
| Blockquote | `> text` |
| Table | Standard markdown table syntax |
| Horizontal rule | `---` |
| Ordered list | `1. item` |
| Unordered list | `- item` |
| Nested list | Indent with 2 or 4 spaces |

## Publishing

A single script handles both creating new posts and syncing updates:

```bash
uv run scripts/wordpress/publish.py posts/<slug>/index.md
```

- **No `wordpress_id` in frontmatter** → creates a new WordPress draft
- **Has `wordpress_id`** → syncs updates to the existing post

Both paths automatically:
- Upload local images to WordPress media library
- Convert markdown to HTML (with code highlighting, mermaid diagrams, tables)
- Resolve categories and tags against WordPress
- Set Yoast SEO metadata
- Update the file's frontmatter with `wordpress_id`, `wordpress_url`, and `last_synced`

### Automatic Publishing (CI)

A GitHub Actions workflow automatically publishes articles when changes to `posts/` are pushed to `main`. Contributors do not need WordPress credentials.

**How it works:**
1. Write your post on a feature branch.
2. Open a pull request for review.
3. Once approved and merged to `main`, the workflow automatically:
   - Detects which articles were added or modified
   - Publishes new articles as WordPress drafts (or syncs updates)
   - Commits updated frontmatter (`wordpress_id`, `wordpress_url`, `last_synced`) back to `main`

### Manual Publishing (Local)

You can also publish locally if you have WordPress credentials in `.env`:

```bash
uv run scripts/wordpress/publish.py posts/<slug>/index.md
```

### Repository Secrets

The following secrets must be configured in the GitHub repo settings (`Settings > Secrets and variables > Actions`):

| Secret | Description |
|--------|-------------|
| `WP_TOKEN` | WordPress application password |
| `WP_API_URL` | WordPress REST API URL (e.g., `https://openteams.com/wp-json/wp/v2`) |
| `WP_USERNAME` | WordPress username |

## Contributing

1. Create a branch from `main`.
2. Add your post following the structure above.
3. Submit a PR with a clear title and summary.
4. Address review feedback, then merge — publishing happens automatically.
