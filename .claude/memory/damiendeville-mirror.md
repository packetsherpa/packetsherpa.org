---
name: damiendeville-mirror
description: damiendeville.com is an auto-synced mirror repo serving this same site; write here, never there.
metadata:
  type: project
---

`packetsherpa/damiendeville.com` (local clone at
`~/Documents/GitHub/damiendeville.com`) serves this identical site at
`https://damiendeville.com/`. It exists because `packetsherpa.org` was
registered in late July 2026 and some DNS filtering services block newly
registered domains, so a share of readers could not resolve it.

**This repo is the source of truth.** The mirror's tree differs from it by
exactly one path, `.github/`. Its `.github/workflows/pages.yml` runs hourly and
on manual dispatch: it replaces its whole tree with this repo's, restores only
its own `.github/`, commits, pushes, then builds with
`--baseURL https://damiendeville.com/` and deploys. Anything committed to the
mirror outside `.github/` is discarded at the next sync.

Design decisions worth not relitigating:

- Sync, build, and deploy live in **one** workflow because a push made with
  `GITHUB_TOKEN` does not trigger further workflow runs — a separate build
  workflow listening on `push` would never fire after the sync commit.
- The sync **drops** upstream's `.github/` rather than merging it. `GITHUB_TOKEN`
  cannot push new workflow files, so a workflow added here would leak into the
  mirror and break every subsequent sync.
- The mirror's build overwrites `static/CNAME` (which says `packetsherpa.org`)
  before running Hugo. Shipping upstream's CNAME in the artifact would make Pages
  reassign the mirror's custom domain to a domain this repo already owns.
- `--baseURL` is passed explicitly, not taken from `configure-pages`, for the
  same http:// reason documented in this repo's own workflow.

Which domain is canonical is **not** decided here — see
[[canonical-domain-override]].

Known gap as of 2026-08-03: the Pages certificate covers the apex only, so
`https://www.damiendeville.com/` fails TLS. `www` is a `CNAME` to
`packetsherpa.github.io` and GitHub usually adds it on a later revalidation.
