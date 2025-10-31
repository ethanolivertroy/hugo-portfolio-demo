# Quick Start Guide

Get your cybersecurity portfolio live in under 30 minutes!

## Prerequisites

- Git installed
- GitHub account
- Hugo installed (or you'll install it in Step 2)

## Step-by-Step Setup

### 1. Fork & Clone

```bash
# Fork this repo on GitHub, then clone your fork
git clone https://github.com/YOUR-USERNAME/hugo-cybersecurity-portfolio
cd hugo-cybersecurity-portfolio
git submodule update --init --recursive
```

### 2. Install Hugo

**macOS:**
```bash
brew install hugo
```

**Linux:**
```bash
sudo snap install hugo
```

**Windows:**
```bash
choco install hugo-extended
```

**Verify installation:**
```bash
hugo version  # Should be v0.120.0 or higher
```

### 3. Test Locally

```bash
hugo server -D
```

Open http://localhost:1313 - you should see the template!

### 4. Customize

Edit `hugo.yaml`:
```yaml
baseURL: "https://your-name.netlify.app/"  # Line 1
title: "Your Name - Cybersecurity Professional"  # Line 3

hero:
  title: "Your Name"  # Line ~87
  subtitle: "Security Engineer"  # Your title
```

### 5. Deploy

**To Netlify (easiest):**
1. Push to GitHub: `git push`
2. Go to [netlify.com](https://netlify.com)
3. Click "New site from Git"
4. Select your repo
5. Deploy!

**To Vercel:**
```bash
npm i -g vercel
vercel
```

## What to Customize First

### Priority 1 (Do today):
- [ ] Change hero title to your name
- [ ] Update about section with your bio
- [ ] Add your profile photo to `static/images/profile.png`
- [ ] Change social links (LinkedIn, GitHub, Twitter)

### Priority 2 (This week):
- [ ] Replace example blog posts
- [ ] Update projects with your work
- [ ] Add your resume PDF
- [ ] Customize colors/theme

### Priority 3 (Next week):
- [ ] Add more blog posts
- [ ] Fill gallery with your photos
- [ ] Set up custom domain
- [ ] Enable analytics

## Common Issues

**Hugo server won't start:**
- Make sure you're in the project directory
- Run `git submodule update --init --recursive`

**Site looks broken:**
- Check Hugo version (`hugo version`)
- Clear browser cache
- Check `hugo.yaml` syntax

**Images not showing:**
- Images must be in `static/images/`
- Use absolute paths: `/images/profile.png`

## Next Steps

- Read [CUSTOMIZATION.md](CUSTOMIZATION.md) for detailed config
- See [DEPLOYMENT.md](DEPLOYMENT.md) for custom domains
- Check [CONTENT.md](CONTENT.md) for adding content

## Need Help?

Open an [issue](https://github.com/yourusername/hugo-cybersecurity-portfolio/issues) - we're here to help!
