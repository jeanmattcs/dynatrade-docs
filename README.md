# dynatrade-docs

Public documentation for DynaTrade, built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/).

This docs repo tracks the public operator and player-facing surface of the `0.7.0` line, including:

- installation and upgrade guidance
- configuration and item tuning
- command and permission reference
- admin operations and troubleshooting
- the current production posture of the Free runtime

## Structure

- `docs/` - user-facing pages
- `mkdocs.yml` - site navigation and theme configuration

## Publishing

With MkDocs Material installed:

```bash
mkdocs gh-deploy
```

## Sync policy

Internal architecture notes, audits, and milestone planning stay in the private repository. This public docs repo should reflect:

- current supported installation flow
- current config surface
- current runtime behavior that matters to operators
- current validated version and production defaults

When the plugin behavior changes, update this repo before the next public release or distribution push.

