# Invoice Collector Documentation

This project now includes comprehensive documentation built with Material for MkDocs.

## What Was Added

### Documentation Structure

A complete documentation site with the following sections:

#### 📚 Getting Started
- **Installation** - Step-by-step installation guide using Docker
- **Quick Start** - Get started quickly with your first invoice collection
- **Configuration** - Complete environment variable reference and Bitwarden setup

#### 📖 User Guide
- **Overview** - Core concepts and workflows
- **Collectors** - Available suppliers and collector types
- **Credentials** - Managing supplier credentials securely
- **Invoices** - Working with collected invoices

#### 🔌 API Reference
- **Overview** - API introduction and quick start
- **Authentication** - Bearer token and UI token authentication
- **Endpoints** - Complete REST API endpoint reference
- **Webhooks** - Callback configuration and implementation

#### 💻 Developer Guide
- **Setup** - Development environment setup
- **Architecture** - System architecture and component overview
- **Creating Collectors** - Step-by-step guide to create new collectors
- **Testing** - Manual and automated testing guides
- **Contributing** - How to contribute to the project

#### 🚀 Deployment
- **Docker** - Production deployment with Docker
- **Environment Variables** - Complete variable reference
- **Database** - MongoDB setup and management

## Features

### Material for MkDocs Theme
- 🎨 Modern, responsive design
- 🌓 Light/Dark mode toggle
- 🔍 Full-text search
- 📱 Mobile-friendly
- ⚡ Fast and lightweight

### Documentation Features
- ✅ Code syntax highlighting
- ✅ Tabbed content sections
- ✅ Admonitions (notes, tips, warnings)
- ✅ Mermaid diagrams for architecture
- ✅ Copy-to-clipboard for code blocks
- ✅ Navigation breadcrumbs
- ✅ Table of contents on each page

## Building the Documentation

### Locally

1. Install dependencies:
   ```bash
   pip install mkdocs-material
   ```

2. Serve locally:
   ```bash
   mkdocs serve
   ```
   
   Open http://127.0.0.1:8000

3. Build static site:
   ```bash
   mkdocs build
   ```

### Deployment

The documentation is automatically deployed to GitHub Pages when changes are pushed to the `master` branch via the `.github/workflows/docs.yml` workflow.

## File Structure

```
invoice-collector/
├── mkdocs.yml                          # MkDocs configuration
├── docs/                               # Documentation source
│   ├── index.md                        # Homepage
│   ├── getting-started/
│   │   ├── installation.md
│   │   ├── quick-start.md
│   │   └── configuration.md
│   ├── guides/
│   │   ├── overview.md
│   │   ├── collectors.md
│   │   ├── credentials.md
│   │   └── invoices.md
│   ├── api/
│   │   ├── overview.md
│   │   ├── authentication.md
│   │   ├── endpoints.md
│   │   └── webhooks.md
│   ├── developers/
│   │   ├── setup.md
│   │   ├── architecture.md
│   │   ├── creating-collectors.md
│   │   ├── testing.md
│   │   └── contributing.md
│   └── deployment/
│       ├── docker.md
│       ├── environment-variables.md
│       └── database.md
└── .github/workflows/
    └── docs.yml                        # GitHub Pages deployment
```

## Accessing the Documentation

Once deployed to GitHub Pages, the documentation will be available at:
https://eckphi.github.io/invoice-collector/

## Contributing to Documentation

See `docs/README.md` for guidelines on contributing to the documentation.

## Technologies Used

- **MkDocs** - Static site generator
- **Material for MkDocs** - Modern theme
- **Python Markdown Extensions** - Enhanced Markdown features
- **Pygments** - Syntax highlighting
- **GitHub Actions** - Automated deployment
