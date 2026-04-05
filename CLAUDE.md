# CLAUDE.md

Hugo portfolio site using the hugo-profile theme. Single config file at `hugo.yaml`.

## Commands

```bash
hugo server              # Dev server at localhost:1313
hugo --gc --minify       # Production build
```

## Structure

- `hugo.yaml` - All site config (hero, about, experience, projects, certifications, contact)
- `content/blogs/` - Blog posts in Markdown
- `static/images/` - Site images
- `themes/hugo-profile/` - Theme (git submodule, don't edit directly)

## Notes

- Theme uses YAML config (not TOML)
- Homepage sections are configured in `hugo.yaml` under `params`, not as content files
- Blog posts go in `content/blogs/`, not `content/posts/`
- Deployed via GitHub Pages using `.github/workflows/hugo.yml`
