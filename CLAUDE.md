# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A personal blog/portfolio site (https://hithisisyeshwanth.github.io) built with **Hugo** and the **PaperMod** theme. PaperMod is imported as a Hugo module. GitHub Actions (`.github/workflows/hugo.yml`) builds and deploys the site to GitHub Pages on every push to `master`. There is no test suite.

The site used to be Jekyll (Minimal Mistakes). The last Jekyll version is kept on the `jekyll-stable` branch and tagged `jekyll-final` (`19b78cf`).

## Deployment and rollback

Pages is set to `build_type: workflow`. Only `master` and `jekyll-stable` may deploy to the `github-pages` environment. A failed workflow run leaves the previous deployment live.

To put the old Jekyll site back (takes about 2 minutes, with no git history changes):

```bash
gh api -X PUT repos/hithisisyeshwanth/hithisisyeshwanth.github.io/pages \
  -f build_type=legacy -f "source[branch]=jekyll-stable" -f "source[path]=/"
gh workflow disable "Deploy Hugo site to Pages"
```

To undo the rollback, run `gh api -X PUT .../pages -f build_type=workflow`, then `gh workflow enable "Deploy Hugo site to Pages"`, then re-run the latest deploy.

## Commands

```bash
hugo server -D        # dev server at http://localhost:1313 with live reload; -D includes drafts
hugo --minify         # production build into public/ (what CI runs)
hugo mod get -u       # update PaperMod to its latest commit (updates go.mod/go.sum)
```

Hugo modules need Go installed. Keep the local Hugo version equal to `HUGO_VERSION` in the workflow (currently 0.167.0, extended).

## Architecture

- **The theme is a module, not vendored.** PaperMod's layouts are not in this repo; they live in Hugo's module cache (`hugo config mounts` prints the path). To change a theme template, copy it to the same path under `layouts/`, which overrides it. Hugo ≥0.146 uses the new template layout: `layouts/_partials/`, `layouts/_markup/`, `layouts/_shortcodes/`, and top-level `single.html`/`list.html`.
- **Local overrides:**
  - `layouts/home.html` replaces PaperMod's home page. It shows the intro (reusing the theme's `index_profile.html` partial), then featured projects, then latest posts, then a call to action. Projects marked `featured: true` show first; if none are, the newest 3 show. Each card is drawn by `layouts/_partials/home_entry.html`, which uses the theme's `post-entry` markup.
  - `layouts/_markup/render-link.html` makes every external http(s) Markdown link open in a new tab.
  - `assets/css/extended/custom.css` is loaded automatically after the theme CSS. It holds the square profile photo, the menu wrapping, and the styles for the home sections and call to action.
- **Config** lives entirely in `hugo.toml`: the menu (Writing · Projects · About · Search), the home intro (`params.profileMode`: headline and supporting line), the call to action (`params.cta`), the footer links to the personal pages (`params.footer.text`), social icons and theme flags.
- **Site structure:** the portfolio (`posts`, `projects`) is in the menu and on the home page. The personal sections (`photography`, `motorcycles`) are linked only from the footer. Their `_index.md` cascades `hiddenInRss: true`, so their posts stay out of the feed. `interview-prep.md` is `draft: true` until it's ready to publish.
- **New projects:** `hugo new projects/<slug>.md` uses `archetypes/projects.md`, a draft with the sections The paper, Implementation, Results (Paper vs Mine table), What I learned and Links.
- **Content** lives in `content/`:
  - Posts go in `content/posts/YYYY-MM-DD-slug.md`. The filename sets the date and slug (`[frontmatter] date = [":filename", …]`), and the URL is `/posts/:slug/`.
  - `content/projects/` is a section. `_index.md` is the `/projects/` list page.
  - Standalone pages set `url:` in front matter.
  - `archives.md` and `search.md` exist only to switch on PaperMod's archive and search layouts. Search reads `/index.json`, which comes from `[outputs] home`.
  - `categories/_index.md` and `tags/_index.md` only set the titles of the taxonomy pages that Hugo generates automatically.
- **Images** that the theme processes, like the profile photo, go in `assets/images/` and are referenced as `images/...`. Files that should be served unchanged go in `static/`.
- **URL compatibility with the old Jekyll site:** `aliases:` in front matter redirect retired URLs (for example, `/hire-me/` redirects to About), and `[outputFormats.RSS] baseName = "feed"` keeps the feed at `/feed.xml`.
- **Markdown** is rendered by Goldmark. Kramdown-style attributes like `{: target="_blank"}` don't work, and raw HTML is stripped unless `markup.goldmark.renderer.unsafe` is enabled.

## Notes

`README.md` contains the owner's running to-do and bug list for the site. Check it for context on in-progress work.
