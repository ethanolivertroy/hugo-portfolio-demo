# Cybersecurity Professional Portfolio Template

A comprehensive, ready-to-fork portfolio template designed specifically for cybersecurity professionals. Built with [Hugo](https://gohugo.io/) and the [hugo-profile](https://github.com/gurusabarish/hugo-profile) theme.

[![Deploy to Netlify](https://www.netlify.com/img/deploy/button.svg)](https://app.netlify.com/start/deploy?repository=https://github.com/yourusername/hugo-cybersecurity-portfolio)

## ✨ Features

- 🎯 **Career Path Coverage**: Example content for GRC, Security Engineering, Offensive Security, and SOC/Defensive roles
- 📝 **10 Example Blog Posts**: Covering technical writeups, CTF solutions, career advice, and personal interests
- 🎨 **Modern, Responsive Design**: Clean, professional appearance on all devices
- 🚀 **One-Click Deploy**: Deploy to Netlify in seconds
- 🌙 **Dark Mode**: Built-in dark mode (perfect for security professionals)
- 📊 **Portfolio Sections**: Projects, Experience, Education, Achievements, Skills
- 🖼️ **Gallery**: Showcase personal interests beyond security
- 🔍 **SEO Optimized**: Meta tags, structured data, sitemap generation
- 📱 **Mobile-First**: Optimized for mobile viewing
- 💬 **Social Integration**: LinkedIn, GitHub, Twitter, blog links
- 🎓 **Beginner-Friendly**: Extensive documentation and comments

## 🎯 Who Is This For?

This template is perfect for:

- 🎓 **Students** entering cybersecurity
- 🔄 **Career changers** transitioning into security
- 👔 **Mid-career professionals** building their personal brand
- 🎤 **Conference speakers** and security researchers
- 💼 **Consultants** showcasing their work
- 🌍 **Digital nomads** in security

## 🚀 Quick Start

### Option 1: Fork and Clone (Recommended)

```bash
# 1. Fork this repository on GitHub

# 2. Clone your fork
git clone https://github.com/YOUR-USERNAME/hugo-cybersecurity-portfolio
cd hugo-cybersecurity-portfolio

# 3. Initialize submodules (for the theme)
git submodule update --init --recursive

# 4. Install Hugo (if not already installed)
# macOS:
brew install hugo

# Linux:
sudo snap install hugo

# Windows:
choco install hugo-extended

# 5. Run local development server
hugo server -D

# 6. Open http://localhost:1313 in your browser
```

### Option 2: One-Click Deploy to Netlify

1. Click the "Deploy to Netlify" button above
2. Connect your GitHub account
3. Netlify will fork the repo and deploy automatically
4. Start customizing!

## 📝 Customization Guide

### Step 1: Update Site Configuration

Edit `hugo.yaml`:

```yaml
baseURL: "https://your-domain.com/"  # Your custom domain or Netlify URL
title: "Your Name - Cybersecurity Professional"
```

### Step 2: Personalize Your Profile

Update these sections in `hugo.yaml`:

**Hero Section**:
```yaml
hero:
  title: "Your Name"  # ← Change this
  subtitle: "Cybersecurity Professional"  # ← Your title
  content: "Your elevator pitch here"  # ← Your story
```

**About Section**:
```yaml
about:
  content: |-
    Replace this with your own bio!
    Talk about your journey, expertise, and interests.
```

### Step 3: Add Your Content

```bash
# Replace example blog posts
rm content/blogs/*.md

# Add your own
cp ~/my-blog-post.md content/blogs/

# Add your images
cp ~/headshot.jpg static/images/profile.png
cp ~/project-screenshot.png static/images/projects/
```

## 📚 Documentation

For detailed guides:

- **[Customization Guide](docs/CUSTOMIZATION.md)** - Detailed configuration options
- **[Content Guide](docs/CONTENT.md)** - How to add blog posts, projects, achievements
- **[Deployment Guide](docs/DEPLOYMENT.md)** - Deploy to Netlify, Vercel, GitHub Pages
- **[Troubleshooting](docs/TROUBLESHOOTING.md)** - Common issues and solutions
- **[Career Path Templates](docs/CAREER_PATHS.md)** - Specific guidance for each security domain

## 🎨 What's Included

### Blog Posts (10 Examples)

1. SQL Injection Analysis (Offensive Security)
2. HTB Machine Writeup (CTF)
3. SOC 2 Audit Guide (GRC/Compliance)
4. Security Automation with Python (Engineering)
5. Breaking into Cybersecurity (Career)
6. Digital Nomad Security (Personal)
7. Homelab Setup Guide (Technical)
8. Threat Hunting Guide (SOC/Defensive)
9. AWS Security Best Practices (Cloud)
10. Password Manager Comparison (Tools)

### Projects (11 Examples)

- GRC: Risk assessment framework, policy management
- Security Engineering: SIEM automation, security pipeline
- Offensive: Web app scanner, AD attack framework, CTF writeups
- SOC/Defensive: Threat hunting, IR automation
- Personal: Homelab, portfolio site

## 🛠️ Technology Stack

- **Static Site Generator**: Hugo v0.148.0+
- **Theme**: hugo-profile
- **Hosting**: Netlify (recommended), Vercel, GitHub Pages
- **Content**: Markdown
- **Styling**: Bootstrap 5

## 📦 Project Structure

```
hugo-cybersecurity-portfolio/
├── content/
│   ├── blogs/              # Blog posts (10 examples)
│   └── gallery.md          # Personal photo gallery
├── static/
│   └── images/             # All images
├── docs/                   # Detailed documentation
├── hugo.yaml               # Main configuration
├── netlify.toml            # Netlify build config
├── README.md               # This file
├── LICENSE                 # MIT License
└── CONTRIBUTING.md         # Contribution guidelines
```

## 🚀 Deployment

### Netlify (Recommended)

```bash
git push  # Push to GitHub
# Then connect repo in Netlify dashboard
```

### Vercel

```bash
vercel  # Run from project directory
```

See [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md) for detailed instructions.

## 🤝 Contributing

Contributions welcome! See [CONTRIBUTING.md](CONTRIBUTING.md).

## 📄 License

MIT License - see [LICENSE](LICENSE) file.

## 💬 Support

- 🐛 **Issues**: [GitHub Issues](https://github.com/yourusername/hugo-cybersecurity-portfolio/issues)
- 💬 **Discussions**: [GitHub Discussions](https://github.com/yourusername/hugo-cybersecurity-portfolio/discussions)

## ⭐ Show Your Support

If this template helped you:
- ⭐ Star this repository
- 🍴 Fork and customize
- 📢 Share with others

## 🤖 Built with AI

This entire portfolio template was created with the assistance of [Claude](https://claude.ai) (Anthropic's AI assistant). From the initial concept to the final implementation—including:
- 10 comprehensive blog posts covering all security career paths
- 11 example projects with detailed descriptions
- Complete portfolio configuration
- Documentation and guides
- Image directory structure

Claude helped transform a basic Hugo site into a production-ready cybersecurity professional portfolio template in a single session.

**Want to build something similar?** Claude Code can help you create, customize, and deploy your own projects!

---

<p align="center">
  Made with ❤️ for the cybersecurity community<br>
  Built with 🤖 <a href="https://claude.ai">Claude</a>
</p>

<p align="center">
  <a href="#-features">Features</a> •
  <a href="#-quick-start">Quick Start</a> •
  <a href="#-customization-guide">Customize</a> •
  <a href="#-deployment">Deploy</a>
</p>
