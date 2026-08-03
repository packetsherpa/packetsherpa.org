# Default Deny (packetsherpa.org) — Live State

> Live state only: what is in flight, blocked, and next. Durable facts live in
> `.claude/memory/` (see `.claude/memory/MEMORY.md`). Changelog lives in git
> history.

## Goal

Maintain and publish **Default Deny** (packetsherpa.org), Damien DeVille's
technology writing — security, artificial intelligence, complex systems, and
technical leadership — with a low-friction "write markdown locally, push to
`main`" workflow.

## Active Work

- This repo was split out of `steady.org` on 2026-08-02, cloned with full git
  history so the three existing technology notes keep their commits. `steady.org`
  keeps music and personal writing; this repo takes technology. See
  [[split-from-steady-org]].
- Carried over: three technology notes (`it-wasnt-air-gapped`,
  `shadow-ai-is-a-demand-signal-not-a-policy-failure`,
  `egress-filtering-is-the-control-we-never-implemented`), the about page, the
  PaperMod submodule, the Pages workflow, and the `ideas/` staging tree.
- Posts publish at the site root (`/:contentbasename/`, i.e. the bundle folder name) rather than `/technology/:slug/`;
  `/technology/` is now the Archive listing.
- Image optimization added: `cover.responsiveImages = true` plus a new in-body
  image render hook at `layouts/_markup/render-image.html`.

- Live at `https://packetsherpa.org/` with HTTPS enforced.
- Also live at `https://damiendeville.com/` from an auto-synced mirror repo
  (2026-08-03). Some DNS filtering services block `packetsherpa.org` for being a
  newly registered domain, so the established domain now serves the same site
  and is the canonical origin. See [[damiendeville-mirror]] and
  [[canonical-domain-override]].

## Blockers

- `steady.org` still contains the technology section. It has not been cut down
  yet — that is a separate change in that repo.

## Next

- **Around mid-September 2026** (4–6 weeks after 2026-08-03), check whether the
  resolvers that were blocking `packetsherpa.org` have stopped. If so, set
  `canonicalBaseURL = "https://packetsherpa.org/"` in `hugo.toml` — that single
  line flips canonical back for both sites. `damiendeville.com` keeps serving.
- `https://www.damiendeville.com/` fails TLS: the Pages certificate covers the
  apex only. GitHub usually adds `www` on a later revalidation; recheck, and if
  it has not resolved, re-set the custom domain in Settings → Pages.
- Cut `steady.org` down to music: delete `content/technology/`,
  `archetypes/technology.md`, `ideas/technology/`, fix its `hugo.toml`, and
  rewrite its about/home copy. No redirects — readership is negligible and
  Damien explicitly deferred them on 2026-08-02.
- Optional: add the same image render hook to `steady.org`.
- Write posts.
