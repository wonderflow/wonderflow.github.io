# Blog Operation

## Install

```bash
sudo apt-get install hugo
git clone git@github.com:wonderflow/wonderflow.github.io.git
git submodule update --init --recursive
```

## New Post

```bash
hugo new posts/<YYYY>-<MM>-<DD>-<title>.md
```

### Post Front Matter Options

```toml
---
title: "Post Title"
date: YYYY-MM-DDTHH:MM:SS+08:00
description: "Post description (shows below title on single page)"
toc: true  # Enable Table of Contents
featured_image: "/images/folder/image.jpg"  # Show image in summary card
---
```

### Images

```bash
cp myimage.png ./static/images/<folder>/
```

Link it by `![description](/images/<folder>/myimage.jpg)`

## Preview

```bash
hugo server -D
```

Build and run locally: `http://localhost:1313`

## Build

```bash
hugo -e production
```

Output: `docs/` directory

## Release

```bash
git add .
git commit -m "message"
git push origin master
```

---

# Blog Styling & Customization

This blog uses the **Ananke theme** (gohugo-theme-ananke) with comprehensive custom styling optimized for technical blogs.

## Design Features (2024 Update)

### Auto Dark Mode
- Automatically detects system preference (`prefers-color-scheme`)
- GitHub-inspired dark color palette
- Zero configuration required - works out of the box

### Reading Experience
- **Reading Progress Bar**: Blue gradient bar at the top shows scroll progress
- **Back to Top Button**: Floating button appears after scrolling down
- **Smooth Scrolling**: CSS-based smooth scroll for anchor links
- **Sticky Table of Contents**: ToC stays visible while scrolling on desktop

### Typography
- Improved line height (1.8) for better readability
- H2 headings have subtle bottom borders for visual separation
- Better paragraph spacing (1.5rem)
- Monospace font stack for code: SF Mono, Fira Code, Monaco, Consolas

### Code Blocks
- GitHub-style syntax highlighting (Chroma)
- Rounded corners with subtle borders
- Inline code highlighted in pink/coral color
- Dark mode compatible

### UI Elements
- Card hover effects with lift animation
- Accent color links with underline on hover
- Improved blockquotes with blue left border
- Zebra-striped tables with hover highlight

## Design Overview

- **Theme**: Ananke (Tachyons CSS framework)
- **Color Scheme**: CSS Variables for easy theming
  - Light: Near-white background (`#fafafa`) + blue accents (`#007bff`)
  - Dark: GitHub dark (`#0d1117`) + blue accents (`#58a6ff`)
- **Typography**: Avenir (sans-serif) for body
- **Custom CSS**: `static/css/custom.css`
- **Syntax Highlighting**: Hugo Chroma (github style)

## CSS Variables Reference

All colors are defined as CSS variables in `:root` for easy customization:

```css
:root {
  --bg-primary: #fafafa;       /* Main background */
  --bg-secondary: #f5f5f5;     /* Cards, ToC */
  --bg-card: #ffffff;          /* Card background */
  --text-primary: #1a1a1a;     /* Headings */
  --text-secondary: #4a4a4a;   /* Body text */
  --text-muted: #666666;       /* Captions */
  --accent-color: #007bff;     /* Links, buttons */
  --accent-hover: #0056b3;     /* Hover state */
  --border-color: #e5e5e5;     /* Borders */
}
```

Dark mode overrides these automatically via `@media (prefers-color-scheme: dark)`.

## Changing Colors (Zero Maintenance)

Edit `config.toml`:

```toml
[params]
# Available Tachyons background colors:
# Grayscale: black, near-black, dark-gray, mid-gray, gray, silver,
#            light-silver, moon-gray, light-gray, near-white, white
# Reds: dark-red, red, light-red, orange, gold, yellow, light-yellow
# Greens: dark-green, green, light-green, washed-green
# Blues: navy, dark-blue, blue, light-blue, lightest-blue, washed-blue
background_color_class = "bg-near-white"

# Text color options: black, dark-gray, mid-gray, gray, light-silver
text_color = "dark-gray"
```

Available colors reference: https://tachyons.io/docs/themes/skins/

## Changing Typography (Zero Maintenance)

Edit `config.toml`:

```toml
[params]
# Body font: avenir, helvetica, georgia, garamond, baskerville
body_classes = "avenir"

# Post content font (default: serif)
# Use "f4 measure-wide" for better line length
post_content_classes = "f4 measure-wide"
```

## Chinese Font Configuration

The blog uses **Noto Sans SC** for Chinese text (loaded via Google Fonts).

### Available Chinese Fonts

| Font | Style | How to Enable |
|------|-------|---------------|
| Noto Sans SC | 无衬线，现代 | Default (already enabled) |
| LXGW WenKai (霞鹜文楷) | 楷体，优雅 | Uncomment in `custom.css` |

### Switch to LXGW WenKai

Edit `static/css/custom.css`, find the font section and uncomment:

```css
/* Uncomment this line: */
@import url('https://cdn.jsdelivr.net/npm/lxgw-wenkai-webfont@1.1.0/style.css');

/* And change font-family to: */
body, .avenir, .nested-copy-line-height, .lh-copy {
  font-family: 'LXGW WenKai', serif;
}
```

## Syntax Highlighting Configuration

Edit `config.toml`:

```toml
[markup.highlight]
  style = "github"       # Theme: github, monokai, dracula, etc.
  lineNos = false        # Set to true for line numbers
  guessSyntax = true     # Auto-detect language
  tabWidth = 4
```

Available styles: https://xyproto.github.io/splash/docs/all.html

Popular choices for technical blogs:
- `github` - Clean, light theme (default)
- `monokai` - Classic dark theme
- `dracula` - Modern dark theme
- `nord` - Subtle, muted colors

## Table of Contents

To enable ToC on a post, add to front matter:

```yaml
---
toc: true
---
```

ToC configuration in `config.toml`:

```toml
[markup.tableOfContents]
  startLevel = 2    # Start from H2
  endLevel = 4      # End at H4
  ordered = false   # Use bullets, not numbers
```

## Social Media Configuration

### Follow Icons (Footer/Header)

Edit `config.toml`:

```toml
[ananke.social.follow]
networks = ["x-twitter", "github"]

[ananke.social.networks.x-twitter]
username = "wonderflow1"

[ananke.social.networks.github]
username = "wonderflow"
```

### Share Buttons (On Posts)

Edit `config.toml`:

```toml
[ananke.social.share]
icons = true      # Show share icons
sharetext = true  # Show "Share:" label
networks = [
  "x-twitter",
  "bluesky",
  "linkedin",
  "reddit",
  "email"
]
```

Available networks: `x-twitter`, `bluesky`, `threads`, `linkedin`, `mastodon`, `reddit`, `hackernews`, `email`, `facebook`, `telegram`, `whatsapp`, `youtube`

## CSS Customizations

All custom styles are in `static/css/custom.css`.

### Key Sections

| Section | What it controls |
|---------|-----------------|
| CSS Variables | All theme colors (light & dark mode) |
| Dark Mode | Auto dark mode via `prefers-color-scheme` |
| Typography | Text alignment, line height, headings |
| Post Summary Cards | Hover effects, shadows, borders |
| Read More Button | Hover gradient |
| Archive List | Spacing, hover effects |
| Code Blocks | Syntax highlighting, inline code |
| Table of Contents | Sticky sidebar, navigation styling |
| Reading Progress | Top progress bar |
| Back to Top | Floating button |
| Accessibility | Focus styles, selection highlight |
| Print | Clean printing styles |

### Customizing Accent Color

Edit `static/css/custom.css` - change the CSS variable:

```css
:root {
  --accent-color: #007bff;  /* Change to your brand color */
  --accent-hover: #0056b3;  /* Darker shade for hover */
}

@media (prefers-color-scheme: dark) {
  :root {
    --accent-color: #58a6ff;  /* Lighter shade for dark mode */
    --accent-hover: #79c0ff;
  }
}
```

### Disabling Features

To disable reading progress bar or back-to-top button, edit `layouts/partials/site-footer.html`:
- Remove the `<div class="reading-progress">` element
- Remove the `<button class="back-to-top">` element
- Remove the corresponding `<script>` block

## Theme Submodule Updates

The Ananke theme is a git submodule. To update:

```bash
cd themes/ananke
git pull origin main
cd ../..
git add themes/ananke
git commit -m "Update Ananke theme"
```

## File Structure

```
wonderflow.github.io/
├── config.toml          # Site configuration
├── content/
│   ├── About.md
│   ├── archives.md
│   └── posts/           # Blog posts (137+ articles)
├── layouts/             # Custom layout overrides
│   ├── index.html       # Homepage
│   ├── _default/
│   │   ├── archive.html # Archive page
│   │   ├── page.html    # Static pages
│   │   ├── single.html  # Post template
│   │   └── _markup/
│   │       └── render-heading.html  # Anchor links for headings
│   └── partials/
│       ├── site-footer.html    # Footer + progress bar + back-to-top
│       ├── site-header.html    # Header + favicon
│       ├── site-navigation.html
│       └── summary-with-image.html  # Post cards
├── static/
│   ├── css/
│   │   └── custom.css   # All custom styles (600+ lines)
│   ├── favicon_io/      # Favicon set
│   └── images/          # Blog images
├── themes/
│   └── ananke/          # Ananke theme (submodule)
└── docs/                # Build output (git tracked)
```

## Maintenance Checklist (Easy)

- [ ] Update theme: `cd themes/ananke && git pull`
- [ ] Test: `hugo server -D`
- [ ] Build: `hugo -e production`
- [ ] Deploy: `git push origin master`

## Quick Color Palette Reference

### Light Mode
| Color | Hex | Usage |
|-------|-----|-------|
| Background | `#fafafa` | Page background |
| Card BG | `#ffffff` | Cards, ToC |
| Primary Blue | `#007bff` | Links, accents |
| Hover Blue | `#0056b3` | Link hover |
| Text Primary | `#1a1a1a` | Headings |
| Text Secondary | `#4a4a4a` | Body text |
| Border | `#e5e5e5` | Borders |

### Dark Mode
| Color | Hex | Usage |
|-------|-----|-------|
| Background | `#0d1117` | Page background |
| Card BG | `#1c2128` | Cards, ToC |
| Primary Blue | `#58a6ff` | Links, accents |
| Hover Blue | `#79c0ff` | Link hover |
| Text Primary | `#e6edf3` | Headings |
| Text Secondary | `#b1bac4` | Body text |
| Border | `#30363d` | Borders |

---

## Design Philosophy

This blog follows modern technical blog best practices:

1. **Readability First**: Optimal line height (1.8), measure width, and spacing
2. **Dark Mode**: Respects user system preference automatically
3. **Progressive Enhancement**: Core content works without JS
4. **Minimal Maintenance**: CSS-only solutions where possible
5. **Performance**: No external CSS frameworks loaded, minimal JS
6. **Accessibility**: Focus styles, ARIA labels, sufficient contrast

Inspired by: GitHub, dev.to, Hashnode, and other developer-focused platforms.
