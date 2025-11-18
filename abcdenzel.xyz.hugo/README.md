# Denzel's Blog

Hugo-based blog deployed to GitHub Pages at https://blog.abcdenzel.xyz

## Quick Start

### Local Development

```bash
# Start Hugo development server
hugo serve

# Start with drafts visible
hugo serve -D

# Build for production
hugo
```

### Deployment

The site automatically deploys to GitHub Pages when you push to the `master` branch.

**First-time setup:**

1. Push this repository to GitHub
2. In GitHub repository settings:
   - Go to **Settings > Pages**
   - Under "Build and deployment", set **Source** to **GitHub Actions**
3. Configure custom domain:
   - In **Settings > Pages > Custom domain**, enter `blog.abcdenzel.xyz`
   - Click **Save** (this creates a CNAME file in the repo)
4. In your DNS provider (e.g., Cloudflare), add DNS records:
   ```
   Type: CNAME
   Name: blog
   Target: yourusername.github.io
   ```
5. Wait for DNS propagation (can take up to 24 hours, usually ~10 minutes)
6. In **Settings > Pages**, check "Enforce HTTPS" once DNS is working

## Repository Structure

```
.
├── .github/
│   └── workflows/
│       └── deploy.yml          # GitHub Actions deployment
├── archetypes/                 # Content templates
├── content/
│   ├── posts/
│   │   ├── meta/               # Meta posts (blogging, philosophy, etc.)
│   │   └── tech/               # Technical posts
│   ├── about.md                # About page
│   └── author.md               # Author bio
├── themes/
│   ├── diary/                  # Current theme (hugo-theme-diary)
│   └── nightfall/              # Alternative theme (git submodule)
├── hugo.toml                   # Hugo configuration
├── CLAUDE.md                   # Instructions for Claude Code
└── README.md                   # This file
```

## Configuration

Main configuration in `hugo.toml`:

- **baseURL**: `https://blog.abcdenzel.xyz/`
- **Theme**: `diary` (hugo-theme-diary)
- **Hugo version**: 0.152.2+

### Before Publishing

1. Replace `[your-email@example.com]` in:
   - `content/about.md`
   - `content/author.md`

2. Optionally configure analytics in `hugo.toml` (currently disabled)

3. Review and un-draft posts in `content/posts/`:
   - Change `draft = true` to `draft = false` in frontmatter

## Writing Posts

Create a new post:

```bash
hugo new content/posts/tech/my-post-title.md
```

Post frontmatter example:

```toml
+++
date = '2025-11-16T10:30:00-04:00'
draft = false
title = 'My Post Title'
tags = ['tag1', 'tag2']
categories = ['tech']
+++
```

## Themes

Two themes available:

1. **diary** (current): Mobile-friendly journal theme with dark mode
2. **nightfall**: Minimal dark theme (git submodule at `themes/nightfall`)

To switch themes, edit `hugo.toml`:

```toml
theme = "nightfall"  # or "diary"
```

Update the Nightfall theme:

```bash
git submodule update --remote --merge
```

## Troubleshooting

### GitHub Actions fails to build

- Check `.github/workflows/deploy.yml` for syntax errors
- Verify Hugo version matches in `env.HUGO_VERSION`
- Check submodules are properly initialized

### Custom domain not working

- Verify CNAME record in DNS points to `yourusername.github.io`
- Wait for DNS propagation (use `dig blog.abcdenzel.xyz` to check)
- Ensure "Enforce HTTPS" is checked in GitHub Pages settings
- Check that CNAME file exists in repo root (created by GitHub)

### Build works locally but not on GitHub

- Check that all files are committed (especially themes/)
- Verify submodules are properly referenced in `.gitmodules`
- Check for absolute paths that won't work in different environments

## Resources

- [Hugo Documentation](https://gohugo.io/documentation/)
- [GitHub Pages Docs](https://docs.github.com/en/pages)
- [hugo-theme-diary](https://github.com/AmazingRise/hugo-diary)
- [Nightfall Theme](https://github.com/LordMathis/hugo-theme-nightfall)
