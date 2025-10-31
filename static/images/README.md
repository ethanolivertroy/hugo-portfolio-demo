# Image Assets for Hugo Cybersecurity Portfolio

This directory contains all images used in the portfolio site.

## Directory Structure

```
static/images/
├── hero.svg                 # Hero section animated illustration
├── profile.png              # Your professional headshot
├── fav.png                  # Favicon (32x32 or 64x64)
├── resume.pdf               # Your resume (not an image, but stored here)
├── blogs/                   # Blog post featured images
├── projects/                # Project screenshots/images
├── achievements/            # Certification and achievement badges
├── gallery/                 # Personal gallery photos
│   ├── travel/             # Travel photography
│   ├── tech/               # Tech/conference photos
│   └── personal/           # Personal hobby photos
└── profiles/               # Additional profile images

```

## Image Guidelines

### Required Images

**1. Hero Image (`hero.svg` or `hero.png`)**
- Dimensions: 800x600px recommended
- Format: SVG preferred (scalable), or PNG
- Purpose: Main hero section background/illustration
- Tip: Use illustrations from [undraw.co](https://undraw.co/) or [humaaans.com](https://www.humaaans.com/)

**2. Profile Photo (`profile.png`)**
- Dimensions: 400x400px (square)
- Format: PNG or JPG
- Purpose: About section and avatar
- Tip: Professional headshot, neutral background

**3. Favicon (`fav.png`)**
- Dimensions: 64x64px or 32x32px
- Format: PNG (or ICO)
- Purpose: Browser tab icon
- Tip: Simple, recognizable symbol or initials

### Blog Post Images

**Location:** `blogs/`

**Recommended:**
- Dimensions: 1200x630px (good for social media sharing)
- Format: JPG or PNG
- Naming: Match blog post slug (e.g., `sql-injection-vulnerability-analysis.png`)

**Examples needed:**
- `sql-injection.png` - Database/security themed
- `htb-writer.png` - Hack The Box logo or machine screenshot
- `soc2.png` - Compliance/audit themed
- `python-security.png` - Python logo with security symbols
- `career-path.png` - Career growth illustration
- `travel-security.png` - Globe/travel with security
- `homelab.png` - Server rack or network diagram
- `threat-hunting.png` - Hunting/detection themed
- `aws-security.png` - AWS logo with security
- `password-managers.png` - Password/lock themed

**Where to find images:**
- [Unsplash](https://unsplash.com/) - Free high-quality photos
- [Pexels](https://www.pexels.com/) - Free stock photos
- [Icons8 Illustrations](https://icons8.com/illustrations) - Themed illustrations
- Create custom: Use Canva or Figma

### Project Images

**Location:** `projects/`

**Recommended:**
- Dimensions: 800x600px or 1200x900px
- Format: PNG (for screenshots) or JPG (for photos)
- Purpose: Project screenshots, architecture diagrams, or demo images

**Projects needing images:**
1. `risk-framework.png` - Risk assessment dashboard screenshot
2. `policy-mgmt.png` - Policy management interface
3. `siem-automation.png` - SIEM dashboard or automation workflow
4. `security-pipeline.png` - CI/CD pipeline diagram
5. `webapp-scanner.png` - Scanner output or tool interface
6. `ad-framework.png` - Active Directory attack diagram
7. `ctf-writeups.png` - CTF challenge screenshot
8. `threat-hunting.png` - Threat hunting dashboard
9. `ir-automation.png` - Incident response workflow
10. `homelab.png` - Homelab network diagram
11. `portfolio.png` - Screenshot of this site!

**Tips:**
- Screenshots: Use browser devtools to set viewport before screenshot
- Diagrams: Use [draw.io](https://draw.io/), [Excalidraw](https://excalidraw.com/), or [Lucidchart](https://www.lucidchart.com/)
- Mock dashboards: Use Grafana, Splunk screenshots, or create in Figma

### Achievement Images

**Location:** `achievements/`

**Recommended:**
- Dimensions: 400x400px (square) or 600x400px
- Format: PNG (with transparency if possible)
- Purpose: Certification badges, award graphics

**Examples needed:**
1. `defcon.png` - DEF CON logo/badge
2. `cve.png` - CVE logo or custom graphic
3. `bsides.png` - BSides conference logo
4. `h1.png` - HackerOne logo
5. Custom graphics for other achievements

**Where to get:**
- Official certification badges from issuing organizations
- Conference logos (check usage rights)
- Create custom badges in Canva or Photoshop

### Gallery Images

**Location:** `gallery/`

#### Travel Photos (`gallery/travel/`)
**Examples needed:**
- `lisbon-sunset.jpg`
- `swiss-alps.jpg`
- `amsterdam.jpg`
- `tokyo-night.jpg`
- `bali-rice.jpg`
- `bangkok-temple.jpg`
- `machu-picchu.jpg`
- `patagonia.jpg`

**Specs:**
- Dimensions: 1920x1080px or 2400x1600px
- Format: JPG (compressed for web)
- Tip: Use your own travel photos!

#### Tech/Conference (`gallery/tech/`)
**Examples needed:**
- `homelab-setup.jpg` - Your actual homelab
- `desk-setup.jpg` - Your workspace
- `server-rack.jpg` - Server equipment
- `bsides-talk.jpg` - Conference speaking photo
- `defcon.jpg` - Conference photos
- `ctf-team.jpg` - Team photos

#### Personal (`gallery/personal/`)
**Examples needed:**
- `camera-gear.jpg` - Photography equipment
- `cat-keyboard.jpg` - Pet photos
- `dog-hiking.jpg` - Pet/outdoor photos
- `climbing.jpg` - Hobby photos
- `scuba.jpg` - Adventure sports
- `motorcycle.jpg` - Personal interests

## Quick Setup Options

### Option 1: Use Placeholder Images

Generate placeholder images at [placeholder.com](https://placeholder.com/):
```bash
# Download placeholders
wget https://via.placeholder.com/1200x630/4A90E2/FFFFFF?text=Blog+Post -O blogs/placeholder.jpg
wget https://via.placeholder.com/400x400/2ECC71/FFFFFF?text=Profile -O profile.png
wget https://via.placeholder.com/800x600/E74C3C/FFFFFF?text=Project -O projects/placeholder.png
```

### Option 2: Use Unsplash

Search Unsplash for relevant images:
```
Security: https://unsplash.com/s/photos/cybersecurity
Coding: https://unsplash.com/s/photos/programming
Travel: https://unsplash.com/s/photos/travel
```

### Option 3: Create Custom Graphics

Tools for creating custom images:
- **Canva** (easiest, templates available)
- **Figma** (professional, free tier)
- **GIMP** (free Photoshop alternative)
- **Inkscape** (free vector graphics)

## Image Optimization

Before adding images to the site, optimize them:

```bash
# Install ImageMagick and jpegoptim
sudo apt-get install imagemagick jpegoptim optipng

# Resize and optimize JPG
convert input.jpg -resize 1200x630^ -gravity center -extent 1200x630 -quality 85 output.jpg

# Optimize PNG
optipng -o5 image.png

# Batch process all images in directory
for img in *.jpg; do
  jpegoptim --size=200k "$img"
done
```

## Copyright & Attribution

**Important:**
- Only use images you have rights to use
- If using stock photos, check license (most Unsplash/Pexels are free)
- Attribute photographers if required
- Never use copyrighted logos without permission
- Create your own screenshots when possible

## Current Status

✅ Directory structure created
⚠️ **TODO: Add actual images before deployment**

Replace all placeholder references in `hugo.yaml` and blog posts with actual image paths once you add real images.

## Need Help?

Check the hugo-profile documentation:
- [Theme Demo](https://hugo-profile.netlify.app)
- [GitHub Repository](https://github.com/gurusabarish/hugo-profile)
- [Image optimization guide](https://web.dev/fast/#optimize-your-images)
