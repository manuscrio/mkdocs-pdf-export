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
npx manuscrio@0.1.0 export ./site --logo ./docs/assets/logo.svg --theme lapis
```

That writes one PDF per documentation edition into `./manuscrio-output`.

`--theme lapis` sets the accent on top-level chapter titles and the contents table. Five themes
ship, named for mineral pigments; the default `ink` carries no accent at all, so an unthemed manual
stays readable printed in greyscale. See [Branding](https://manuscrio.com/docs/branding/).

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

This repository **is** a GitHub Action. Point it at the directory your build produced:

```yaml
- run: pip install -r requirements.txt && mkdocs build

- uses: manuscrio/mkdocs-pdf-export@v1
  with:
    build: site
    logo: docs/assets/logo.svg
    theme: lapis

- uses: actions/upload-artifact@v7
  with:
    name: manual
    path: manuscrio-output/*.pdf
```

| Input | Default | |
| --- | --- | --- |
| `build` | `site` | the directory your build produced |
| `output-dir` | `manuscrio-output` | where the manuals are written |
| `scope` | engine default | `edition`, `section` or `sidebar-root` |
| `theme` | `ink` | `ink`, `lapis`, `malachite`, `garnet`, `amethyst` |
| `logo` | — | cover and running-header mark. **Supply one**: Material describes no logo an exporter can read |
| `concurrency` | `4` | parallel renders — each is a browser, so lower it on a small runner |
| `license` | — | pass from a secret, never a committed file |

[`.github/workflows/example.yml`](.github/workflows/example.yml) runs exactly this against the
example with `uses: ./`, so CI here proves the Action itself rather than only the command it wraps.

On GitLab CI or any other provider there is no Action to use — call the wrapper directly, as in
[Try it](#try-it) above. See [Run in CI](https://manuscrio.com/docs/ci/).

## The example in this repository

[`example/`](example/) is a small but real MkDocs Material site — six pages across two navigation
sections. CI builds it and exports it on every push, so the PDF is a downloadable artifact on [the
latest run](https://github.com/manuscrio/mkdocs-pdf-export/actions/workflows/example.yml).

Copy it, or copy just the workflow.

To drive it locally, the repository carries a [mise](https://mise.jdx.dev) config pinning the same
Python and Node versions CI uses, and creating the virtualenv for you:

```bash
mise trust     # mise asks before running a config it has not seen before
mise install
mise run build      # build the example site
mise run inspect    # what manuals does this build contain?
mise run export     # one manual per edition, with --logo
```

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

In a real pipeline, supply one from a secret:

```yaml
- uses: manuscrio/mkdocs-pdf-export@v1
  with:
    license: ${{ secrets.MANUSCRIO_LICENSE }}
```

See [Licensing](https://manuscrio.com/docs/licensing/) for how licences are issued and renewed.

## Licence

This repository — the example project, the workflows, and this README — is **MIT**. Copy it freely.

**The Manuscrio engine image it runs is proprietary software.** MIT covers the glue in this
repository and nothing else.

---

[manuscrio.com](https://manuscrio.com) · [`manuscrio` on npm](https://www.npmjs.com/package/manuscrio) · [Docusaurus](https://github.com/manuscrio/docusaurus-pdf-export) · [Starlight](https://github.com/manuscrio/starlight-pdf-export)
