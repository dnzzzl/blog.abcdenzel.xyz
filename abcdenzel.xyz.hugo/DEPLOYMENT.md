# Deployment Guide

## Prerequisites

1. GitHub repository created and code pushed
2. Domain `abcdenzel.xyz` registered and accessible
3. Access to DNS management (Cloudflare, etc.)

## Step-by-Step Deployment

### 1. Configure GitHub Repository

```bash
# Initialize git if not already done
git init
git add .
git commit -m "Initial commit: Hugo blog setup"

# Add remote and push (replace with your GitHub username)
git remote add origin git@github.com:yourusername/blog.git
git push -u origin master
```

### 2. Enable GitHub Pages

1. Go to your repository on GitHub
2. Navigate to **Settings** > **Pages**
3. Under "Build and deployment":
   - **Source**: Select **GitHub Actions**
   - (This allows the `.github/workflows/deploy.yml` to handle deployment)

### 3. Configure Custom Domain in GitHub

1. Still in **Settings** > **Pages**
2. Under "Custom domain":
   - Enter: `blog.abcdenzel.xyz`
   - Click **Save**
3. GitHub will create a `CNAME` file in your repository
4. Pull the CNAME file to your local repo:
   ```bash
   git pull origin master
   ```

### 4. Configure DNS Records

In your DNS provider (Cloudflare, Namecheap, etc.), add the following record:

**For CNAME (Recommended):**
```
Type:  CNAME
Name:  blog
Target: yourusername.github.io.
TTL:   Auto or 3600
```

**Alternative: A Records (if CNAME isn't available):**
```
Type:  A
Name:  blog
Target: 185.199.108.153
TTL:   3600

# Add all four GitHub Pages IPs:
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```

### 5. Wait for DNS Propagation

DNS changes can take up to 24 hours to propagate, but usually complete in 5-30 minutes.

Check DNS propagation:
```bash
# Check if DNS is propagating
dig blog.abcdenzel.xyz

# Or use online tools
# https://dnschecker.org
```

### 6. Enable HTTPS

1. Once DNS is working, return to **Settings** > **Pages** on GitHub
2. Check the box for **Enforce HTTPS**
3. GitHub will provision a Let's Encrypt SSL certificate (takes a few minutes)

### 7. Verify Deployment

1. Push a test commit:
   ```bash
   # Make a small change to any content file
   git add .
   git commit -m "Test deployment"
   git push
   ```

2. Watch the GitHub Actions build:
   - Go to **Actions** tab in your repository
   - You should see the "Deploy Hugo site to GitHub Pages" workflow running
   - Wait for it to complete (usually 1-2 minutes)

3. Visit your site:
   - https://blog.abcdenzel.xyz

## Before Going Live

### Content Review

1. **Add your email** in:
   - `content/about.md` (line 61)
   - `content/author.md` (line 69)

2. **Review and publish posts**:

   Currently all new posts are marked as `draft = true`. Change to `draft = false` for posts you want published:

   - `content/posts/meta/theory-of-lifecycles.md` ✓ (reviewed and edited)
   - `content/posts/tech/block-devices-and-filesystems.md` (new)
   - `content/posts/tech/software-development-paradigms.md` (new)
   - `content/posts/tech/infrastructure-as-code-zero-to-deployed.md` (new)

   Empty drafts to consider removing or completing:
   - `content/posts/meta/the-reason-for-blogging.md` (empty)
   - `content/posts/meta/importance-of-good-conversation.md` (empty)
   - `content/posts/tech/NetSecOps.md` (empty)
   - `content/posts/tech/agentic-network-intrusion-detection.md` (empty)
   - `content/posts/tech/network-traffic-anomaly-detection.md` (empty)

3. **Test locally before pushing**:
   ```bash
   # Build and serve locally with drafts hidden (production mode)
   hugo serve

   # Check for broken links, formatting issues
   # Verify all pages load correctly
   ```

## Troubleshooting

### Build Fails on GitHub Actions

Check the Actions tab for errors. Common issues:
- Submodules not properly initialized (themes/)
- Syntax errors in frontmatter
- Missing dependencies

Fix:
```bash
# Ensure submodules are tracked
git submodule add https://github.com/LordMathis/hugo-theme-nightfall.git themes/nightfall
git submodule update --init --recursive
git commit -m "Fix submodules"
git push
```

### Custom Domain Shows 404

- Verify CNAME file exists in root of repository
- Check DNS with `dig blog.abcdenzel.xyz` - should point to GitHub
- Wait longer for DNS propagation
- Verify `baseURL` in `hugo.toml` matches custom domain

### HTTPS Certificate Error

- Wait 10-15 minutes after enabling "Enforce HTTPS"
- Disable and re-enable "Enforce HTTPS" in GitHub Pages settings
- Verify DNS is correctly configured
- Try accessing via http:// first, then https://

### Theme Not Loading

- Check git submodules are initialized
- Verify `theme = "diary"` in `hugo.toml`
- Check `.github/workflows/deploy.yml` includes `submodules: recursive`

## Post-Deployment

### Regular Workflow

1. Create content locally:
   ```bash
   hugo new content/posts/tech/my-new-post.md
   ```

2. Write and preview:
   ```bash
   hugo serve -D  # See drafts while writing
   ```

3. Publish:
   ```bash
   # Change draft = true to draft = false in frontmatter
   git add .
   git commit -m "New post: My New Post"
   git push
   ```

4. Verify deployment in GitHub Actions tab

### Updating Hugo Version

Edit `.github/workflows/deploy.yml`:
```yaml
env:
  HUGO_VERSION: 0.152.2  # Update this version
```

### Analytics (Optional)

To add Google Analytics, uncomment and configure in `hugo.toml`:
```toml
[services]
  [services.googleAnalytics]
    id = "G-XXXXXXXXXX"  # Your Google Analytics ID
```

## Backup Strategy

The git repository IS your backup. Keep it safe:

```bash
# Clone to another location periodically
git clone git@github.com:yourusername/blog.git ~/backups/blog

# Or add a second remote (GitLab, Bitbucket, etc.)
git remote add backup git@gitlab.com:yourusername/blog.git
git push backup master
```

## Need Help?

- [Hugo Forums](https://discourse.gohugo.io/)
- [GitHub Pages Documentation](https://docs.github.com/en/pages)
- [Hugo on GitHub Pages Guide](https://gohugo.io/host-and-deploy/host-on-github-pages/)
