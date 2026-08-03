# Memory

Durable facts, decisions, and landmines for Default Deny (packetsherpa.org).
One file per fact. Live/churning state lives in `project-context.md`, not here.

- [Split out of steady.org](split-from-steady-org.md) — this repo took the technology writing on 2026-08-02; music stays at steady.org, and git history before the split is shared between the two repos.
- [Posts publish at the site root](posts-publish-at-site-root.md) — `content/technology/` maps to `/:contentbasename/`; `/technology/` is the Archive listing.
- [Images are optimized at build time](images-are-optimized-at-build-time.md) — commit full-resolution originals; PaperMod and a render hook produce WebP srcsets.
- [Site deploys from main via GitHub Pages](site-deploys-from-main.md) — production deploys come from pushes to `main` through the Pages workflow.
- [Site renders via the PaperMod theme submodule](site-renders-via-papermod-submodule.md) — layouts come from `themes/PaperMod`, vendored as a git submodule; fetch it with `git submodule update --init --recursive`.
- [damiendeville.com is an auto-synced mirror](damiendeville-mirror.md) — a second repo serves this same site at `damiendeville.com`; it differs only in `.github/` and anything else committed there is discarded.
- [Canonical domain is one param](canonical-domain-override.md) — `params.canonicalBaseURL` plus a local `head.html` copy decides the canonical origin for both sites; currently damiendeville.com.
- [Markdown source uses straight quotes](source-uses-straight-quotes.md) — Goldmark's typographer converts them to curly at build time, so never hand-type curly characters into content.
