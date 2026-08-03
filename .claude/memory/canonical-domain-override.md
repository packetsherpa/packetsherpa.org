---
name: canonical-domain-override
description: A local head.html copy plus params.canonicalBaseURL decides which domain is canonical for both packetsherpa.org and the damiendeville.com mirror.
metadata:
  type: project
---

`layouts/_partials/head.html` is a **verbatim copy** of
`themes/PaperMod/layouts/_partials/head.html` with one block changed: the
`rel="canonical"` line. It honors a site param, `params.canonicalBaseURL` in
`hugo.toml`, and falls back to `.Permalink` when the param is unset.

Because the same `hugo.toml` builds both this site and the
[[damiendeville-mirror]], that one param decides the canonical origin for both.
Flipping it is a one-line change; nothing else needs to move.

As of 2026-08-03 it points at `https://damiendeville.com/` because some DNS
filtering services block `packetsherpa.org` for being a newly registered domain.
Damien's plan is to flip it back to `https://packetsherpa.org/` roughly 4–6
weeks later (target mid-September 2026), after confirming the blocking resolvers
have stopped filtering it.

Two landmines:

- The canonical is built by string concatenation, **not** `urls.JoinPath`.
  `urls.JoinPath` uses `path.Join` semantics and strips the trailing slash, so it
  would emit `https://example.com/my-note` while Hugo actually serves
  `/my-note/` — a canonical that does not match the real URL.
- If the PaperMod submodule is bumped, this file goes stale silently. Re-copy the
  theme partial and re-apply the canonical block. See
  [[site-renders-via-papermod-submodule]].
