# DynaTrade Docs

Public-facing documentation for DynaTrade.

This folder is a standalone scaffold for a public docs repository. It is based on the current internal wiki and is meant to be copied into its own public GitHub repository, such as `jeanmattcs/dynatrade-docs`.

## Contents

- `docs/` - markdown pages for users and server operators
- `mkdocs.yml` - ready-to-publish navigation for a MkDocs site

## Suggested publish flow

1. Create a new public GitHub repository for the docs.
2. Copy the contents of this folder into that repository root.
3. Push the repository.
4. Optionally publish it with MkDocs Material and GitHub Pages.

## Notes

- These docs are intended to stay user-facing and product-aligned.
- Internal planning, audits, and release work should stay in the private main repository.
- When product behavior changes, update the internal source docs first, then mirror the curated public version here.
