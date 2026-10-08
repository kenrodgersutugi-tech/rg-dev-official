# RG DEVs — Official Software Distribution Website

The official RG DEVs website and software download portal.

**Live site:** https://rgdevs.floot.app

The application is built and hosted on Floot (project `RG DEVs Official`,
id `5bbbc420-4377-48e2-945b-37651505b504`). This repository holds the brand
assets and the maintenance notes; the source of the site lives in the Floot
project, where it is edited and published.

## What the site does

A client opens the site, sees the available software, clicks one button and
the Windows installer downloads immediately. Binaries are **never** hosted,
uploaded or proxied here — every download button is a plain anchor pointing
directly at the public GitHub Release asset, which GitHub serves with
`Content-Disposition: attachment`. GitHub Releases stays the source of truth.

## Pages

| Route                | Purpose                                                    |
| -------------------- | ---------------------------------------------------------- |
| `/`                  | Hero, software catalogue, about, contact                   |
| `/software`          | Full software catalogue with every download                |
| `/software/<slug>`   | Product detail: features, requirements, release info       |
| `/about`             | About RG DEVs                                              |
| `/contact`           | Phone, email, WhatsApp, Instagram                          |
| anything else        | Branded RG DEVs 404                                        |

## Current software

| Product         | Version | Platform | Category             | Editions            |
| --------------- | ------- | -------- | -------------------- | ------------------- |
| RG DEV POS      | 1.1.6   | Windows  | Point of Sale        | Installer           |
| Herdly          | 1.0.0   | Windows  | Livestock Management | Installer, Portable |
| RG Pharma-POS   | 1.0.0   | Web app  | Pharmacy Point of Sale | Browser (installable PWA) |

## Adding another application

Edit **one** file in the Floot project: `helpers/software.tsx`. Append an
object to the `software` array:

```ts
{
  name: "New Software",
  slug: "new-software",
  version: "1.0.0",
  platform: "Windows",
  category: "Business Software",
  description: "Description here",
  downloadUrl: "https://github.com/.../Setup-1.0.0.exe",
  repositoryUrl: "https://github.com/...",
}
```

The card, the detail page at `/software/new-software`, the footer link and the
download buttons are all generated from that object. Optional fields:
`portableDownloadUrl`, `downloads[]`, `logoUrl`, `icon`, `screenshots`,
`featured`, `tagline`, `features[]`, `systemRequirements[]`, `releaseNotes[]`,
`releaseDate`, `installerFileSize`, `portableFileSize`, `priceKes`
(pricing is always Kenyan Shillings). Optionally add the new
`/software/<slug>` URL to `static/sitemap.xml`.

A **web app** is the same object with `webAppUrl` instead of `downloadUrl`
(plus `pwa: true` and `offlineCapable: true` where they apply). It is listed
in the "Web apps" section and its primary action opens the app instead of
downloading a file.

Every release-specific field mirrors what the GitHub Releases API returns, so
the catalogue can later be populated from that API with no frontend changes.

## Brand assets

`components/RgDevsLogo.tsx` draws the logo as inline SVG — a blue tile with
the RG monogram over a green command underscore, plus the wordmark. Variants:
`compact` (mark only), `horizontal` (navbar, footer), `full` (adds the
"Official Software Distribution" line); tones `auto`, `light`, `dark`.

The rasterised assets in `brand/` are generated from that same mark and are
uploaded to Floot's asset storage, registered in `helpers/brandAssets.tsx`:

```
brand/favicon/favicon.ico            brand/favicon/favicon-16x16.png
brand/favicon/favicon-32x32.png      brand/favicon/apple-touch-icon.png
brand/favicon/rg-devs-icon-512.png   brand/social/rg-devs-og-image.png
```

Product glyphs (`components/ProductGlyph.tsx`) share one optical grid, 2px
blue strokes and a single green accent; Herdly and Herdly Portable share a
body so they read as one product family.

## Brand system

- **Colour:** blue `#1450b4` (primary/trust), green `#0d8f5b` (accent,
  downloads, WhatsApp), graphite/black `#0a0f18` (depth, footer, dark mode),
  over cool neutral surfaces. Defined as tokens in `base.css` for light and
  dark mode.
- **Type:** Sora (display/headings), IBM Plex Sans (body/UI), IBM Plex Mono
  (versions, file names, sizes, all machine-readable metadata).
- **Full design principles:** `static/__dev/design-principles.md` in the Floot
  project.

## Contact

RG DEVs — +254 140 205 383 · rgdev.ke@gmail.com ·
[WhatsApp](https://wa.me/254140205383) ·
[Instagram @rg_utugi](https://instagram.com/rg_utugi)

© 2026 RG DEVs. All rights reserved.
