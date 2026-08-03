# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

An HSF (HEP Software Foundation) training lesson, "Introduction to Docker and Podman", built as a
Jupyter Book v1 (classic, Sphinx-based) site. The published site is deployed by GitHub Actions
(`.github/workflows/book.yml`) from the `gh-pages` branch — which is also the default/PR branch,
not `main`. There is no application code; the "source" is Markdown lesson content in
`docker_podman/`.

## Commands

```sh
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt          # jupyter-book 1.0.4 pinned
jupyter-book build docker_podman/        # build to docker_podman/_build/html
jupyter-book build -W --keep-going docker_podman/   # strict build — this is what CI runs
jupyter-book clean docker_podman/        # clear the build cache
python -m http.server -d docker_podman/_build/html 8000   # local preview
pre-commit run --all-files               # lint: whitespace/EOF fixers + codespell
```

## Content structure

- `docker_podman/NN-*.md` — the lesson chapters, ordered by numeric filename prefix (gaps in
  numbering are fine, e.g. 07 → 11). Page order is defined by `docker_podman/_toc.yml`; every page
  MUST be listed there (`only_build_toc_files: true` ignores unlisted files).
- `docker_podman/intro.md` — landing page (`root` in `_toc.yml`), `docker_podman/setup.md` —
  installation instructions.
- `docker_podman/fig/` — images, referenced via MyST `{figure}`/`{image}` directives with
  `fig/...` paths.
- Pages use MyST conventions: colon-fence admonitions (`:::{admonition} Title` with `:class: tip`,
  `important`, `note`, `caution`; solutions use `:class: dropdown`). Nested admonitions need more
  colons on the outer fence than the inner. Each page has exactly one H1 on line 1, an Overview
  admonition at the top, and (for episodes) a Key Points admonition at the end.
- `examples/main.c` is referenced from episode 06 by raw GitHub URL — don't delete it.

## Repo conventions

- Content edits must keep the strict build green: `jupyter-book build -W --keep-going
  docker_podman/` must exit 0 (CI treats warnings as errors).
- Content edits should pass pre-commit (trailing whitespace, end-of-file, codespell on `*.md`)
  before committing; CI runs the same hooks.
- `Migration.md` at the repo root is the historical playbook used to convert this lesson from
  Jekyll/Carpentries to Jupyter Book; it is kept for reference only.

## Instructions for Claude Code

Do not commit as "Co-authored", ALWAYS say "Assisted with: Claude Code"
