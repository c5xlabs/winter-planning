# Documentation site engineering

Engineering for `epic:documentation`.

## Brief requirements

- engineering:docs-repo — The site lives in its own repository, `blizzard-docs`, built with Astro and Starlight and
  deployed to GitHub Pages on push. Serves `product:docs-site`.
- engineering:llms-txt — The site publishes its single-file form through `starlight-llms-txt`. Serves
  `product:agent-readable-site`.
