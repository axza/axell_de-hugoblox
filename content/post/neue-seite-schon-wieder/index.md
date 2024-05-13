---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Neue Seite - schon wieder"
subtitle: ""
summary: "Warum ich Netlify den Rücken kehre und auf GitHub Pages setze (mit Hugo Blox)"
authors: [admin]
tags: [hugo, GitHub, academic, foss, golang]
categories: []
date: 2024-05-07T13:37:42+02:00
lastmod: 
featured: false
draft: false

image:
  caption: "Symbolbild: Schönes Bild"
  focal_point: "top"
  preview_only: false

editable: true

projects: [it]
---
## Hugo Blox
Hugo Blox ist wie ein Baukasten für Webseiten: Du kannst vorgefertigte Elemente einfach zusammenstecken und deine Seite individuell gestalten. meine bisherige Seite baute auf das project wowchemy auf, doch das gibt's so nicht mehr... Nunja es heißt einfach nur anders ;)

Unter [Hugo Blox](https://hugoblox.com/) ist das Projekt zu finden und verhält sich ähnlich wie wowchemy aber mit massig breaking changes ... 🙃 

Die Grundlage dazu bildet [HUGO](https://gohugo.io/), das in [golang](https://go.dev/) geschrieben eine statische Seite erzeugt.

Aus ein in paar dateien (yaml 🥲) lässt sich, aus vorgefertigten layout blöcken, die landing page zusammenstecken - irgendwie vergleichbar mit klemmbausteinen®️.

## GitHub pages
Wenn ich die Seite schonmal anfasse habe ich auch gleichmal *bye bye, **netlify*** gesagt da die Seite auch "super" mit den gh-pages auskommt.

hier die GitHub action, die das bauen und das deployment nach gh-pages übernimmt:

```yaml
name: Deploy website to GitHub Pages

env:
  WC_HUGO_VERSION: '0.124.1'

on:
  # Trigger the workflow every time you push to the `main` branch
  push:
    branches: ["main"]
  # Allows you to run this workflow manually from the Actions tab on GitHub.
  workflow_dispatch:

# Provide permission to clone the repo and deploy it to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  # Build website
  build:
    if: github.repository_owner != 'HugoBlox'
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          # Fetch history for Hugo's .GitInfo and .Lastmod
          fetch-depth: 0
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: ${{ env.WC_HUGO_VERSION }}
          extended: true
      - uses: actions/cache@v4
        with:
          path: /tmp/hugo_cache_runner/
          key: ${{ runner.os }}-hugomod-${{ hashFiles('**/go.mod') }}
          restore-keys: |
            ${{ runner.os }}-hugomod-
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      - name: Build with Hugo
        env:
          HUGO_ENVIRONMENT: production
        run: |
          echo "Hugo Cache Dir: $(hugo config | grep cachedir)"
          hugo --minify --baseURL "${{ steps.pages.outputs.base_url }}/"
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  # Deploy website to GitHub Pages hosting
  deploy:
    if: github.repository_owner != 'HugoBlox'
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

![image](https://github.com/axza/axell_de-hugoblox/assets/10170631/551b27a8-a21c-497a-a8ba-faffba89b022)

Performancewunder habe ich keine erwartet, dennoch für meine kleine seite völlig ausreichend. danke github🤗
