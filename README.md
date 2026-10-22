# Pure Peel Co.

Production e-commerce platform for Pure Peel Co., a small-batch dehydrated citrus brand based in Toronto.

**Live site:** [purepeelco.com](https://purepeelco.com)

<!-- Add a screenshot: drag an image into this file on GitHub, then replace this comment -->

## What it does

- Storefront for six products with a cart, Stripe Checkout, and order tracking
- Webhook-driven order pipeline: payments create orders in PostgreSQL and trigger confirmation emails
- Admin dashboard for managing orders, buying shipping labels through ChitChats, and sending tracking notifications
- Email subscriber list with a welcome email and signed one-click unsubscribe links
- English and French, with per-page SEO metadata and structured data

## Architecture

```
Browser ──► React + Vite frontend (Vercel)
               │
               ▼
        Express REST API (Render) ──► Neon PostgreSQL
               │
               ├──► Stripe     (Checkout + webhooks)
               ├──► Resend     (transactional email + subscriber contacts)
               └──► ChitChats  (shipping labels + tracking)
```

| Layer | Tech |
|---|---|
| Frontend | React, Vite, Tailwind CSS |
| Backend | Node.js, Express |
| Database | PostgreSQL (Neon) |
| Payments | Stripe Checkout, Stripe webhooks |
| Email | Resend |
| Shipping | ChitChats API |
| Hosting | Vercel (frontend), Render (API) |

## Engineering notes

**Idempotent order creation.** Stripe may deliver the same webhook event more than once, and the checkout success page can also confirm a session. Before creating an order, the server checks whether that checkout session has already been recorded, so retries and races never produce duplicate orders.

**Duplicate-safe subscriptions.** The subscribe endpoint reserves an email in memory before any async work, so two simultaneous signups (a double-click, for example) can't both pass the "already subscribed?" check and send two welcome emails.

**Signed unsubscribe links.** Each marketing email carries an HMAC-signed, expiring token. The API verifies it before removing the contact, and the `List-Unsubscribe` / `List-Unsubscribe-Post` headers support one-click unsubscribe from email clients (RFC 8058).

**API hardening.**
- Per-route rate limits: stricter on checkout and order lookup than on general endpoints
- Request validation with `express-validator`
- CORS restricted to the site's own origins
- Security headers: HSTS, `X-Frame-Options`, and others

## Running locally

See [`LOCAL_DEV.md`](LOCAL_DEV.md) for setup. Secrets are loaded from a local `.env` file, which is git-ignored.

Main environment variables:

| Variable | Purpose |
|---|---|
| `DATABASE_URL` | Neon PostgreSQL connection string |
| `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET` | Payments and webhook signature verification |
| `RESEND_API_KEY`, `RESEND_FROM_EMAIL` | Outgoing email |
| `FRONTEND_URL`, `PUBLIC_API_URL` | Base URLs used in emails and redirects |
| `ADMIN_PASSWORD` | Admin dashboard login |

## More documentation

- [Email flow](docs/EMAIL_FLOW_AND_STRUCTURE.md)
- [Admin dashboard](docs/HOW_ADMIN_DASHBOARD_WORKS.md)
- [Shipping](docs/HOW_SHIPPING_WORKS.md)
- [Security](docs/WEBSITE_SECURITY_GUIDE.md)

---

Built and maintained by [Matthew Granato](https://www.linkedin.com/in/matthew-granato/).
