# yoDEV Apex Bridge

> **v2 — YOD-671 (2026-10-02): signup-first.** The CTA no longer links to the homepage. It asks anonymous readers who finished an article to **create an account in place** (Discourse `/signup`, which is a yoDEV account), with a secondary "Ya tengo cuenta" link. Copy leads with community value (polls, asking other devs, weekly digest) and names the product second, per category bucket. Discourse drops `?ref=` through signup, so attribution is a Plausible custom event **`CTA Click`** (props `placement`, `bucket`, `target`) plus Discourse's own signup counts — add `CTA Click` as a goal in Plausible. Settings: `signup_path`, `login_path` (same-origin paths only; anything else is refused), `ref_value`, `enabled`. Signups had held at ~20/month (~0.2% of pageviews) for four months while v1 was never installed. The sections below describe v1 and remain accurate for gating, placement and SPA handling.

A Discourse theme component that shows **anonymous** readers a contextual link to the yoDEV homepage at the end of a topic.

## Why (YOD-479)

Search Console, 28 days to 2026-08-14 (`sc-domain:yodev.dev`):

| Surface | Organic clicks | Impressions | Indexable pages |
| --- | --- | --- | --- |
| `www.yodev.dev` (community) | **439** | ~118,700 | hundreds of topics |
| `yodev.dev` (apex) | **2** | ~300 | **1** |

The apex is technically healthy — indexed, prerendered, valid sitemap, correct canonical. It ranks for nothing because its sitemap contains one URL and nothing has ever linked to it. Meanwhile ~439 people a month arrive at the community from Google, read an article, and leave without learning the product exists.

This component bridges the two.

**It is deliberately NOT a redirect.** Post URLs must keep landing on the post: the ranking belongs to the article's content, and intercepting search traffic would forfeit it and read as cloaking to Google.

## How it works

Rendered at the end of a topic, immediately above the suggested-topics block — the reader has finished and is deciding what is next.

**Anonymous only.** Gated on the *presence* of the login control rather than the *absence* of `#current-user`: absence is ambiguous during Ember's first paint, and that ambiguity would show the CTA to logged-in members. Requiring a positive anonymous signal fails the safe way — when unsure, it renders nothing.

**Copy varies by category**, read from the `category-<slug>` class Discourse puts on `<body>`. Four buckets rather than one string per category — 23 categories would be 23 things to keep in sync, and the meaningful distinction is what the reader came for:

| Bucket | Categories | Angle |
| --- | --- | --- |
| `ai` | Claude Code, Cursor, Copilot, ChatGPT, Gemini, Windsurf, Amazon Q, Aider, AI Dev Tools, AI & Data Sci, AI Editors, Transición AI | Workplace's AI copilot + Claude Code integration |
| `career` | Trabajos/Carrera, Junior Devs | worX — finding freelance work |
| `dev` | Desarrollo, FrontEnd, BackEnd, Architecture, Cybersecurity, Code Swap | Workplace — real-time collaboration |
| `default` | everything else, including new categories | The platform generally |

**Copy is evergreen on purpose** — it does not mention the 90-day launch offer. The offer lives on the landing page the reader arrives at, and a component nobody is going to babysit should not carry a claim that expires.

**Attribution.** Every link carries `?ref=comm-topic-end`. YOD-478 captures that on landing and merges it into every GA event, so signups are attributable to this placement, not just clicks. The `ref` goes in the query string *before* any hash, because YOD-478 reads `window.location.search`.

Built as manual DOM injection rather than `api.renderInOutlet`: `<script type="text/discourse-plugin">` was found not to execute on this Discourse setup, which is why the beta-offer and activation-redirect components inject directly too. Settings reach the script through CSS custom properties written by `common.scss`.

## Settings

| Setting | Default | Description |
| --- | --- | --- |
| `apex_url` | `https://yodev.dev` | The homepage to link to — NOT the community host. Must be https. Blank disables the component. |
| `ref_value` | `comm-topic-end` | The `?ref=` value. Change only to report a placement separately. |
| `enabled` | `true` | Render the CTA. Turn off without uninstalling. |

## Installation

1. Discourse admin → **Customize → Components → Install** → from a git repository → this repo URL.
2. Add the component to the active theme.
3. Check the settings; the defaults are correct for production.

## Also worth doing

`exclude_rel_nofollow_domains` is currently **`www.yodev.dev` only**. Adding `yodev.dev` means links that *users* post to the apex pass authority instead of being nofollowed.

This does **not** affect this component's link: `add_rel_nofollow_to_user_content` applies to cooked post markdown, and theme markup never goes through that pipeline. Verified on the live forum — the rendered CTA carries no `rel` attribute.

## Verified before release (2026-08-20)

Against a live anonymous topic page on `www.yodev.dev`, Discourse 2026.8.0:

- Anonymous detection, topic detection, and `category-claude-code` → `ai` bucket all correct
- Insertion resolves to `.more-topics__container` (the primary target, not the fallback)
- Link built as `https://yodev.dev/?ref=comm-topic-end#workplace` — query before hash
- Rendered link carries no `rel` attribute
- Contrast measured, not eyeballed: title 12.62:1, body 5.42:1, CTA 4.70:1 — all pass WCAG AA

⚠️ The CTA uses `#fff` rather than `var(--secondary)`. Discourse's own button pattern is tertiary-on-secondary, which measured **4.24:1** on this theme — under the 4.5:1 threshold, and 16px semi-bold does not qualify as large text. White on the same accent measures 4.70:1. **This was measured against the current dark theme**; re-measure if the colour scheme changes.

## Measurement

GA on the apex will show `www.yodev.dev` as referrer, with `ref` splitting by placement. Expect a readable signal in 1–2 weeks.

Out of scope: raising the apex's *organic search* numbers. That needs the apex to have pages, and is separate work. This moves visits and signups only.

## Note on the splash

Anonymous community visitors also see the launch-offer splash from `discourse-yodev-beta-offer`. The two are complementary — the splash is an interstitial on arrival, this is contextual at the end of an article — but they do both target the same anonymous reader. Worth watching that the combination is not excessive.
