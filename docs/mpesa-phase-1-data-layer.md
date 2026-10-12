# M-Pesa integration — Phase 1: the data layer

Sandbox only. No real money is involved at any point in this phase.

## Where everything lives

| Thing | Where |
|---|---|
| Site code (pages, components, endpoints) | The **Floot** project, not this repo |
| Database | **Neon** project `rgdev-site` (`green-band-34092690`), eu-central-1, Postgres 18 |
| App database | Neon branch **`main`** (`br-round-term-b1z20uqi`) |
| Test database | Neon branch **`test`** (`br-old-sea-b1ugi0gr`) |
| Secrets | Floot server-side resources. Placeholders in `.env.example` |

The test branch is a real, separate database created from `main`, so the test
suite can write orders and never touch anything the site is using. It cost
nothing and took a second, which is the advantage of Neon branching.

## Tables

### `products`
One row per paid product. Three columns matter:

- **`price_kes`** — the only source of truth for the amount charged. The
  browser never sends a price; the server reads it from here.
- **`requires_payment`** — the per-product paywall switch. **Ships `false`**,
  so the site keeps behaving exactly as it does today until it is turned on.
- `slug` — matches the slug in the site's `helpers/software.tsx`, so the
  catalogue and the database agree on what a product is.

### `product_assets`
The files a purchase unlocks — buying Herdly unlocks **both** its installer and
its portable edition, which is why this is a separate table rather than a
column. It stores *where to find* each file on GitHub Releases
(`github_owner`, `github_repo`, `release_tag`, `asset_name`), never the file
itself. Phase 4 uses these to mint a short-lived signed link.

### `orders`
- **`reference`** — unguessable public id, the only order identifier a customer
  ever sees.
- **`amount_kes`** — *copied* from `products.price_kes` when the order is
  created. Changing a price later never rewrites history.
- `status` — Postgres enum: `pending | paid | failed | cancelled | expired`.
- `merchant_request_id`, `checkout_request_id`, `mpesa_receipt_number`,
  `result_code`, `result_desc` — what Daraja gives back.
- `checkout_request_id` has a **unique index**, which is what makes repeated
  callbacks safe: the same STK request can only ever map to one order.
- `expires_at` — set 10 minutes ahead. A `pending` order past it is `expired`.
- `mpesa_env` — records whether an order was made against sandbox or
  production, so test orders can never be mistaken for real ones.

### `payment_events`
Append-only audit log. Every callback and every STK query response is stored
**raw, before anything is interpreted** — including payloads that match no
order (a forged or unknown `CheckoutRequestID`). Never updated, never deleted.

### `downloads`
One row per download link issued: the link token, when it expires (5 minutes),
when it was redeemed, the IP and user agent. Every download of a paid file is
accounted for.

### `lookup_attempts`
Rate limiting for order recovery, so an order reference cannot be found by
guessing.

## Seeded data

| Product | Price | Paywall | Unlocks |
|---|---|---|---|
| RG DEV POS | KES 50 | off | `RG-Dev-POS-Setup-1.1.6.exe` (94 MB) |
| Herdly | KES 50 | off | `HerdlySetup-1.0.0.exe` (110 MB) + `Herdly-1.0.0-portable.exe` (110 MB) |

RG Pharma-POS is deliberately absent: it is a free web app and stays unchanged.

To change a price later: update `products.price_kes`. No code change, no deploy.
