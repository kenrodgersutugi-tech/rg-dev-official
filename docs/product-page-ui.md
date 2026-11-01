# Product page UI — screenshots and Apple-style layout

## What was added

**A screenshot gallery** (`components/ScreenshotShowcase.tsx`). One large
screenshot in hardware chrome, a thumbnail strip when there is more than one,
and a full-screen viewer — dark scrim, arrows, a counter, Escape to close,
focus returned to the button that opened it.

**Device frames** (`components/DeviceFrame.tsx`). A screenshot floating on a
page reads as a stray image; framed, it reads as the application. Windows apps
get a window with a title bar, web apps get browser chrome with the real
address. Drawn in CSS, so there are no frame image assets and it follows the
theme.

**A bento feature grid** (`components/FeatureBento.tsx`). Features were a flat
checklist in a narrow column; they are now a full-width grid of tiles with
varied widths. A feature written as `Lead — detail` or `Lead: detail` is split
so the lead sets as a heading. `grid-auto-flow: dense` backfills the hole a
wide tile would otherwise leave.

**Layout rhythm.** The features grid moved out of the two-column block to full
width, with the screenshot band between the download panel and the detail. The
download panel deliberately stays directly under the header: this is a
download site, and the primary action should not sit below a large image.

## The rule about screenshots

`screenshots` in `helpers/software.tsx` must be **real screenshots of the
application**. Never a mock-up, never an illustration of an interface that does
not exist. People decide whether to install from these images.

A product with no screenshots renders no gallery at all — the page simply keeps
the shape it had. An empty frame reads as a broken image, and an invented
screenshot would be worse than either.

## What is there now

| Product | Screenshots |
|---|---|
| RG Pharma-POS | 1 — the live app's sign-in screen, captured from the deployed site |
| RG DEV POS, RG-Present, Herdly, Herdly Pro | none yet |

RG Pharma-POS's dispensing and till screens are behind its sign-in, so they can
only be captured by someone with an account. The four Windows products can only
be captured on a Windows machine running them.

## Adding screenshots

1. Take them at 16:10 if you can (2880x1800 is ideal; 1440x900 is fine).
2. Upload each to the project's assets and copy the `/_cdn/static/...` path.
3. Add them to that product in `helpers/software.tsx`:

```ts
screenshots: [
  "/_cdn/static/....png",
  "/_cdn/static/....png",
],
```

The gallery, thumbnails and viewer appear on that product's page with no other
change. Absolute `https://` URLs work too — `isUsableImage` in
`helpers/softwareCatalog.tsx` accepts both, while download URLs stay restricted
to absolute links to real release assets.

## Verified live

Gallery, thumbnails and full-screen viewer on RG Pharma-POS; Escape closes it;
9 bento tiles on RG DEV POS; no console errors at 1440 / 834 / 390 px; no
horizontal overflow at any of those widths; the RG DEV POS 1.6.0 download still
returns `RG-Dev-POS-Setup-1.6.0.exe`.
