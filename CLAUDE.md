# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

This repository is a Hugo static site for a Chinese personal blog titled `我的灵感记录`, served at `https://www.vdong.xyz/`. The site uses the `DoIt` Hugo theme from the `themes/DoIt` git submodule.

The content is primarily Chinese technical notes and personal writing. The main Hugo configuration lives in `config.toml`; it sets Chinese as the default language, enables CJK handling, uses the DoIt theme, configures Fuse search, Waline comments, Google Analytics, menu entries, taxonomies, and Git-based `lastmod` front matter handling.

## Common commands

```bash
# Initialize/update the theme submodule after cloning
git submodule init
git submodule update

# Run local development server with drafts
hugo server -D

# Build the static site
hugo

# Build with garbage collection and minification, matching the current validation command
hugo --gc --minify

# Check the installed Hugo version
hugo version
```

There is no project-specific test suite in this repository. Validate content/configuration changes by running `hugo --gc --minify` and reviewing any warnings or errors.

## Content architecture

- `content/` contains the site pages and posts.
- Content pages are organized as Hugo leaf Page Bundles: each page directory contains an `index.md` file and any page-local assets such as images.
- Use relative Markdown image links for assets stored in the same bundle directory, for example `![alt](diagram.png)`.
- `content/posts/` contains blog posts grouped by topic directories such as `ddia`, `golang`, `php-web`, `unixs`, and `elastic`.
- Top-level content pages such as links/offline pages are also Page Bundles.
- `archetypes/default.md` is the template for new Hugo content. It currently sets `typora-root-url: .` so Typora image paths are bundle-local.

## Static and generated files

- `static/` contains global static files that should remain globally addressable, such as avatars and site manifest assets.
- Prefer page-local assets inside Page Bundles for article images instead of adding new article images under `static/imgs`.
- `public/` is Hugo's generated output directory. Treat it as build output; source content and configuration changes should usually be made under `content/`, `static/`, `archetypes/`, or `config.toml`.

## Theme and configuration

- `themes/DoIt` is a git submodule pointing to `https://github.com/HEIGE-PCloud/DoIt.git` on the `main` branch.
- Do not edit the theme submodule for ordinary content changes. Prefer Hugo configuration or project-local overrides when possible.
- Navigation, search, comments, analytics, taxonomies, and page behavior are configured in `config.toml`.
- The home page profile uses `/imgs/avatar.webp`, so keep global avatar assets available under `static/imgs/` unless the configuration is changed.

## Known build warnings

A normal `hugo --gc --minify` build currently succeeds but may warn that raw HTML was omitted in some Markdown files unless Goldmark unsafe rendering is enabled or those pages are rewritten without raw HTML. Treat these warnings as content-rendering issues rather than test failures unless the task specifically targets them.
