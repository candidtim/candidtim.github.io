# AGENTS.md

## Project

Personal Jekyll blog hosted on GitHub Pages (`candidtim.github.io`).

## Structure

- `_posts/` — blog posts (Markdown, filename format: `YYYY-MM-DD-slug.md`)
- `_layouts/` — Jekyll layouts (`default.html`, `post.html`, `page.html`)
- `_includes/` — partials (`head.html`, `header.html`, `footer.html`)
- `_sass/` — SCSS source files
- `css/main.scss` — entry point that imports `_sass/` files
- `_site/` — generated output (gitignored, do not edit)
- `.jekyll-cache/` — Jekyll cache (safe to delete)
- `.devcontainer/` — dev container configuration for VS Code
- `Gemfile` — Ruby dependencies

## Local Development

Open the workspace in a dev container (`.devcontainer/devcontainer.json`), then run:

```
jekyll serve -H 0.0.0.0
```

Site serves at `http://localhost:4000`. Jekyll watches for file changes and rebuilds automatically. Port 4000 is forwarded automatically.

## Gotchas

- No build, lint, test, or typecheck steps — it's a plain Jekyll blog.
- Posts use `kramdown` for Markdown and `rouge` for syntax highlighting.
- `_site/` is generated; never commit or edit files there.

## Troubleshooting: Post Not Showing on `jekyll serve`

`jekyll serve` reads directly from the filesystem — no git commit or staging is required. If a post isn't visible, check:

1. **Future dates** — Posts with a date in the future are ignored by default.
2. **Incorrect filename format** — Filenames must follow `YYYY-MM-DD-title.md`.
3. **Post not in the right directory** — Posts must be inside `_posts/`.
4. **Front matter issues** — Every post needs properly formatted YAML front matter between `---` markers. Missing, malformed, or empty front matter will cause the post to be skipped.
5. **Date mismatch** — If the date in the front matter conflicts with the filename date, behavior can be unpredictable.
6. **Stale cache** — Run `jekyll clean && jekyll serve --no-incremental` to force a fresh build.
