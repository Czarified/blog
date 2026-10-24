# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Personal Hugo blog ("Bloggy McBlogFace") using the **TailBliss** theme (Tailwind CSS v4), deployed to GitHub Pages at `https://czarified.github.io/blog/`. There is no application code or test suite; the work is content, `hugo.toml` config, and theme overrides. `README.md` is the owner's workflow guide (home page edits, writing posts). Keep it accurate when changing how things work. The owner is a Python developer with little web/Node experience, so prefer config-only solutions over new tooling.

## Commands

```bash
git submodule update --init --recursive   # theme lives in themes/tailbliss (git submodule)
hugo server -D                            # dev server at http://localhost:1313/blog/, includes drafts
hugo new content posts/my-post/index.md   # new post as a page bundle (archetypes/default.md, draft = true)
hugo --gc --minify                        # production build into public/ (a successful build is the only "test")
```

Requires **Hugo Extended**. Node/npm are not needed locally (see CSS below); CI still runs `npm ci`.

## Deployment

`.github/workflows/deploy.yaml` builds on every push to `main` with the latest Hugo Extended (`--baseURL` overridden from the Pages config) and deploys `public/` to GitHub Pages. Pushing to `main` publishes the site.

## Architecture notes

- **Subpath URLs:** the site lives under `/blog/`, so `hugo.toml` uses `canonifyURLs = true`, `relativeURLs = false`. With `relativeURLs` on, Hugo-processed image URLs became `./blog/...` and 404'd. Menu `url`s must have **no leading slash** (`posts/`), because the theme pipes them through `absLangURL` and `/posts/` resolves to the host root.
- **Theme loaded via `[module]` imports, not `theme =`:** it mounts every theme folder except `content`, so the theme's demo posts and pages stay out of the site. The theme's `hugo.yaml` params and menus still deep-merge into the site config. That's why `hugo.toml` explicitly sets `newsletter_signup = false`, `social_media.*`, `p3.logos = []`, `p4`, and `menu.footer`.
- **Home page** content comes from `params.heroimage`, `heroalt`, `p1`–`p4` in `hugo.toml`. Those templates use `resources.Get`, so image paths are relative to **`assets/`** with no leading slash.
- **Project overrides** in `layouts/` (copies of theme files, edited and marked with `Override:` comments): `index.html` (hero from params, `Fill` instead of `Resize`), `partials/head.html` (hero preload, KaTeX CSS, small inline CSS), `partials/post-tile.html` and `_default/single.html` (`featured_image` resolves against the page bundle first, then `assets/`).
- **Posts** are page bundles (`content/posts/<slug>/index.md` + images beside it). Markdown images use bare file names. TOML front matter: `title`, `date`, `draft`, `summary` (card text), `tags`, `featured_image`. Future-dated posts aren't built. `content/posts/example-post/` is a permanent draft demonstrating images, tables, and math.
- **Math:** a Goldmark passthrough (`$…$`, `$$…$$`, `\(…\)`, `\[…\]`) sends math to `layouts/_markup/render-passthrough.html`, which renders it at build time with `transform.ToMath` (KaTeX) and sets `.Page.Store "hasMath"`. `head.html` evaluates `.Content` before checking that flag, since the head renders before the body.
- **CSS:** `themes/tailbliss/layouts/partials/head.html` links any `static/css/main.*.css` directly and skips the PostCSS/Tailwind pipeline. The committed `static/css/main.mrqoho8d.css` is a pre-built full theme stylesheet, so Tailwind classes that aren't already in it won't work. Put small custom CSS in the `<style>` block in `layouts/partials/head.html`.
