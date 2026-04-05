# Hugo Portfolio Demo for Cybersecurity Professionals

A ready-to-deploy portfolio template built with [Hugo](https://gohugo.io/) and the [hugo-profile](https://github.com/gurusabarish/hugo-profile) theme. Designed for cybersecurity professionals who want a professional online presence without web development experience.

**[Live Demo](https://ethanolivertroy.github.io/hugo-portfolio-demo/)**

## Features

- Pre-configured sections for experience, projects, certifications, education, and blog
- Light/dark mode toggle
- Full-text search
- Mobile-responsive design
- Blog with tags and social sharing
- Contact form integration (FormSpree)
- Automatic deployment via GitHub Pages

## Quick Start

### Option A: Use This Template (Recommended)

1. Click **"Use this template"** > **"Create a new repository"** at the top of this page
2. Name your repo (e.g., `my-portfolio` or `username.github.io`)
3. In your new repo, go to **Settings** > **Pages** > set Source to **GitHub Actions**
4. Edit `hugo.yaml` with your information (name, bio, experience, projects, social links)
5. Push your changes. The site deploys automatically.

### Option B: Fork and Clone

```bash
# Fork this repo first, then:
git clone --recursive https://github.com/YOUR_USERNAME/hugo-portfolio-demo.git
cd hugo-portfolio-demo
hugo server
```

Visit `http://localhost:1313` to preview.

## Customization

Everything lives in `hugo.yaml`. Open it and replace the placeholder content:

| Section | What to Change |
|---|---|
| `hero` | Your name, tagline, and social links |
| `about` | Bio and skills list |
| `experience` | Work history with company names and descriptions |
| `education` | Degrees, schools, dates |
| `achievements` | Certifications (OSCP, CISSP, AWS, etc.) |
| `projects` | Your tools, repos, and writeups |
| `contact` | Email address or FormSpree form ID |
| `footer.socialNetworks` | GitHub, LinkedIn, Twitter URLs |

### Images

Replace the placeholder images in `static/images/`:

- `me.png` - Your profile photo (used in About section)
- `hero.svg` - Hero section graphic
- `projects/` - Project screenshots or thumbnails

### Blog Posts

Create new posts in `content/blogs/`:

```bash
hugo new content content/blogs/my-post-title.md
```

## Local Development

Requirements: [Hugo](https://gohugo.io/installation/) (v0.87.0+)

```bash
# Start dev server with live reload
hugo server

# Build for production
hugo --gc --minify
```

## Deployment Options

### GitHub Pages (default)

Already configured. Push to `main` and the GitHub Actions workflow handles the rest.

### Netlify

[![Deploy to Netlify](https://www.netlify.com/img/deploy/button.svg)](https://app.netlify.com/start/deploy?repository=https://github.com/ethanolivertroy/hugo-portfolio-demo)

### Vercel

Import the repo at [vercel.com/new](https://vercel.com/new). Set the framework to Hugo and it auto-detects the config.

## Project Structure

```
.
├── .github/workflows/hugo.yml   # GitHub Pages deployment
├── content/blogs/               # Blog posts (Markdown)
├── static/images/               # Images and assets
├── themes/hugo-profile/         # Theme (git submodule)
├── hugo.yaml                    # All site configuration
└── README.md
```

## Credits

- [hugo-profile](https://github.com/gurusabarish/hugo-profile) theme by Gurusabarish
- [Hugo](https://gohugo.io/) static site generator
- [Bootstrap](https://getbootstrap.com/) and [Font Awesome](https://fontawesome.com/)

## License

MIT
