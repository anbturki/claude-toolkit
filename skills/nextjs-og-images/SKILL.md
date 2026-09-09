---
name: nextjs-og-images
description: Generate favicons, app icons, and Open Graph/Twitter share images for a Next.js App Router site using code (next/og's ImageResponse). Covers icon.tsx/apple-icon.tsx/opengraph-image.tsx conventions, embedding real brand fonts under the 500KB bundle limit via font subsetting, and verifying the render before shipping. Use when a project needs a favicon, app icon, or social share card that matches its actual design system instead of a generic placeholder.
user-invocable: true
allowed-tools: Read, Write, Edit, Grep, Glob, Bash
argument-hint: "[nothing for a fresh setup, or a design detail to match]"
---

# Next.js OG Images & Icons

Generate icons and social share images as code, so they render in the project's actual
typography and colors instead of a generic template. Everything here targets the App
Router file-convention APIs (`icon`, `apple-icon`, `opengraph-image`, `twitter-image`)
backed by `next/og`'s `ImageResponse` (Satori + Resvg under the hood).

## Phase 1: Assess

1. **Confirm App Router** - these conventions only exist under `app/`, not `pages/`.
2. **Find the design system** - read `globals.css` / Tailwind theme / `tailwind.config` for
   the actual color tokens and `next/font` setup. The images must reuse these, not invent
   a new palette.
3. **Check for existing icons** - `Glob("app/{favicon.ico,icon.*,apple-icon.*,opengraph-image.*,twitter-image.*}")`.
   A stale `favicon.ico` from `create-next-app` is the default state; replace it.
4. **Note the site's real copy** - name, tagline, one-line description. Pull from an
   existing `metadata` object or `site.ts`/`site config` file if one exists - don't invent
   new marketing copy for the share card.

## Phase 2: Pick static file vs generated route

| Need | Use |
|---|---|
| Icon/image is fixed, no text, no brand font needed | A literal file: `icon.svg`, `icon.png`, `apple-icon.png` |
| Icon/image needs the project's actual typeface, or is built from live data (name, title) | A generated route: `icon.tsx`, `apple-icon.tsx`, `opengraph-image.tsx` |
| `favicon` specifically | **Must** be a literal `favicon.ico` - Next.js does not support generating this one via code. Build the `icon.tsx` first, fetch its rendered PNG, then convert to `.ico` (Phase 6). |

Default to generated routes when the design has a real serif/display font - a hand-picked
system font for a favicon looks off-brand next to the rest of the site.

## Phase 3: Icons

```tsx
// app/icon.tsx - served as the modern favicon (32x32 by convention, but any size works)
import { ImageResponse } from "next/og";
import { readFile } from "node:fs/promises";
import { join } from "node:path";

export const size = { width: 32, height: 32 };
export const contentType = "image/png";

// Read once at module scope - this data doesn't depend on the request.
const brandFont = await readFile(join(process.cwd(), "assets/fonts/Brand-Medium.ttf"));

export default async function Icon() {
  return new ImageResponse(
    (
      <div
        style={{
          width: "100%",
          height: "100%",
          display: "flex",
          alignItems: "center",
          justifyContent: "center",
          background: "#<ink>",
          color: "#<paper>",
          fontFamily: "Brand",
          fontSize: 18,
          lineHeight: 1,
        }}
      >
        AB
      </div>
    ),
    { ...size, fonts: [{ name: "Brand", data: brandFont, style: "normal", weight: 500 }] },
  );
}
```

```tsx
// app/apple-icon.tsx - same shape, larger canvas (iOS home-screen convention: 180x180)
export const size = { width: 180, height: 180 };
// scale fontSize up proportionally (roughly size.height * 0.6), everything else identical
```

**Background must be opaque, never transparent-with-a-tinted-mark.** Tab strips, pinned
tabs, and bookmark bars don't reliably follow `prefers-color-scheme` - a mark meant to
look good on both light and dark chrome needs its own fixed background baked in, not a
transparent PNG that inherits whatever the browser chrome happens to be.

**Getting the monogram centered is trial and error, not math.** Satori's line-box
centering doesn't always match visual centering for a given font's metrics. Iterate with
the verification loop in Phase 6 rather than guessing a `fontSize`/`padding` combination -
two or three round trips is normal.

## Phase 4: Open Graph & Twitter images

| Platform | Recommended | Notes |
|---|---|---|
| Facebook / Open Graph (official spec) | 1200x630, min 200x200 | 1.91:1 ratio, 8MB max file size |
| LinkedIn | 1200x627 | Inherits the OG spec |
| Slack / Discord | Inherits OG spec | No separate published spec |
| X / Twitter, `summary_large_image` | ~1200x628-630 | 5MB max file size. X's own docs are paywalled as of this writing - this is well-corroborated secondary consensus, not a fetched primary source |

1200x630 is the safe default that satisfies every platform above at once. Keep the
important content (name, headline) within the center ~80% of the canvas - X crops toward
a 16:9 frame, so content near the top/bottom edge can get clipped there even though it's
fine on Facebook/LinkedIn.

```tsx
// app/opengraph-image.tsx - 1200x630 is the safe default across Facebook/LinkedIn/Slack/Discord
import { ImageResponse } from "next/og";
import { readFile } from "node:fs/promises";
import { join } from "node:path";
import { SITE_NAME, SITE_TAGLINE } from "@/lib/site"; // reuse real site copy, don't invent new

export const alt = `${SITE_NAME} · ${SITE_TAGLINE}`;
export const size = { width: 1200, height: 630 };
export const contentType = "image/png";

const [displayFont, bodyFont] = await Promise.all([
  readFile(join(process.cwd(), "assets/fonts/Brand-Display.ttf")),
  readFile(join(process.cwd(), "assets/fonts/Brand-Body.ttf")),
]);

export default async function Image() {
  return new ImageResponse(
    (
      <div
        style={{
          width: "100%",
          height: "100%",
          display: "flex",
          flexDirection: "column",
          justifyContent: "space-between",
          background: "#<paper>",
          padding: "88px",
          fontFamily: "Brand Body",
        }}
      >
        <div style={{ display: "flex", flexDirection: "column" }}>
          <div style={{ fontFamily: "Brand Display", fontSize: 104, color: "#<ink>" }}>
            {SITE_NAME}
          </div>
          <div style={{ fontSize: 34, color: "#<muted>", marginTop: 16 }}>{SITE_TAGLINE}</div>
        </div>
        {/* second block: one supporting line, keep it to a single sentence */}
      </div>
    ),
    { ...size, fonts: [
      { name: "Brand Display", data: displayFont, style: "normal", weight: 500 },
      { name: "Brand Body", data: bodyFont, style: "normal", weight: 400 },
    ] },
  );
}
```

**Don't also create `twitter-image.tsx` unless the card needs to differ from the OG
image.** X (Twitter) falls back to `og:image` when no `twitter:image` is present - Next.js
mirrors this in its own metadata resolution, confirmed by inspecting the rendered
`<head>`: setting `twitter: { card: "summary_large_image" }` in `metadata` with no
`images` field still populates `twitter:image` from `openGraph.images`. Generating a
second 1200x630 image doubles build time and font-bundle weight for a card that will look
identical.

**Design it like the site, not like a generic template.** Reuse the real color tokens,
the real display font, and copy pulled from the actual `metadata.description` - not a
new tagline invented for the card. A share card that doesn't match the site it links to
reads as low-effort the instant someone clicks through.

## Phase 5: Embedding a real brand font under the 500KB limit (and avoiding a crash)

`ImageResponse` caps the total bundle (JSX + CSS + fonts + images) at **500KB**, and only
accepts `ttf`, `otf`, or `woff` - **not `woff2`**. This collides with how Google Fonts
ships today: most families in the `google/fonts` GitHub repo are now variable-font-only,
and a single variable `.ttf` (e.g. Newsreader, IBM Plex Sans) commonly runs 400-550KB by
itself - already over budget before adding a second weight or any JSX.

It's not just a size problem, either: **loading a variable font into Satori can crash the
render outright.** [vercel/satori#162](https://github.com/vercel/satori/issues/162) (open)
documents `Cannot read properties of undefined (reading '256')` thrown from Satori's
`fvar`-table parser on a variable font; there's no maintainer-documented workaround beyond
"use a static instance." So a static, single-weight font isn't a nice-to-have
optimization here - it's the only reliable path.

The fix is to fetch an already-subsetted static instance, not the full variable font:

1. Request Google Fonts' `css2` endpoint with an **old User-Agent string**. Modern
   browsers get served `woff2`; an old one (no `woff2` support signaled) gets served a
   real `.ttf` URL instead:
   ```bash
   curl -s -H "User-Agent: Mozilla/5.0 (Windows NT 5.1; rv:1.9)" \
     "https://fonts.googleapis.com/css2?family=<Family>:wght@<weight>&display=swap"
   ```
2. Add `&text=<url-encoded exact string you'll render>` to the same request. Google
   subsets the returned font to only the glyphs in that string - this is the single
   biggest size win, since a general-purpose static font still carries every language's
   glyph set, ligatures, and Unicode blocks you'll never render in a two-line headline.
   ```bash
   curl -s -H "User-Agent: Mozilla/5.0 (Windows NT 5.1; rv:1.9)" \
     "https://fonts.googleapis.com/css2?family=<Family>:wght@<weight>&text=$(python3 -c 'import urllib.parse,sys;print(urllib.parse.quote(sys.argv[1]))' "Exact Ali Turki headline text")&display=swap"
   ```
   Pick the `text` string as the exact union of every character you'll actually render
   with that font across all the images/icons that share it - over-including is fine
   (still small), under-including means a missing-glyph render.
3. Download the resulting `.ttf` from the `src: url(...)` in the response and commit it
   to the repo (e.g. `assets/fonts/`), read via `fs.readFile` at module scope. A 400KB+
   variable font commonly shrinks to **under 15KB** subsetted this way - three fonts
   (display + body + a semibold) comfortably fit the 500KB budget with room to spare.

Never fetch fonts over the network inside the image route itself - read a committed local
file. The Next.js docs' own examples do this (`readFile(join(process.cwd(), ...))`); a
runtime fetch adds latency to every regeneration and a new failure mode.

## Phase 6: Verify before shipping

Don't ship an OG image or icon without actually looking at the rendered output - Satori's
text-centering and line-wrapping don't always match intuition, and a broken font
reference fails silently (falls back to a system font) rather than erroring.

```bash
bun run dev &
curl -s http://localhost:3000/icon -o /tmp/icon.png
curl -s http://localhost:3000/opengraph-image -o /tmp/og.png
```

Then actually view the files (the Read tool renders images inline). For icons specifically,
check centering objectively instead of eyeballing a tiny thumbnail - `magick`'s trim
reports the exact content bounding box so you can tell if margins are balanced:

```bash
magick /tmp/icon.png -fuzz 5% -trim info:
# e.g. "25x13 32x32+3+8" → content is 25x13, offset 3px from left, 8px from top
# left/right margins should be close (3 vs 32-3-25=4 ✓); same for top/bottom
```

Iterate `fontSize`/`padding` against this feedback loop, not by re-reading the same tiny
thumbnail repeatedly.

To build `favicon.ico` from a working `icon.tsx`, render it at a few sizes and pack them:

```bash
magick /tmp/icon.png -resize 16x16 /tmp/f16.png
magick /tmp/icon.png -resize 32x32 /tmp/f32.png
magick /tmp/icon.png -resize 48x48 /tmp/f48.png
magick /tmp/f16.png /tmp/f32.png /tmp/f48.png app/favicon.ico
```

## Known limitations

All sourced from Satori's own README/GitHub issues (https://github.com/vercel/satori) and
Next.js's docs unless noted otherwise.

- **500KB total bundle** (JSX + CSS + fonts + images combined) - the build fails over this,
  not a silent truncation. Font subsetting (Phase 5) is what makes real brand fonts
  feasible at all.
- **`ttf`/`otf`/`woff` only - no `woff2`.** Google Fonts' default modern response is
  `woff2`; the old-User-Agent trick in Phase 5 is required to get a compatible format.
- **Variable fonts can crash the render**, not just render at the wrong weight - see
  Phase 5. Always resolve to a static instance before handing a font to `ImageResponse`.
- **`favicon` cannot be code-generated.** Only `icon`/`apple-icon` support `.tsx`; `favicon`
  is `.ico`-only, and only at the `app/` root.
- **CSS is a real, documented subset** (from Satori's README, not folklore):
  - `display` supports only `flex` (default), `block`, `contents`, `none`, `-webkit-box`.
    Satori's own guidance: *use flex, contents, or none on any div with multiple
    children* - a plain block div with several child elements can lay out wrong.
  - No `calc()`, no 3D transforms, no `z-index` (SVG output has no stacking context).
  - `currentColor` only resolves for the `color` property itself, not e.g.
    `border-color: currentColor`.
  - `gap`, `flexWrap`, `alignItems`/`justifyContent`/`alignSelf`, `textWrap: balance`,
    `lineClamp`, and `whiteSpace` are all supported.
- **No bidi/RTL layout support.** Mixed LTR/RTL text (e.g. an Arabic phrase inside an
  English headline) is not guaranteed to lay out in correct visual order - glyph shaping
  is fine, run ordering isn't. Treat any OG image with Arabic/Hebrew content as needing
  visual verification, not just a clean build.
- **Emoji need explicit handling** - they render as images (via the `emoji` option:
  `twemoji`/`blobmoji`/`noto`/`openmoji`, or a custom `graphemeImages` map), not as font
  glyphs, and are scaled to the current font size as a square.
- **Generated images are statically optimized by default** (rendered once at build time
  and cached) unless the route uses request-time APIs (`cookies()`, `headers()`,
  uncached `fetch`) - which is what makes them safe to use on a fully static/exported site.
- **A missing or misnamed font falls back silently** to Satori's default font rather than
  erroring - always verify visually (Phase 6), don't trust a clean build log alone.

## Rules

1. **Reuse the real design system** - actual color tokens, actual typeface, actual copy.
   Never invent new brand colors or taglines for a share card.
2. **Read fonts from a committed local file at module scope**, never fetch at request time.
3. **Subset fonts to the exact text you render** (Phase 5) - don't commit a full
   variable font and hope it's under budget.
4. **Skip `twitter-image.tsx`** unless the Twitter card genuinely needs to differ from Open
   Graph - it falls back automatically.
5. **Verify the actual rendered PNG** before calling it done - never assume a clean build
   means a good-looking image.
6. **Opaque backgrounds only** on icons - no transparent marks relying on browser chrome
   theme.

## See also

- [[nextjs-seo]] - the `metadata` object, `robots.ts`/`sitemap.ts`/`manifest.ts`, and
  structured data that reference the images this skill generates.
