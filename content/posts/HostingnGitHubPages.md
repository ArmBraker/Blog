+++ 
draft = false
date = 2025-01-18T16:55:28+01:00
title = "Host on GitHub Pages"
description = "How to host Hugo on GitHub Pages"
slug = "hostingonGitHubPages"
authors = []
tags = [
    "github",
    "hugo",
    "hosting",

]
categories = [    
    "setup",
    "configuration",
    ]
externalLink = ""
series = []
+++

## Procedure

### Step 1 - Create a GitHub repository

- I created this repository via GitHub website.
- Name: `Blog`

### Step 2 - Push your local repository to GitHub

- We need to initiliaze git for remote repository:

 ```powershell
 git remote add origin https://github.com/ArmBraker/Blog.git
 ```

- Push the Code to GitHub

```powershell
git branch -M main
git push -u origin main
```

### Step 3

- Visit your GitHub repository. From the main menu choose **Settings > Pages**. In the center of your screen you will see this:
![Build and deployment](https://gohugo.io/hosting-and-deployment/hosting-on-github/gh-pages-1.png)

### Step 4

- Change the **Source** to `GitHub Actions`. The change is immediate; you do not have to press a Save button.
![Build and deployment](https://gohugo.io/hosting-and-deployment/hosting-on-github/gh-pages-2.png)

### Step 5

Create a file named hugo.yaml in a directory named .github/workflows.

```powershell
mkdir .github/workflows


    Directory: C:\Users\<reduced>\Coding\_Blog\blog\.github


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         1/18/2025   5:16 PM                workflows
```

### Step 6

Copy and paste the YAML below into the file you created. Change the branch name and Hugo version as needed.

```yaml
# Sample workflow for building and deploying a Hugo site to GitHub Pages
name: Deploy Hugo site to Pages

on:
  # Runs on pushes targeting the default branch
  push:
    branches:
      - main

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
# However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
concurrency:
  group: "pages"
  cancel-in-progress: false

# Default to bash
defaults:
  run:
    shell: bash

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.141.0
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: Install Dart Sass
        run: sudo snap install dart-sass
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
      - name: Build with Hugo
        env:
          HUGO_CACHEDIR: ${{ runner.temp }}/hugo_cache
          HUGO_ENVIRONMENT: production
          TZ: America/Los_Angeles
        run: |
          hugo \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  # Deployment job
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### Step 7

Commit and push the change to your GitHub repository.

```powershell
git add -A
git commit -m "Create hugo.yaml"
git push
```

### Step 8

From GitHub’s main menu, choose Actions. You will see something like this:
![Workflows](https://gohugo.io/hosting-and-deployment/hosting-on-github/gh-pages-3.png)

### Step 9

When GitHub has finished building and deploying your site, the color of the status indicator will change to green.
