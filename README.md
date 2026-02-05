# Project Steakholder Documentation

Documentation site for Project Steakholder, an AI-driven stakeholder simulation tool for requirements elicitation practice.

Built with [Just the Docs](https://just-the-docs.github.io/just-the-docs/) and hosted on GitHub Pages.

**Live site:** https://project-steak-holder.github.io/project-docs/

## Local Development

### Prerequisites

- Ruby 3.x
- Bundler (`gem install bundler`)

### Setup

```bash
git clone https://github.com/project-steak-holder/project-docs.git
cd project-docs
bundle config set --local path 'vendor/bundle'
bundle install
```

### Running the Server

```bash
bundle exec jekyll serve
```

Open http://localhost:4000/project-docs/ in your browser.

To stop the server, press `Ctrl+C`.

For live reload during development:

```bash
bundle exec jekyll serve --livereload
```

## Project Structure

```
project-docs/
├── _config.yml          # Jekyll configuration
├── docs/                # Documentation pages
├── assets/              # Images and static files
├── index.md             # Home page
├── Gemfile              # Ruby dependencies
└── README.md
```

## Adding Pages

Create a new Markdown file in `docs/` with frontmatter:

```yaml
---
layout: default
title: Page Title
nav_order: 1
---
```

The `nav_order` determines sidebar position (lower numbers appear first).

## Troubleshooting

**`bundle install` fails with native extension errors:**
```bash
sudo apt install build-essential ruby-dev
```

**Port 4000 already in use:**
```bash
bundle exec jekyll serve --port 4001
```

**Changes not appearing:**
```bash
bundle exec jekyll clean
bundle exec jekyll serve
```

## Installing Ruby (Ubuntu/WSL)

```bash
sudo apt update
sudo apt install rbenv libyaml-dev libffi-dev
rbenv install 3.2.0
rbenv global 3.2.0
```

Restart your shell, then verify with `ruby --version`.
