# Setting Up GitHub Actions for Documentation Deployment

A step-by-step guide to automating your documentation workflow with GitHub Actions.

## Introduction

GitHub Actions lets you automatically build and deploy your documentation site whenever you push changes. Instead of manually running build commands and uploading files, your docs update automatically within minutes of committing changes.

**What you'll learn:**
- How GitHub Actions works
- Setting up automated deployment for documentation
- Configuring workflows for MkDocs and Docusaurus
- Troubleshooting common issues

**Prerequisites:**
- A GitHub account
- A documentation project (MkDocs or Docusaurus)
- Basic familiarity with Git
- No CI/CD experience required

**Time to complete:** 20-30 minutes

---

## Table of Contents

1. [Understanding GitHub Actions](#1-understanding-github-actions)
2. [How Documentation Deployment Works](#2-how-documentation-deployment-works)
3. [Setting Up GitHub Pages](#3-setting-up-github-pages)
4. [Creating Your First Workflow](#4-creating-your-first-workflow)
5. [MkDocs Deployment Workflow](#5-mkdocs-deployment-workflow)
6. [Docusaurus Deployment Workflow](#6-docusaurus-deployment-workflow)
7. [Understanding Workflow Syntax](#7-understanding-workflow-syntax)
8. [Testing Your Workflow](#8-testing-your-workflow)
9. [Troubleshooting](#9-troubleshooting)
10. [Advanced Configurations](#10-advanced-configurations)

---

## 1. Understanding GitHub Actions

GitHub Actions is a CI/CD (Continuous Integration/Continuous Deployment) platform built into GitHub. It automates tasks whenever something happens in your repository.

### Key Concepts

| Term | Description |
|------|-------------|
| **Workflow** | An automated process defined in a YAML file |
| **Event** | Something that triggers a workflow (push, pull request, etc.) |
| **Job** | A set of steps that run on the same machine |
| **Step** | An individual task within a job |
| **Action** | A reusable unit of code (like a plugin) |
| **Runner** | The server that runs your workflow |

### How It Works

```
┌─────────────────────────────────────────────────────────────┐
│                     YOUR REPOSITORY                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   1. You push changes to main branch                        │
│              │                                               │
│              ▼                                               │
│   2. GitHub detects the push event                          │
│              │                                               │
│              ▼                                               │
│   3. Workflow file (.github/workflows/deploy.yml) runs      │
│              │                                               │
│              ▼                                               │
│   4. Runner builds your documentation                       │
│              │                                               │
│              ▼                                               │
│   5. Built files deploy to GitHub Pages                     │
│              │                                               │
│              ▼                                               │
│   6. Your site is live at username.github.io/repo           │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. How Documentation Deployment Works

Documentation sites like MkDocs and Docusaurus work similarly:

1. **Source files** — You write content in Markdown
2. **Build process** — The tool converts Markdown to HTML
3. **Output** — Static HTML/CSS/JS files ready for hosting
4. **Deployment** — Files are uploaded to a web server

**Without automation:**
```bash
# You run these commands manually every time
mkdocs build          # or npm run build
# Then upload files somewhere
```

**With GitHub Actions:**
```yaml
# This runs automatically when you push
- run: mkdocs build
- run: deploy to GitHub Pages
```

---

## 3. Setting Up GitHub Pages

Before creating a workflow, enable GitHub Pages for your repository.

### Step 1: Go to Repository Settings

1. Navigate to your repository on GitHub
2. Click **Settings** (gear icon)
3. Scroll down and click **Pages** in the left sidebar

### Step 2: Configure Pages Source

1. Under **Build and deployment**, find **Source**
2. Select **GitHub Actions** from the dropdown

```
Build and deployment
─────────────────────────────────────
Source
┌────────────────────────────────┐
│ GitHub Actions              ▼  │
└────────────────────────────────┘

Your site will be built and deployed using a workflow.
```

> **Note:** The older method used a `gh-pages` branch. The newer GitHub Actions method is simpler and recommended.

### Step 3: Note Your Site URL

Your documentation will be available at:
```
https://<username>.github.io/<repository-name>/
```

For example:
- Username: `henrywinnerman`
- Repository: `api-docs`
- URL: `https://henrywinnerman.github.io/api-docs/`

---

## 4. Creating Your First Workflow

Workflow files live in the `.github/workflows/` directory of your repository.

### Step 1: Create the Workflows Directory

In your repository:

1. Click **Add file** → **Create new file**
2. Type: `.github/workflows/deploy.yml`
3. This creates both folders and the file

### Step 2: Add Workflow Content

We'll start with a minimal example, then show complete workflows for MkDocs and Docusaurus.

**Basic structure:**

```yaml
name: Deploy Documentation

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      # Build and deploy steps go here
```

### Step 3: Commit the File

1. Scroll down to **Commit new file**
2. Add a commit message: "Add deployment workflow"
3. Click **Commit new file**

The workflow will run immediately after committing.

---

## 5. MkDocs Deployment Workflow

MkDocs is a Python-based static site generator popular for technical documentation.

### Complete Workflow File

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy MkDocs to GitHub Pages

# Run on pushes to main branch
on:
  push:
    branches:
      - main
  
  # Allow manual trigger from Actions tab
  workflow_dispatch:

# Set permissions for GitHub Pages deployment
permissions:
  contents: read
  pages: write
  id-token: write

# Prevent multiple deployments running simultaneously
concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      # Step 1: Get the code
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history for git-revision-date plugin

      # Step 2: Set up Python
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.x'

      # Step 3: Install MkDocs and dependencies
      - name: Install dependencies
        run: |
          pip install mkdocs
          pip install mkdocs-material

      # Step 4: Build the documentation
      - name: Build documentation
        run: mkdocs build

      # Step 5: Configure GitHub Pages
      - name: Setup Pages
        uses: actions/configure-pages@v4

      # Step 6: Upload the built site
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: 'site'  # MkDocs outputs to 'site' folder

  deploy:
    needs: build
    runs-on: ubuntu-latest
    
    # Deploy to GitHub Pages environment
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}

    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### Required Files in Your Repository

Make sure your repository has:

```
your-repo/
├── .github/
│   └── workflows/
│       └── deploy.yml
├── docs/
│   ├── index.md
│   └── other-pages.md
└── mkdocs.yml
```

### Sample mkdocs.yml

```yaml
site_name: My Documentation
site_url: https://username.github.io/repo-name/

theme:
  name: material

nav:
  - Home: index.md
  - Getting Started: getting-started.md
  - API Reference: api-reference.md
```

### Adding More Plugins

If you use MkDocs plugins, add them to the install step:

```yaml
- name: Install dependencies
  run: |
    pip install mkdocs
    pip install mkdocs-material
    pip install mkdocs-minify-plugin
    pip install mkdocs-git-revision-date-localized-plugin
```

Or use a `requirements.txt` file:

```yaml
- name: Install dependencies
  run: pip install -r requirements.txt
```

---

## 6. Docusaurus Deployment Workflow

Docusaurus is a JavaScript-based documentation framework from Meta.

### Complete Workflow File

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy Docusaurus to GitHub Pages

on:
  push:
    branches:
      - main
  
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      # Step 1: Get the code
      - name: Checkout repository
        uses: actions/checkout@v4

      # Step 2: Set up Node.js
      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'

      # Step 3: Install dependencies
      - name: Install dependencies
        run: npm ci

      # Step 4: Build the documentation
      - name: Build documentation
        run: npm run build

      # Step 5: Configure GitHub Pages
      - name: Setup Pages
        uses: actions/configure-pages@v4

      # Step 6: Upload the built site
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: 'build'  # Docusaurus outputs to 'build' folder

  deploy:
    needs: build
    runs-on: ubuntu-latest
    
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}

    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### Required Configuration

Update your `docusaurus.config.js`:

```javascript
const config = {
  // Your GitHub username or organization
  organizationName: 'your-username',
  
  // Your repository name
  projectName: 'your-repo-name',
  
  // GitHub Pages URL
  url: 'https://your-username.github.io',
  
  // Base URL (repository name with slashes)
  baseUrl: '/your-repo-name/',
  
  // Deployment settings
  trailingSlash: false,
  
  // ... rest of your config
};
```

### Using Yarn Instead of npm

If your project uses Yarn:

```yaml
- name: Set up Node.js
  uses: actions/setup-node@v4
  with:
    node-version: '18'
    cache: 'yarn'

- name: Install dependencies
  run: yarn install --frozen-lockfile

- name: Build documentation
  run: yarn build
```

---

## 7. Understanding Workflow Syntax

Let's break down the key parts of a workflow file.

### Workflow Name

```yaml
name: Deploy Documentation
```

This appears in the Actions tab on GitHub.

### Triggers (on)

```yaml
on:
  push:
    branches:
      - main
```

**Common triggers:**

| Trigger | When it runs |
|---------|--------------|
| `push` | When code is pushed |
| `pull_request` | When a PR is opened/updated |
| `workflow_dispatch` | Manual trigger from GitHub UI |
| `schedule` | On a cron schedule |

**Only run on changes to docs:**

```yaml
on:
  push:
    branches:
      - main
    paths:
      - 'docs/**'
      - 'mkdocs.yml'
```

### Permissions

```yaml
permissions:
  contents: read
  pages: write
  id-token: write
```

These grant the workflow access to:
- Read repository contents
- Write to GitHub Pages
- Authenticate for deployment

### Jobs and Steps

```yaml
jobs:
  build:                    # Job name
    runs-on: ubuntu-latest  # Machine type
    
    steps:
      - name: Step description
        uses: action/name@version  # Use a pre-built action
        with:
          parameter: value
      
      - name: Another step
        run: |                     # Run shell commands
          command one
          command two
```

### Using Actions

Actions are reusable pieces of code:

```yaml
# Official GitHub actions
- uses: actions/checkout@v4
- uses: actions/setup-python@v5
- uses: actions/setup-node@v4

# Third-party actions
- uses: username/action-name@v1
```

### Running Commands

```yaml
# Single command
- run: npm install

# Multiple commands
- run: |
    npm install
    npm run build
    npm run test
```

---

## 8. Testing Your Workflow

### Viewing Workflow Runs

1. Go to your repository on GitHub
2. Click the **Actions** tab
3. See all workflow runs and their status

```
Actions
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
All workflows

Deploy Documentation
  ✓ Add new API docs page           2 min ago
  ✓ Update getting started guide    1 hour ago
  ✗ Fix typo in index               2 hours ago
  ✓ Initial documentation           1 day ago
```

### Checking Run Details

Click on a workflow run to see:
- Which jobs ran
- Duration of each step
- Logs for debugging
- Artifacts produced

### Manual Trigger

If you added `workflow_dispatch`:

1. Go to **Actions** tab
2. Select your workflow on the left
3. Click **Run workflow** button
4. Select branch and click **Run workflow**

### Testing Locally First

Before pushing, test your build locally:

**MkDocs:**
```bash
mkdocs build
mkdocs serve  # Preview at localhost:8000
```

**Docusaurus:**
```bash
npm run build
npm run serve  # Preview at localhost:3000
```

---

## 9. Troubleshooting

### Common Issues and Solutions

#### Build Fails: "Module not found"

**Problem:** Missing dependencies

**Solution:** Add the missing package to your install step:

```yaml
- name: Install dependencies
  run: |
    pip install mkdocs
    pip install mkdocs-material
    pip install missing-plugin  # Add this
```

#### Deploy Fails: "Permission denied"

**Problem:** Workflow lacks permissions

**Solution:** Add permissions block:

```yaml
permissions:
  contents: read
  pages: write
  id-token: write
```

Also check repository settings:
1. Go to **Settings** → **Actions** → **General**
2. Under **Workflow permissions**, select **Read and write permissions**

#### Site Shows 404

**Problem:** Pages not configured correctly

**Solution:**
1. Go to **Settings** → **Pages**
2. Ensure **Source** is set to **GitHub Actions**
3. Wait 2-3 minutes after deployment

#### Wrong Base URL

**Problem:** Links and assets broken

**Solution:** Configure base URL correctly:

**MkDocs (mkdocs.yml):**
```yaml
site_url: https://username.github.io/repo-name/
```

**Docusaurus (docusaurus.config.js):**
```javascript
baseUrl: '/repo-name/',
```

#### Workflow Not Triggering

**Problem:** Workflow doesn't run on push

**Solution:** Check:
1. File is in `.github/workflows/` directory
2. Branch name matches trigger (`main` vs `master`)
3. YAML syntax is valid

### Reading Workflow Logs

When a workflow fails:

1. Click on the failed run
2. Click on the failed job
3. Expand the failed step
4. Read the error message

```
Run mkdocs build
  Error: Config file 'mkdocs.yml' does not exist.
  ##[error]Process completed with exit code 1.
```

This tells you exactly what went wrong.

---

## 10. Advanced Configurations

### Deploy Only When Docs Change

```yaml
on:
  push:
    branches:
      - main
    paths:
      - 'docs/**'
      - 'mkdocs.yml'
      - '.github/workflows/deploy.yml'
```

### Build Preview for Pull Requests

Add a separate workflow for PR previews:

```yaml
name: Test Documentation Build

on:
  pull_request:
    branches:
      - main

jobs:
  test-build:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-python@v5
        with:
          python-version: '3.x'
      
      - run: pip install mkdocs mkdocs-material
      
      - name: Test build
        run: mkdocs build --strict
```

This validates the build without deploying.

### Cache Dependencies

Speed up builds by caching:

**Python/pip:**
```yaml
- uses: actions/setup-python@v5
  with:
    python-version: '3.x'
    cache: 'pip'
```

**Node.js/npm:**
```yaml
- uses: actions/setup-node@v4
  with:
    node-version: '18'
    cache: 'npm'
```

### Multiple Documentation Sites

Deploy different docs to different paths:

```yaml
jobs:
  deploy-api-docs:
    # ... build API docs
    
  deploy-user-guide:
    # ... build user guide
```

### Scheduled Rebuilds

Rebuild docs daily (useful if content includes dates):

```yaml
on:
  push:
    branches:
      - main
  schedule:
    - cron: '0 0 * * *'  # Daily at midnight UTC
```

### Environment Variables

Pass configuration to your build:

```yaml
- name: Build documentation
  run: mkdocs build
  env:
    SITE_URL: https://docs.example.com
    ANALYTICS_ID: UA-12345678-1
```

### Notifications

Add Slack notification on failure:

```yaml
- name: Notify on failure
  if: failure()
  uses: slackapi/slack-github-action@v1
  with:
    channel-id: 'docs-alerts'
    slack-message: 'Documentation build failed!'
  env:
    SLACK_BOT_TOKEN: ${{ secrets.SLACK_TOKEN }}
```

---

## Quick Reference

### Workflow File Location

```
.github/workflows/deploy.yml
```

### Minimum Permissions

```yaml
permissions:
  contents: read
  pages: write
  id-token: write
```

### Common Actions

| Action | Purpose |
|--------|---------|
| `actions/checkout@v4` | Get repository code |
| `actions/setup-python@v5` | Install Python |
| `actions/setup-node@v4` | Install Node.js |
| `actions/configure-pages@v4` | Configure GitHub Pages |
| `actions/upload-pages-artifact@v3` | Upload built files |
| `actions/deploy-pages@v4` | Deploy to GitHub Pages |

### Output Directories

| Tool | Build Command | Output Directory |
|------|---------------|------------------|
| MkDocs | `mkdocs build` | `site/` |
| Docusaurus | `npm run build` | `build/` |
| Jekyll | `jekyll build` | `_site/` |
| Hugo | `hugo` | `public/` |

### Checking Deployment

After successful deployment:
1. Go to **Settings** → **Pages**
2. Find the URL under **Your site is live at**
3. Click the URL to view your documentation

---

## Summary

You've learned how to:

1. **Enable GitHub Pages** — Configure your repository for Pages deployment
2. **Create workflows** — Write YAML files that automate your builds
3. **Deploy MkDocs** — Set up Python-based documentation deployment
4. **Deploy Docusaurus** — Set up JavaScript-based documentation deployment
5. **Troubleshoot issues** — Debug common problems with builds and deployments

Your documentation now updates automatically whenever you push changes to your repository.

---

## Next Steps

- **Add a custom domain** — Point your own domain to GitHub Pages
- **Set up branch protection** — Require successful builds before merging
- **Add quality checks** — Lint Markdown, check links, validate spelling
- **Create preview environments** — Deploy PR previews for review

---

## Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [GitHub Pages Documentation](https://docs.github.com/en/pages)
- [MkDocs Documentation](https://www.mkdocs.org/)
- [Docusaurus Documentation](https://docusaurus.io/docs)

---

*Last updated: January 2026*
