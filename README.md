# KNode Technical Documentation

Technical documentation for **KNode**, an AI-assisted matchmaking platform that connects African-focused nonprofits, collectives and funders. Every match is traced back to its source evidence.

Live site: https://akirachix.github.io/alya-technical-documentation/

## Sections

| Page | Covers |
|---|---|
| Overview | Product purpose, users and MVP scope |
| Architecture | System design and data flow |
| Backend | FastAPI, PostgreSQL and API design |
| AI Service | Evidence extraction, matching and explanations |
| Frontend Web | Dashboards for organisations and funders |
| Integrations | External data sources and services |
| Glossary | Key terms used across the project |

## Run locally

Requires [uv](https://docs.astral.sh/uv/).

```bash
git clone git@github.com:akirachix/alya-technical-documentation.git
cd alya-technical-documentation
uv sync
uv run mkdocs serve
```

Open http://127.0.0.1:8000/alya-technical-documentation/

## Project structure

```
.
├── docs/                  # Markdown pages
│   ├── assets/brand/      # Logo
│   ├── stylesheets/       # Custom theme styles
│   └── *.md
├── mkdocs.yml             # Site config and navigation
├── pyproject.toml         # Dependencies
├── uv.lock                # Pinned versions
└── .github/workflows/     # Auto-deploy to GitHub Pages
```

## Contributing

1. Create a branch from `main`, for example `docs/backend-auth`.
2. Edit or add pages in `docs/`. New pages must be added to `nav` in `mkdocs.yml`.
3. Preview with `uv run mkdocs serve`.
4. Open a pull request into `main`.

## Deployment

Merging to `main` triggers a GitHub Actions workflow that builds the site and publishes it to the `gh-pages` branch.

## Built with

[MkDocs](https://www.mkdocs.org/) and [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/),
