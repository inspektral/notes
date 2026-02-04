# Notes

A collection of research notes, guides, and documentation built with MkDocs.

## Setup

Install the required dependencies:

```bash
pip install -r requirements.txt
```

## Local Development

To preview the documentation locally:

```bash
mkdocs serve
```

This will start a local server at `http://localhost:8000` with live reload enabled.

## Building

To build the static site:

```bash
mkdocs build
```

The built site will be in the `site/` directory.

## Deployment

The documentation is automatically deployed to GitHub Pages via GitHub Actions when changes are pushed to the `main` branch.

**First-time setup:** See [GITHUB_PAGES_SETUP.md](GITHUB_PAGES_SETUP.md) for instructions on configuring GitHub Pages to use GitHub Actions as the deployment source.

### Manual Local Deployment (Not Recommended)

If you need to deploy manually from your local machine:

```bash
mkdocs gh-deploy
```

Note: The automated GitHub Actions workflow is the recommended deployment method.

## Features

- **Dark Theme**: Material theme with slate color scheme (dark mode by default)
- **Math Rendering**: LaTeX math equations using MathJax
- **Mermaid Diagrams**: Flowcharts and diagrams using Mermaid
- **Search**: Full-text search functionality
- **Navigation**: Hierarchical navigation with tabs
