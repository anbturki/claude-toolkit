---
name: nextjs-seo
description: Set up comprehensive SEO metadata for a Next.js App Router site - the Metadata API (title/description/OpenGraph/Twitter/robots/canonical), viewport, manifest.ts, robots.ts, sitemap.ts, and JSON-LD structured data. Use when a project needs proper metadata for search engines and link previews, or when auditing an existing site's metadata for gaps.
user-invocable: true
allowed-tools: Read, Write, Edit, Grep, Glob, Bash
argument-hint: "[nothing for a fresh setup, or a route to audit]"
---

# Next.js SEO Metadata

Wire up the full Metadata API surface for a Next.js App Router site: the `metadata`
export, `viewport` export, and the `robots.ts`/`sitemap.ts`/`manifest.ts` file
conventions, plus structured data. Everything here is file-convention driven - Next.js
reads these exports and generates the right `<head>` tags and files itself; the job is
filling them in correctly, not hand-writing `<meta>` tags.

## Phase 1: Assess

1. **Confirm App Router** and locate the root `app/layout.tsx`.
2. **Find or create a site-config source of truth** - a small `lib/site.ts` (or similar)
   exporting `SITE_URL`, `SITE_NAME`, `SITE_DESCRIPTION`. Every metadata field below
   should import from here, never restate the domain/name/description inline in multiple
   files - that's how a rename or domain change leaves half the metadata stale.
3. **Confirm the production domain.** `metadataBase` and every absolute URL depend on it.
   If the domain isn't decided yet, use a clearly-labeled placeholder in `site.ts` and
   say so - don't guess a domain the user hasn't confirmed.
4. **Check for existing OG/icon images** - if [[nextjs-og-images]] hasn't been run yet,
   the `openGraph`/`twitter` blocks below will reference images that don't exist. Do that
   first, or note it as a follow-up.

## Phase 2: The `metadata` object

In `app/layout.tsx`:

```tsx
import type { Metadata, Viewport } from "next";
import { SITE_DESCRIPTION, SITE_NAME, SITE_TAGLINE, SITE_URL } from "@/lib/site";

const title = `${SITE_NAME} · ${SITE_TAGLINE}`;

export const metadata: Metadata = {
  metadataBase: new URL(SITE_URL),
  title: { default: title, template: `%s · ${SITE_NAME}` },
  description: SITE_DESCRIPTION,
  applicationName: SITE_NAME,
  authors: [{ name: SITE_NAME, url: SITE_URL }],
  creator: SITE_NAME,
  publisher: SITE_NAME,
  alternates: { canonical: SITE_URL },
  openGraph: {
    title,
    description: SITE_DESCRIPTION,
    url: SITE_URL,
    siteName: SITE_NAME,
    locale: "en_US",
    type: "website",
  },
  twitter: {
    card: "summary_large_image",
    title,
    description: SITE_DESCRIPTION,
    // no `images` needed if an opengraph-image file/route exists - Next falls back to it
  },
  robots: {
    index: true,
    follow: true,
    googleBot: { index: true, follow: true, "max-image-preview": "large" },
  },
};

export const viewport: Viewport = {
  themeColor: "#<paper-or-background-token>",
  colorScheme: "light", // or "dark", or omit if the site genuinely supports both
};
```

**`themeColor` and `colorScheme` live in a separate `viewport` export, not `metadata`.**
Putting them in `metadata` is the pre-Next-14 shape and is deprecated - `generateViewport`/
`viewport` was introduced specifically to split viewport-affecting fields (which block
initial paint) from the rest of metadata (which can stream).

**Title/description length - Google has no official character limit.** Their own docs
(developers.google.com/search/docs/appearance/title-link) state titles truncate by
*pixel width*, "as needed to fit the device width," and Google may generate its own
title/snippet from other page signals regardless of what you write. Long-standing
practitioner heuristics (not Google-official, but a reasonable target to hedge toward)
land around 155-160 characters for descriptions on desktop, ~120 on mobile, and titles
under ~60 characters. Treat these as a practical ceiling, not a rule the metadata object
enforces for you.

**Don't add a `keywords` field.** It does nothing for Google - confirmed dead for ranking
since a 2009 Matt Cutts statement, reconfirmed by John Mueller in 2022 ("the answer is
still no"). Bing doesn't use it for ranking either, and a former Bing PM has said a
stuffed keywords tag can be read as a *spam signal*. There's no upside and a small
downside; leave it out rather than cargo-culting it in because older SEO guides mention it.

**`metadataBase` gotchas:** omitting it entirely makes Next.js fall back to
`http://localhost:3000` with a build warning - easy to miss, and it silently ships a
wrong-origin canonical/OG URL if it slips through. A trailing-slash mismatch between
`metadataBase` and `alternates.canonical`/`openGraph.url` was reported on Next 14.1
([vercel/next.js#62522](https://github.com/vercel/next.js/issues/62522)) producing a
stray trailing slash on `og:url`; it wasn't reproducible on Next 16.3.4 in testing for
this skill, but it costs nothing to check your own rendered `<head>` output (Phase 5)
rather than assume it's fixed.

**Icons are automatic - don't hand-write an `icons` field.** If `favicon.ico`,
`icon.tsx`/`icon.png`, and `apple-icon.tsx`/`apple-icon.png` exist under `app/` (see
[[nextjs-og-images]]), Next.js detects them and injects the right `<link rel="icon">` /
`<link rel="apple-touch-icon">` tags itself. A manual `icons: {...}` block is only needed
for icons that *don't* follow the file convention (e.g. serving from a CDN).

## Phase 3: `robots.ts`, `sitemap.ts`, `manifest.ts`

```ts
// app/robots.ts
import type { MetadataRoute } from "next";
import { SITE_URL } from "@/lib/site";

export default function robots(): MetadataRoute.Robots {
  return {
    rules: { userAgent: "*", allow: "/" },
    sitemap: `${SITE_URL}/sitemap.xml`,
  };
}
```

```ts
// app/sitemap.ts - one entry per real route; don't fabricate routes that don't exist
import type { MetadataRoute } from "next";
import { SITE_URL } from "@/lib/site";

export default function sitemap(): MetadataRoute.Sitemap {
  return [
    { url: SITE_URL, lastModified: new Date(), changeFrequency: "monthly", priority: 1 },
    // one object per additional route, once the site actually has more than one
  ];
}
```

```ts
// app/manifest.ts
import type { MetadataRoute } from "next";
import { SITE_DESCRIPTION, SITE_NAME, SITE_TAGLINE } from "@/lib/site";

export default function manifest(): MetadataRoute.Manifest {
  return {
    name: `${SITE_NAME} · ${SITE_TAGLINE}`,
    short_name: SITE_NAME,
    description: SITE_DESCRIPTION,
    start_url: "/",
    display: "standalone",
    background_color: "#<paper>",
    theme_color: "#<paper>",
    icons: [
      { src: "/icon", sizes: "32x32", type: "image/png" },
      { src: "/apple-icon", sizes: "180x180", type: "image/png" },
    ],
  };
}
```

These three are auto-served at `/robots.txt`, `/sitemap.xml`, and `/manifest.webmanifest`
respectively - no manual `<link rel="manifest">` needed, Next.js injects it. Both `.ts`
files are statically rendered at build time by default (same rule as the image routes).

## Phase 4: Structured data (JSON-LD)

For a personal site/portfolio, Google's own docs
(developers.google.com/search/docs/appearance/structured-data/profile-page) name
`ProfilePage` wrapping a `mainEntity: Person` as the correct shape - explicitly listed as
the right fit for "About Me" / personal / author pages (explicitly *wrong* for a store
homepage or an org review site, so don't reach for it there). `Person.name` is the only
hard requirement; `sameAs` (your external profile URLs), `image`, and
`dateModified`/`dateCreated` are recommended, not required.

Next.js's own JSON-LD guide (nextjs.org/docs/app/guides/json-ld) gives the exact injection
pattern - plain `<script type="application/ld+json">`, not `next/script` (that component
is for *executable* JS; JSON-LD isn't executable, so it doesn't need it):

```tsx
// app/page.tsx
import { SITE_NAME, SITE_URL } from "@/lib/site";
import { profile } from "@/lib/data";

const jsonLd = {
  "@context": "https://schema.org",
  "@type": "ProfilePage",
  dateModified: new Date().toISOString().slice(0, 10),
  mainEntity: {
    "@type": "Person",
    name: SITE_NAME,
    url: SITE_URL,
    jobTitle: profile.role,
    sameAs: profile.links.map((l) => l.href), // LinkedIn, GitHub, etc.
  },
};

export default function Home() {
  return (
    <>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{
          // Next.js's own docs are explicit: JSON.stringify alone does NOT sanitize
          // against XSS. Escape `<` so a value containing e.g. "</script>" can't break
          // out of the tag, even though every field here is site-owned, not user input.
          __html: JSON.stringify(jsonLd).replace(/</g, "\\u003c"),
        }}
      />
      {/* page content */}
    </>
  );
}
```

Compose the object only from site-owned constants (`site.ts`, typed profile data) - never
interpolate unsanitized request/user input into it, even with the escape above. For
typing the object, the community `schema-dts` package exports `WithContext<Person>` etc.
if the project wants compile-time schema.org shape checking; optional, not required.

Validate the actual output with Google's Rich Results Test or the generic
[Schema Markup Validator](https://validator.schema.org/) once it's live - a shape that
looks right isn't the same as a shape Google accepts.

## Phase 5: Verify

```bash
bun run build   # confirms static generation succeeds for /, /robots.txt, /sitemap.xml, /manifest.webmanifest
bun run dev &
curl -s http://localhost:3000/ | grep -oE '<title>[^<]*</title>|<meta[^>]*property="og:[^"]*"[^>]*>|<meta[^>]*name="twitter:[^"]*"[^>]*>|<link rel="canonical"[^>]*>'
curl -s http://localhost:3000/robots.txt
curl -s http://localhost:3000/sitemap.xml
curl -s http://localhost:3000/manifest.webmanifest
```

Read the actual `<head>` output, don't just trust that the build didn't error - a typo'd
field name in the `metadata` object fails silently (TypeScript catches most, but a wrong
nested shape can still produce an empty tag).

## Rules

1. **One source of truth for URL/name/description** - a `site.ts` config, never restated
   inline across `layout.tsx`, `manifest.ts`, `robots.ts`, `sitemap.ts`, and image routes.
2. **`viewport` export for `themeColor`/`colorScheme`**, never inside `metadata` - the old
   location is deprecated.
3. **Don't hand-write `icons` in `metadata`** if the file-convention icons exist - Next.js
   auto-detects them.
4. **Confirm the production domain before hardcoding it** - ask rather than guess if it
   isn't already established.
5. **`ProfilePage`/`Person` structured data for a personal site** - not optional polish,
   it's what makes identity resolvable to search engines and other tools. Escape `<` in
   the JSON-LD `__html` string; `JSON.stringify` alone isn't XSS-safe.
6. **No `keywords` field** - confirmed dead weight for Google, spam-risk on Bing.
7. **Verify the rendered `<head>`**, not just a green build.

## See also

- [[nextjs-og-images]] - generates the icon/OG-image files this skill's `metadata` and
  `manifest.ts` reference.
