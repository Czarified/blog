# Bloggy McBlogFace

Personal blog of Benjamin Crews, published at <https://czarified.github.io/blog/>.

This README explains how the site works and the two everyday jobs: **changing the home page** and **publishing a post**.

## How the pieces fit together

If you know Python, think of this as a build script that renders Jinja-style templates with Markdown content:

| Piece | What it is | Do you touch it? |
| --- | --- | --- |
| **Hugo** | The program that turns `content/**/*.md` + `hugo.toml` into a folder of plain HTML (`public/`). Like `pandoc` + Jinja in one executable. | You run it. |
| **`hugo.toml`** | Site settings and all the home page text and image paths. | Yes, often. |
| **`content/posts/`** | Your blog posts, written in Markdown. | Yes, this is where you write. |
| **TailBliss theme** (`themes/tailbliss/`) | The templates and styling. It's a git submodule (a copy of someone else's repo). | **No.** Override it instead (see below). |
| **Tailwind CSS** | The CSS framework the theme is styled with. It's already compiled into `static/css/main.mrqoho8d.css`. | **No.** You never need Node or npm locally. |
| **GitHub Actions** (`.github/workflows/deploy.yaml`) | On every push to `main`, it runs Hugo in the cloud and publishes the result to GitHub Pages. | No. |

**Overrides:** any file in the repo's own `layouts/` folder replaces the theme file at the same path. The site currently overrides these files:

| File | Why |
| --- | --- |
| `layouts/index.html` | Hero image comes from `hugo.toml`; images are cropped instead of stretched. |
| `layouts/partials/head.html` | Preloads your hero image; loads the math font CSS on pages with math. |
| `layouts/partials/post-tile.html`, `layouts/_default/single.html` | `featured_image` can be a file in the post's own folder. |
| `layouts/_markup/render-passthrough.html` | Renders `$...$` math with KaTeX. |

## One-time setup (Windows)

```bash
winget install Hugo.Hugo.Extended
```

```bash
git submodule update --init --recursive
```

## Preview locally

```bash
hugo server -D
```

Open <http://localhost:1313/blog/>. The page reloads automatically every time you save a file. `-D` includes draft posts, which never appear on the live site. Press `Ctrl+C` to stop.

## A) Changing the home page

Everything on the home page is set in `hugo.toml`. From top to bottom:

| Section | Keys in `hugo.toml` | Image is cropped to |
| --- | --- | --- |
| Hero (title, tagline, big photo) | `title`, `params.moto`, `params.description`, `params.heroimage`, `params.heroalt` | 576 × 576 square |
| About This Site | `[params.p1]`: `heading`, `image`, `content` | 600 × 600 square |
| What I'm Working On | `[params.p2]`: `heading`, `image`, `content` | 800 × 600 |
| Third section | `[params.p3]`: `heading`, `content`, `links`, `logos` | (logo grid, see below) |
| Latest posts | `[params.p4]`: `heading` (shows the 3 newest posts automatically) | — |

### Changing a home page image

1. Put the image file in **`assets/images/`**. It can be any size and shape (JPG, PNG or WebP).
2. Set the path in `hugo.toml`, relative to `assets/` and **without a leading slash**:

   ```toml
   heroimage = 'images/global/hero.jpg'
   ```

3. Save. `hugo server` shows the change immediately.

Hugo resizes and crops the image automatically. The crop keeps the most interesting part of the image (`smart` anchor), so a portrait photo works fine in the square hero.

**Why `assets/` and not `static/`?** Files in `static/` are copied to the site byte-for-byte. Files in `assets/` can be resized, cropped and converted to WebP by Hugo. The home page templates only look in `assets/`, which is why images in `static/` never showed up there.

### The third section's logo grid

The theme's third section was designed as a "sponsors" grid. `logos = []` hides the grid. To show logos, list them:

```toml
logos = [
  { image = 'images/global/sample-logo.svg', alt = 'Some company' },
]
```

### The navigation menu

`[[menu.main]]` entries are the top bar; `[[menu.footer]]` entries are the footer. Write menu URLs **without** a leading slash (`url = 'posts/'`). A leading slash would point to `czarified.github.io/posts/`, outside the blog.

## B) Writing and publishing a post

### 1. Create the post

```bash
hugo new content posts/my-post-title/index.md
```

This creates a **folder** for the post with an `index.md` inside it. Put the post's images in the same folder. The folder name becomes the URL: `.../blog/posts/my-post-title/`.

### 2. Fill in the front matter

The block between the `+++` lines at the top of `index.md`:

```toml
+++
date = '2026-10-24T09:00:00-04:00'
draft = true                       # change to false when ready to publish
title = 'My Post Title'
summary = 'One or two sentences shown on the post card.'
tags = ['python', 'structures']
featured_image = 'cover.jpg'       # optional; a file in this folder
+++
```

### 3. Write in Markdown

[`content/posts/example-post/index.md`](content/posts/example-post/index.md) is a draft showing every feature. Run `hugo server -D` and open it to see the result next to the source. In short:

**Images.** Put the file next to `index.md`. No folders, no leading slash:

```markdown
![Alt text describing the image](fbd.png)
```

**Tables.** Standard Markdown tables:

```markdown
| Material   | E (GPa) | Yield (MPa) |
| :--------- | ------: | ----------: |
| Al 7075-T6 |    71.7 |         503 |
```

**Math.** LaTeX, like in Jupyter:

```markdown
Inline: $\sigma = \frac{M y}{I}$

$$
\delta_{max} = \frac{P L^3}{48 E I}
$$
```

Math is rendered to HTML when the site is built (by KaTeX), so it appears instantly with no JavaScript. KaTeX supports most of the LaTeX math commands you'd use in a notebook (`aligned`, `matrix`, `cases`, Greek letters, etc.); see [the KaTeX support list](https://katex.org/docs/supported). A plain dollar sign must be written `\$`. If an equation has a typo, Hugo stops with an error naming the file and line.

**Code.** Fenced blocks with a language name get syntax highlighting:

````markdown
```python
print("hello")
```
````

### Starting from a Jupyter notebook

```bash
jupyter nbconvert --to markdown analysis.ipynb
```

This writes `analysis.md` plus an `analysis_files/` folder of plot images. Move both into the post's folder, rename `analysis.md` to `index.md`, and add the front matter block at the top. The image links already point into `analysis_files/`, which works inside a post folder, and notebook math uses the same `$` syntax.

### 4. Publish

1. Set `draft = false`.
2. Check that `date` isn't in the future. Future-dated posts are hidden until that date.
3. Commit and push to `main`:

   ```bash
   git add -A
   ```

   ```bash
   git commit -m "New post: My Post Title"
   ```

   ```bash
   git push
   ```

4. Watch the **Actions** tab on GitHub. The site updates about a minute after the job turns green.

## Where do image files go?

| The image is used in... | Put it in | Refer to it as |
| --- | --- | --- |
| A post | The post's folder (`content/posts/my-post/`) | `fbd.png` |
| The home page (`hugo.toml`) | `assets/images/...` | `images/global/hero.jpg` (no leading slash) |
| Somewhere that needs an untouched file (e.g. a PDF download) | `static/...` | `/files/report.pdf` (leading slash) |

## Things not to change

- `canonifyURLs = true` and `relativeURLs = false` in `hugo.toml`. The site lives under `/blog/`, and with `relativeURLs` every theme image pointed at `/blog/blog/...` and broke.
- The `[module]` block. It loads the theme without the theme's demo content; without it the theme's sample posts reappear.
- Files under `themes/tailbliss/`. Copy a file into the root `layouts/` folder and edit the copy instead. Overridden files don't pick up future theme updates, so keep overrides few.
