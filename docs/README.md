# Invoice Collector Documentation

This directory contains the documentation for Invoice Collector built with [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/).

## Local Development

### Prerequisites

- Python 3.8+
- pip

### Installation

```bash
pip install mkdocs-material
```

### Serve Locally

```bash
mkdocs serve
```

Then open http://127.0.0.1:8000 in your browser.

### Build

```bash
mkdocs build
```

The static site will be generated in the `site/` directory.

## Structure

```
docs/
├── index.md                    # Home page
├── getting-started/            # Installation and setup
│   ├── installation.md
│   ├── quick-start.md
│   └── configuration.md
├── guides/                     # User guides
│   ├── overview.md
│   ├── collectors.md
│   ├── credentials.md
│   └── invoices.md
├── api/                        # API documentation
│   ├── overview.md
│   ├── authentication.md
│   ├── endpoints.md
│   └── webhooks.md
├── developers/                 # Developer guides
│   ├── setup.md
│   ├── architecture.md
│   ├── creating-collectors.md
│   ├── testing.md
│   └── contributing.md
└── deployment/                 # Deployment guides
    ├── docker.md
    ├── environment-variables.md
    └── database.md
```

## Contributing

To contribute to the documentation:

1. Fork the repository
2. Create a new branch
3. Make your changes
4. Test locally with `mkdocs serve`
5. Submit a pull request

## Deployment

Documentation is automatically deployed to GitHub Pages when changes are pushed to the `master` branch.

The deployment is handled by the `.github/workflows/docs.yml` GitHub Action.

## Style Guide

- Use clear, concise language
- Include code examples where appropriate
- Use admonitions for tips, warnings, and notes
- Keep line length reasonable for readability
- Use proper Markdown formatting

### Admonitions

```markdown
!!! note
    This is a note

!!! tip
    This is a tip

!!! warning
    This is a warning

!!! danger
    This is a danger notice
```

### Code Blocks

Always specify the language:

```markdown
​```bash
npm install
​```

​```javascript
const app = express();
​```
```

## License

The documentation is licensed under the same license as the main project.
