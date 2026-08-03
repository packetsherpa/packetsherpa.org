# Writing and publishing

How to write a post and put it live on https://packetsherpa.org/ (the site is
named **Default Deny**). It is [Hugo](https://gohugo.io/) + the PaperMod theme,
deployed to GitHub Pages. Publishing is nothing more than pushing Markdown to
`main`.

Music and personal writing go to the separate `steady.org` repo, not this one.

## One-time setup (on a computer)

```sh
git clone https://github.com/packetsherpa/packetsherpa.org.git
cd packetsherpa.org
git submodule update --init --recursive   # pulls the PaperMod theme
```

Install Hugo (`brew install hugo` on macOS). No Node, no Python — just Hugo.

## The everyday loop

1. **Create the post.**

   ```sh
   hugo new content technology/my-note.md
   ```

   This scaffolds the file with `draft: true`. The `technology` section matches
   its archetype (`archetypes/technology.md`) by name automatically.

2. **Write it.** Edit the front matter and body in Markdown. While
   `draft: true`, it is invisible to the world.

3. **Preview.**

   ```sh
   hugo server -D
   ```

   Open http://localhost:1313/ (`-D` shows drafts; it live-reloads on save).

4. **Publish.** Set `draft: false`, then:

   ```sh
   git add .
   git commit -m "Publish: my note title"
   git push          # to main
   ```

   Pushing to `main` triggers the GitHub Action, which builds and deploys in
   about half a minute. **Pushing to `main` is publishing** — there is no
   separate deploy step.

## Where posts end up

Posts live in `content/technology/`, but they publish at the site root.
`hugo.toml` maps the section with `[permalinks] technology = "/:contentbasename/"`,
so the **bundle folder name is the URL**:

```
content/technology/my-note/index.md   →   https://packetsherpa.org/my-note/
```

The `/technology/` URL is the **Archive** page — the full list, newest first.
The home page lists posts too, so most readers never hit the Archive directly.

## Text-only post vs. post with a header image

A **text-only** post is a single `.md` file.

For a **header image**, make the post a *page bundle* — a folder with
`index.md` — and put the image inside it:

```
content/technology/my-note/
├── index.md
└── feature.jpg
```

Then in the front matter:

```yaml
cover:
  image: "feature.jpg"
  alt: "Describe what is visible in the image."
  relative: true          # required — see gotchas
```

For an image **inside the body**, put it in the same folder and reference it
inline:

```markdown
![Alt text describing the photo](photo.jpg)
```

Working example to copy: `content/technology/it-wasnt-air-gapped/`.

### Image sizes

Commit the full-resolution file. Both paths downscale and convert to WebP at
build time:

- **Header images** — PaperMod does it, because `cover.responsiveImages = true`
  is set in `hugo.toml`. It only runs when `params.env` is `production`, which
  is the committed setting, so `hugo server` skips it for speed.
- **In-body images** — `layouts/_markup/render-image.html` handles it. It emits
  a `srcset` at 480/720/1080px plus a 1440px cap, all WebP at q82, with
  `loading="lazy"`. Formats Hugo cannot process (SVG, GIF) and remote URLs fall
  through to a plain `<img>` untouched.

Generated variants land in `resources/_gen/`, which is git-ignored — CI
regenerates them on every build.

## Front matter reference

```yaml
---
title: "Your Title"
date: 2026-08-01
draft: true            # flip to false to publish
description: "One-sentence summary; shows on cards and link previews."
categories:
  - Technology
tags:
  - security
  - ai
cover:                 # optional; only for bundles that include an image
  image: "feature.jpg"
  alt: ""
  relative: true
---
```

Keep to these fields — standard front matter is what keeps the site portable
and makes a future theme swap a one-line change.

### Starting a post in iA Writer or MiniSeries

`templates/iawriter-post.md` is a ready-to-fill starter: the front matter above
plus a scaffolded body, with no `hugo new` step. Both iA Writer (iOS) and
MiniSeries (macOS) edit plain Markdown, so keep a copy of it in your synced
writing folder (iCloud or Dropbox) and duplicate it for each new post. Rename
the copy to the URL you want, fill in the front matter, and write. When you are
ready, move it into `content/technology/` and push (see below).

## Gotchas

- **`cover ... relative: true`** is required for a bundle's header image, or the
  social-share preview link 404s. On the page itself it looks fine either way,
  so it is easy to miss.
- **Do not run `hugo --panicOnWarning`.** The current PaperMod release triggers
  two harmless Hugo deprecation warnings that would fail the build under that
  flag. Use `hugo server -D` and `hugo --gc --minify` (what CI runs).
- **Write straight quotes and apostrophes in Markdown source.** Hugo's Goldmark
  typographer converts them to curly at build time. Hand-typing curly characters
  produces identical output while making the source inconsistent.
- The About page and the Archive intro are Markdown too: `content/about.md` and
  `content/technology/_index.md`.

## Publishing from a phone or tablet

The site is just Markdown files in a git repo, so anything that can edit a file
and push to `main` can publish. Ranked by how well each handles a real post
(including images):

1. **Working Copy (iOS/iPadOS) — best.** A full git client that clones the repo,
   pulls the theme submodule, creates folders (page bundles), adds photos from
   your camera roll, commits, and pushes to `main`. Pair it with iA Writer or
   its built-in editor (iA Writer opens the repo through the Files app). This is
   the only mobile path that cleanly handles a header image + bundle.

2. **GitHub in a browser or the GitHub mobile app — quickest for text.**
   Go to `content/technology/`, "Add file → Create new file", name it
   `my-note.md`, paste front matter + body, commit to `main`. Live in ~30s. To
   include an image without a computer, use "Add file → Upload files" and type a
   path like `content/technology/my-note/index.md` in the filename to create the
   folder, then upload the image into the same folder. Fiddlier than Working
   Copy, but it works.

**Android:** same idea — a git client like Termux + git or MGit, plus any
Markdown editor. The GitHub web flow is identical.

**Previewing on mobile is the weak spot.** There is no easy local Hugo on a
phone. Two realistic options:

- Lean on drafts: keep `draft: true`, flip to `false` only when you are
  confident, and review the live post right after it deploys.
- Or set up deploy previews (Cloudflare Pages / Netlify) so pushing a branch
  gives a preview URL you can open on your phone before merging. Not configured
  today, but a clean add if you want it.

The same gotchas apply on mobile: keep `cover ... relative: true` for header
images, set `draft: false` to publish, and remember that pushing to `main`
publishes.
