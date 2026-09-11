# andamento.dev — website

Placeholder site for [Andamento](https://github.com/andamento-dev), built with
[Hugo](https://gohugo.io) (no theme — the whole site is one layout).

## Requirements

- Hugo **extended** `0.163.3` (the version pinned in the workflows; `brew install hugo`)

## Local development

```sh
hugo server
```

Then open <http://localhost:1313>.

To produce a production build in `public/`:

```sh
hugo --gc --minify
```

## Layout of the repo

| Path | Purpose |
| --- | --- |
| `hugo.toml` | Site config, GitHub URL and the brand colours |
| `layouts/index.html` | The entire page |
| `assets/css/main.css` | Styles — run through Hugo's template engine, then minified + fingerprinted |
| `assets/fonts/` | Self-hosted Spectral woff2 + its licence |
| `assets/images/andamento-logo.png` | Logo used by the site — transparent, cropped to the artwork |
| `assets/images/og-image.png` | 1200×630 social sharing card |
| `andamento-logo.png` | Original logo as supplied, on a white 400×400 canvas |
| `.github/workflows/ci.yaml` | Build check on every branch and PR |
| `.github/workflows/deploy.yaml` | Builds and deploys `main` to GitHub Pages |

## Typography

The wordmark is **Spectral** (weight 500) by Production Type, licensed under the
[SIL Open Font License 1.1](assets/fonts/OFL.txt).

It is self-hosted rather than loaded from Google Fonts, so no request leaves the
page to a third party. Only the latin subset ships (15 KB). `main.css` is passed
through `resources.ExecuteAsTemplate` so the `@font-face` URL picks up Hugo's
content hash, and the font is preloaded in `<head>` to avoid a flash of fallback
text.

To swap the font, replace the woff2 in `assets/fonts/`, update the `@font-face`
block and `--font-serif` at the top of `main.css`, and re-check the
`.wordmark` size and letter-spacing — serifs and sans-serifs need different
tracking at the same size.

## Brand colours

Sampled from the logo:

| Colour | Hex | Used for |
| --- | --- | --- |
| Deep green | `#003B28` | Wordmark |
| Mid green | `#2A6A52` | GitHub link, background halo |
| Light green | `#468868` | Focus ring |

## Deployment

Pushing to `main` runs `.github/workflows/deploy.yaml`, which builds the site and
publishes it to GitHub Pages at **https://andamento.dev**.

`baseURL` is set once in `hugo.toml`; the workflow deliberately does *not*
override it from the Pages config, so the canonical URL, sitemap and social card
always point at the custom domain.

### One-time setup

1. **Settings → Pages → Source** = **GitHub Actions**.
2. **Settings → Pages → Custom domain** = `andamento.dev`.
3. Tick **Enforce HTTPS** once the certificate is issued.

`static/CNAME` is published to the site root on every deploy, which stops the
custom domain being reset.

### DNS

For the apex domain, four `A` records and four `AAAA` records:

| Type | Name | Value |
| --- | --- | --- |
| A | `@` | `185.199.108.153` |
| A | `@` | `185.199.109.153` |
| A | `@` | `185.199.110.153` |
| A | `@` | `185.199.111.153` |
| AAAA | `@` | `2606:50c0:8000::153` |
| AAAA | `@` | `2606:50c0:8001::153` |
| AAAA | `@` | `2606:50c0:8002::153` |
| AAAA | `@` | `2606:50c0:8003::153` |
| CNAME | `www` | `andamento-dev.github.io.` |

If your DNS provider supports `ALIAS`/`ANAME` at the apex, a single record
pointing at `andamento-dev.github.io` works instead of the eight above.

> **`.dev` is on the HSTS preload list**, so browsers refuse plain HTTP for it
> entirely — there is no insecure fallback to land on. The site will look broken
> until GitHub has issued the certificate and **Enforce HTTPS** is on. That
> usually takes a few minutes after DNS propagates, but can take up to 24 hours.

Check propagation with:

```sh
dig +short andamento.dev
curl -sSI https://andamento.dev | head -1
```
