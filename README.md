# The Lane Programming Language

This repository contains the mdBook sources for The Lane Programming Language.

- English: `en/`
- Chinese: `zh/`

## Local preview

```sh
mdbook serve en
mdbook serve zh
```

## Build

```sh
mdbook build en
mdbook build zh
```

## GitHub Pages

This repository publishes through the workflow in `.github/workflows/pages.yml`.
In the GitHub repository settings, set Pages source to GitHub Actions. The
published site provides:

- `/en/` for English
- `/zh/` for Chinese
