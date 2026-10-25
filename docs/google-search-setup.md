# Getting RG DEVs found on Google

## The problem that was actually there

The site is a React app. Before this change, only the home page was
pre-rendered: every other URL returned the home page's HTML shell. A crawler
asking for `/software/rg-dev-pos` was handed the title
"RG DEVs | Official Software Downloads" and no `<h1>` at all.

To Google that reads as seven near-duplicate copies of one page, which is the
single most common reason a React site does not rank. Google can run
JavaScript, but rendering is queued and deferred, and duplicate titles get
collapsed regardless.

## What changed

**Pre-rendering is on** (`enableSSR`). Every route is now rendered on the
server, so each URL arrives with its own title, description, canonical and
real content:

| URL | Title | `<h1>` |
|---|---|---|
| `/` | RG DEVs \| Official Software Downloads | Software Built by RG DEVs |
| `/software` | RG DEVs Software \| Official Windows Downloads | RG DEVs Software |
| `/software/rg-dev-pos` | RG DEV POS 1.1.6 for Windows \| RG DEVs | RG DEV POS |
| `/software/herdly` | Herdly 1.0.0 for Windows \| RG DEVs | Herdly |
| `/software/rg-pharma-pos` | RG Pharma-POS \| RG DEVs web app | RG Pharma-POS |
| `/about` | About RG DEVs \| Software Development Company | (page heading) |
| `/contact` | Contact RG DEVs | Contact RG DEVs |

Two things had to be made safe to render on a server first, because both wrap
every page and both touched the DOM *during* render:

- `helpers/themeMode.tsx` — `getCurrentThemeMode()` reads `document.body` from
  the provider's `useState` initialiser.
- `components/ScrollToHashElement.tsx` — `document.getElementById` inside a
  `useMemo`.

Both now check for a DOM first and fall back to the light theme on the server;
the browser applies the real theme on hydration. Verified live: no console
errors, no hydration warnings, theme switching and downloads still work.

**The sitemap is now generated, not static.** With pre-rendering on, Floot
routes `/sitemap.xml` to a `sitemap` GET endpoint, so the old static file was
never served — the URL returned the app's 404 page. `endpoints/sitemap_GET.ts`
builds the XML from the software catalogue, so a product added to
`helpers/software.tsx` appears in the sitemap with no second list to maintain.

**Error pages are no longer advertised as indexable.** The 404 and
"product not found" pages previously said `index, follow` *and* pointed their
canonical at the home page — an invitation to index an error page as the home
page. They now send `noindex, follow` and no canonical.

**Breadcrumb structured data** on product pages, alongside the existing
`SoftwareApplication` data, so results can show
`RG DEVs > Software > RG DEV POS`.

## What you still have to do (Google needs a human)

1. Open **https://search.google.com/search-console** and sign in.
2. Add a property → choose **URL prefix** → enter `https://rgdevs.floot.app`.
3. Pick the **HTML tag** verification method. It shows a tag like
   `<meta name="google-site-verification" content="AbC123..." />`.
4. Send me the `content` value (it is not a secret), or paste it yourself into
   `googleSiteVerification` in `helpers/siteConfig.tsx`. Publish, then press
   **Verify**.
5. In Search Console → **Sitemaps**, submit `sitemap.xml`.
6. Use **URL Inspection** → *Request indexing* for the home page and each
   product page to jump the queue.

## Honest expectations

- Indexing takes days to a few weeks. Nobody, including Google, can force it.
  Verification and the sitemap only make the site *eligible* and discoverable.
- Searches for "RG DEVs" will compete with unrelated results at first. Product
  names ("RG DEV POS", "Herdly livestock software") rank sooner because they
  are distinctive.
- A **custom domain** (rgdevs.co.ke, say) carries more long-term brand weight
  than a `floot.app` subdomain, and lets you verify by DNS. It needs a paid
  Floot plan.
- A wrong URL returns HTTP 200 with a "Page not found" page rather than a real
  404 status; the host does not let a page choose its status code. `noindex`
  plus clear "Page not found" wording is the correct handling available here.
