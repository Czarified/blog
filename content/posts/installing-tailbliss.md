+++
date = '2026-07-18T14:05:00-04:00'
draft = false
title = 'Installing the Tailbliss Theme with Tailwind CSS v4'
+++

In this post, we describe the process of migrating our newly initialized Hugo blog from the default **Ananke** theme to the modern, high-performance **Tailbliss** theme. Tailbliss uses **Tailwind CSS 4** and **Alpine.js** to deliver a responsive, clean, and highly customizable blogging interface.

Here is the step-by-step process we followed to perform this migration:

## 1. Removing the Ananke Theme
First, we cleaned up the old theme to prevent any configuration conflicts. Since the Ananke theme had been referenced as a submodule config but wasn't tracked properly:
- We removed the `themes/ananke` directory.
- We staged the deletion of the existing `.gitmodules` file and committed it to ensure a clean slate.
- We cleaned up Git's internal submodule directory cache.

```powershell
Remove-Item -Recurse -Force themes/ananke
git rm .gitmodules
git commit -m "Remove old ananke submodule configuration"
```

## 2. Adding the Tailbliss Submodule
Next, we cloned the Tailbliss repository directly into our themes directory as a Git submodule:

```powershell
git submodule add https://github.com/nusserstudios/tailbliss.git themes/tailbliss
```

## 3. Installing Dependencies & Building CSS
Tailbliss relies on modern frontend tooling like **Vite** and **Tailwind CSS v4** to compile utility styles. We navigated into the theme directory, installed the development dependencies, and executed the build script:

```powershell
cd themes/tailbliss
pnpm install
pnpm run build:css:dev
```

This built the main stylesheet (e.g., `main.mrqoho8d.css`) under `themes/tailbliss/static/css/`.

## 4. Copying Built CSS to the Project Root
The theme layouts look for a pre-compiled CSS file in the project's root `static/css` folder to bypass the need for configuring PostCSS pipelines globally. We created the `static/css` directory at our project's root and copied the compiled stylesheet into it:

```powershell
New-Item -ItemType Directory -Force static/css
Copy-Item themes/tailbliss/static/css/main.*.css static/css/
```

## 5. Updating the Blog Configuration
We updated `hugo.toml` to change the active theme, specify metadata parameters, and control theme parameters:

```toml
baseURL = 'https://example.org/'
locale = 'en-us'
title = 'My New Hugo Project'
theme = 'tailbliss'

[params]
  author = 'Antigravity'
  description = 'A Hugo blog built with Tailwind CSS 4 and Alpine.js.'
  disable_theme_toggle = false
  disable_stay_uptodate = true
```

## 6. Removing Example Content
Finally, we removed the default `my-first-post.md` created during the initial setup to ensure a clean publication history:

```powershell
Remove-Item content/posts/my-first-post.md
```

With these steps complete, our Hugo static site is now running on **Tailbliss** with compiled **Tailwind CSS v4** assets!
