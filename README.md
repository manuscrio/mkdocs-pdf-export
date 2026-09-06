# MkDocs to PDF

Export a built **MkDocs** site using the **Material** theme to complete, publication-ready PDF
manuals.

**This repository is a runnable example, not a library or a plugin.** It holds a small MkDocs
Material site and the command that turns its build into a PDF, so you can watch the whole thing
work before pointing it at your own docs. Manuscrio is the product it demonstrates:
[manuscrio.com](https://manuscrio.com).

[![Export the example to PDF](https://github.com/manuscrio/mkdocs-pdf-export/actions/workflows/example.yml/badge.svg)](https://github.com/manuscrio/mkdocs-pdf-export/actions/workflows/example.yml)

Manuscrio reads the **build directory** — the `site/` that `mkdocs build` produces. No plugin, no
`mkdocs.yml` changes, no running server. If the build succeeds, Manuscrio has everything it needs.

## Try it

You need Docker or Podman; the `manuscrio` command is a thin wrapper that runs the engine image.

```bash
mkdocs build
npx manuscrio@1.0.0 export ./site --logo ./docs/assets/logo.svg --theme lapis
```

That writes one PDF per documentation edition into `./manuscrio-output`.

Every command and option it accepts is listed in the
[CLI reference](https://manuscrio.com/docs/cli/).

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

```yaml
- run: pip install -r requirements.txt && mkdocs build

- name: Export the docs to PDF
  run: npx --yes manuscrio@1.0.0 export site \
        --logo docs/assets/logo.svg \
        --theme lapis \
        --output-dir manuscrio-output

- uses: actions/upload-artifact@v7
  with:
    name: manual
    path: manuscrio-output/*.pdf
```

[`.github/workflows/example.yml`](.github/workflows/example.yml) in this repository is the complete,
working version of that, run against the example on every push.

The same command is the whole of it on any other provider — GitLab CI, Jenkins, Buildkite. See [Run
in CI](https://manuscrio.com/docs/ci/).

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

In a real pipeline, supply one from a secret. The wrapper passes `MANUSCRIO_LICENSE` to the
container **by name**, so the licence text never appears in a command line or in `ps` output on a
shared runner:

```yaml
- name: Export the docs to PDF
  env:
    MANUSCRIO_LICENSE: ${{ secrets.MANUSCRIO_LICENSE }}
  run: |
    npx --yes manuscrio@1.0.0 export site \
      --logo docs/assets/logo.svg \
      --output-dir manuscrio-output
```

See [Licensing](https://manuscrio.com/docs/licensing/) for how licences are issued and renewed.

## Getting help

**Something wrong with the export itself** — a manual missing content, a framework not detected, a
failure you cannot place: [When an export fails](https://manuscrio.com/docs/troubleshooting/) names
what each failure means, and [MkDocs to PDF](https://manuscrio.com/docs/frameworks/mkdocs/)
covers what Manuscrio does and does not read from a MkDocs build.

**Anything else** — a licence, a question about your own pipeline, or a bug in the export —
**[contact@manuscrio.com](mailto:contact@manuscrio.com)**.

Issues are closed on this repository deliberately. A report about a documentation export usually
carries your own documentation with it, which is often not published yet; email keeps that between
us. A problem with the example project or its workflow is equally welcome there.

## Licence

This repository — the example project, the workflows, and this README — is **MIT**. Copy it freely.

**The Manuscrio engine image it runs is proprietary software.** MIT covers the glue in this
repository and nothing else.

---

[manuscrio.com](https://manuscrio.com) · [`manuscrio` on npm](https://www.npmjs.com/package/manuscrio) · [Docusaurus](https://github.com/manuscrio/docusaurus-pdf-export) · [Starlight](https://github.com/manuscrio/starlight-pdf-export)
