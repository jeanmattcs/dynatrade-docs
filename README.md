# dynatrade-docs

Public documentation for DynaTrade, built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/).

## Structure

- `docs/` — user-facing pages (installation, configuration, commands, GUI, etc.)
- `mkdocs.yml` — site nav and theme config

## Publishing

This repo is meant to be published via GitHub Pages. With MkDocs Material installed:

```
mkdocs gh-deploy
```

## Keeping docs in sync

Internal planning, audits, and architecture notes stay in the main private repo. This repo holds only the public-facing content. When plugin behavior changes, update here before the next release.
