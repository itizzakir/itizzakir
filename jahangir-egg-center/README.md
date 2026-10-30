# 🥚 Jahangir Egg Center — Online Ordering

> Fresh eggs, wholesale & retail — order online, track to your door.

**Shop:** Jahangir Egg Center
**Address:** Near Bulan Darwaja, Shukla Road, Muzaffarpur, Bihar – 842001

This folder holds the planning documentation for the Jahangir Egg Center ordering
platform. We start with a **website (installable PWA)** and later ship a **mobile app**
using the same backend.

## What the product does (in one minute)

| Who | Can do |
|-----|--------|
| **Customer** | Sign up quickly (verified email / mobile), see today's egg price, order by **tray, half carton, or carton** (any quantity), pay cash/UPI on delivery, get notified when the order is **accepted** and **out for delivery**. |
| **Shop owner / staff** | Update prices daily from a phone, see new orders live, accept/reject, mark *out for delivery* / *delivered*, mark wholesale customers. |
| **Owner (khata)** | Record cartons bought from **dealers** (qty, price, advance paid, amount still to pay), keep **customer udhaar** (credit) with payments and WhatsApp reminders, and see stock and profit. |

## Documentation index

| # | Document | What's inside |
|---|----------|---------------|
| 1 | [Product requirements](docs/01-product-requirements.md) | Users, features, units & pricing rules, order life-cycle, scope for v1 |
| 2 | [Architecture & tech stack](docs/02-architecture-and-tech-stack.md) | React + TypeScript + Supabase, libraries, folder structure, how the mobile app reuses it |
| 3 | [Database schema](docs/03-database-schema.md) | Tables, Supabase SQL, Row-Level Security, price snapshots |
| 4 | [Auth & customer verification](docs/04-auth-and-verification.md) | Email OTP, Google sign-in, phone OTP options and real costs in India |
| 5 | [Orders, pricing & notifications](docs/05-orders-pricing-notifications.md) | Cart maths, status flow, push notifications for free |
| 6 | [UI / UX design](docs/06-ui-ux-design.md) | Screens, colours, animations, Hindi/English |
| 7 | [Free hosting & deployment](docs/07-free-hosting-and-deployment.md) | Supabase free tier, Vercel/Netlify, limits and how to stay inside them |
| 8 | [Roadmap](docs/08-roadmap.md) | Phase-by-phase plan from website to mobile app |
| 9 | [Decisions & open questions](docs/09-open-questions.md) | **Owner's answers** and the few details still needed |
| 10 | [Dealers & credit (udhaar) ledger](docs/10-dealers-and-credit-ledger.md) | Dealer purchases, advances & dues, customer udhaar, stock, profit reports |

## Tech stack at a glance

- **Frontend:** React 18 + TypeScript + Vite, Tailwind CSS + shadcn/ui, Framer Motion
- **Backend:** Supabase (Postgres, Auth, Realtime, Edge Functions, Storage) on the free plan
- **Hosting:** Vercel (or Netlify / Cloudflare Pages) on the free plan
- **Notifications:** Web Push (PWA) now → Expo Push for the mobile app later
- **Mobile app (later):** React Native + Expo, reusing types, API code and business rules

**Running cost for v1: ₹0/month.** The only extra cost is SMS OTP, if you want it (see doc 4).
