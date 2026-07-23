# TechCirkle Blog

A Jekyll-powered blog about software engineering and applied AI, syndicated across multiple platforms.

## What is this?

This is the GitHub Pages home for TechCirkle's long-form content — technical guides, case studies, and engineering perspectives on building software products in the AI era. Posts are published daily and syndicated to Dev.to, Hashnode, Medium, Substack, and other platforms for maximum reach.

## How posts are published

1. **Daily automation** — a pipeline drafts 2 original posts per day targeting commercial keywords from TechCirkle's market gap analysis.
2. **Multi-platform syndication** — each post gets 10 distinct rewrites (one per platform) with platform-tailored framing, all stored in `~/Documents/TechCirkle Blog/[Topic]/[Platform]/`.
3. **Auto-publish targets** — Dev.to, Blogger, WordPress.com, Mataroa, LinkedIn, and this GitHub Pages site publish live automatically.
4. **Manual platforms** — Medium, Substack, DZone, and Hashnode are copy-pasted manually for fine-grained editorial control.

## Technical stack

- **Blogging platform**: Jekyll (Minima theme)
- **Syndication**: Custom Node.js publishers (publish-to-*.mjs) that convert Markdown → platform-native formats
- **Image hosting**: Sanity CDN (all posts embed images via public URLs, no local asset burden)
- **SEO**: No canonical links per post (each variant ranks independently); articles carry structured metadata (title, meta description, keywords, FAQ schema)

## Repository structure

```
├── _posts/              # Published posts (Jekyll-managed, auto-generated)
├── _drafts/             # Draft posts (not built by Jekyll)
├── _config.yml          # Jekyll config (Minima theme, future: true, clean permalink)
├── index.md             # Homepage
└── README.md            # This file
```

## Writing for this blog

**Editorial guidelines:**
- **Word count**: 3,000+ words (includes FAQ sections)
- **Structure**: 12–18 sections with H2 headings, at least one H3 subsection
- **Images**: 2 required (hero + one alt image, 1600px for web)
- **FAQ**: 5–10 Q&A pairs at the end (helps with long-tail search + AI-Overview citability)
- **Internal links**: 1–2 natural references to TechCirkle service pages (no hard sells)
- **Voice**: Senior buyer audience (CTO, founder, VP Eng) — practical, opinionated, evidence-based
- **AI angle**: Every post surfaces how AI changed the problem/solution/cost structure (not a tack-on)

## Republishing existing posts

Posts automatically publish to GitHub Pages via:
```bash
node scripts/blog-pipeline/publish-to-github-pages.mjs \
  "~/Documents/TechCirkle Blog/<Topic>/<Platform>/article.md" \
  --publish
```

The script:
1. Parses the Markdown front matter
2. Hosts local hero.jpg/alt.jpg on the Sanity CDN
3. Adds Jekyll front matter (`layout: post`, date, categories, tags)
4. Commits to `_posts/<YYYY-MM-DD>-<slug>.md` via the GitHub API
5. Jekyll rebuilds automatically (~30–60s) and the post goes live

## Live example

[The $40k Line That Didn't Exist Two Years Ago](https://techcirkle-ctrl.github.io/techcirkle/2026/07/23/the-40k-line-that-didnt-exist-two-years-ago/) — an MVP development guide with distinct angles for each platform, all published from one source.

## Contact & contribution

Questions or feedback? Open an issue in the main [TechCirkle repository](https://github.com/techcirkle-ctrl/techcirkle-v2).
