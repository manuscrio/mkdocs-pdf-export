# MkDocs to PDF

Export a built **MkDocs** site using the **Material** theme to complete, publication-ready PDF
manuals.

[![Export the example to PDF](https://github.com/manuscrio/mkdocs-pdf-export/actions/workflows/example.yml/badge.svg)](https://github.com/manuscrio/mkdocs-pdf-export/actions/workflows/example.yml)

Manuscrio reads the **build directory** — the `site/` that `mkdocs build` produces. No plugin, no
`mkdocs.yml` changes, no running server. If the build succeeds, Manuscrio has everything it needs.

## Try it

You need Docker or Podman; the `manuscrio` command is a thin wrapper that runs the engine image.

```bash
mkdocs build
npx manuscrio@0.1.0 export ./site --logo ./docs/assets/logo.svg
```

That writes one PDF per documentation edition into `./manuscrio-output`.

## Material, specifically

**MkDocs itself prescribes no markup.** The theme supplies all of it, so an exporter reads a theme
rather than MkDocs — and Manuscrio reads Material. A build using the built-in theme, or another
third-party theme, emits unrelated markup and is not supported:

```text
No rendered MkDocs documentation pages found
```

That refusal is clean, and it is the point. An exporter that half-reads an unfamiliar theme
produces a plausible PDF missing content nobody notices until a customer does.

### Why `--logo`

Material's logo is not described in the built markup in a way an exporter can read, so Manuscrio
finds none and the manual carries no mark. `--logo` supplies one for the cover and the running
header. `manuscrio inspect ./site` reports an empty `logo` field, which is what that means.

## In GitHub Actions

```yaml
- run: pip install -r requirements.txt && mkdocs build

- name: Export the docs to PDF
  run: |
    npx --yes manuscrio@0.1.0 export site \
      --logo docs/assets/logo.svg \
      --output-dir manuscrio-output

- uses: actions/upload-artifact@v7
  with:
    name: manual
    path: manuscrio-output/*.pdf
```

[`.github/workflows/example.yml`](.github/workflows/example.yml) in this repository is the complete,
working version of that. GitLab CI and other providers: see [Run in
CI](https://manuscrio.com/docs/ci/).

## The example in this repository

[`example/`](example/) is a small but real MkDocs Material site — six pages across two navigation
sections. CI builds it and exports it on every push, so the PDF is a downloadable artifact on [the
latest run](https://github.com/manuscrio/mkdocs-pdf-export/actions/workflows/example.yml).

Copy it, or copy just the workflow.

## Editions and scopes

An edition is one plugin instance, one locale and one version. MkDocs has neither plugin instances
nor versions, so those are the constants `default` and `current`; the locale still separates
editions, so a multilingual site produces one edition per language.

Material's navigation has no section level, so `--scope section` does not apply. Two scopes do:

| Scope | Produces |
| --- | --- |
| `--scope edition` *(default)* | one manual per edition |
| `--scope sidebar-root` | one manual per navigation root |

Full reference is on [MkDocs to PDF](https://manuscrio.com/docs/frameworks/mkdocs/).

## Evaluation Mode

With no licence, Manuscrio produces **complete** manuals carrying an evaluation watermark. Nothing
is truncated and no feature is withheld. The PDF this repository's CI publishes is watermarked,
deliberately: a licence is a bearer token and does not belong in a public repository.

See [Licensing](https://manuscrio.com/docs/licensing/) for how to supply one in a real pipeline.

## Licence

This repository — the example project, the workflows, and this README — is **MIT**. Copy it freely.

**The Manuscrio engine image it runs is proprietary software.** MIT covers the glue in this
repository and nothing else.

---

[manuscrio.com](https://manuscrio.com) · [`manuscrio` on npm](https://www.npmjs.com/package/manuscrio) · [Docusaurus](https://github.com/manuscrio/docusaurus-pdf-export) · [Starlight](https://github.com/manuscrio/starlight-pdf-export)
