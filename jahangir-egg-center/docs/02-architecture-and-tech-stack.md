# 2. Architecture & Tech Stack

## 2.1 Overview

```
 ┌──────────────────────────┐        ┌───────────────────────────────────────┐
 │  Website / PWA (React)   │        │              Supabase (free)          │
 │  - Customer pages        │  HTTPS │  Auth ......... email OTP, Google     │
 │  - /admin panel          ├───────►│  Postgres ..... tables + RLS          │
 │  Hosted on Vercel (free) │◄───────┤  Realtime ..... live order updates    │
 └────────────┬─────────────┘  WS    │  Edge Functions  push notifications,  │
              │                      │                  order validation     │
 ┌────────────▼─────────────┐        │  Storage ...... product photos        │
 │ Mobile app (later)       ├───────►│                                       │
 │ React Native + Expo      │        └───────────────┬───────────────────────┘
 └──────────────────────────┘                        │ Database webhook on
                                                     ▼ order status change
                                         Web Push (VAPID) / Expo Push
```

We have **no separate backend server**. Supabase provides the database, auth, and APIs.
Business rules live in **Postgres functions** and **Edge Functions**, so the
website and the mobile app share exactly the same logic.

## 2.2 Frontend stack

| Concern | Choice | Why |
|---------|--------|-----|
| Framework | **React 18 + TypeScript + Vite** | Fast dev and builds, as you requested |
| Styling | **Tailwind CSS** + **shadcn/ui** (Radix) | Good-looking, accessible components you own; tiny CSS |
| Animation | **Framer Motion** | Smooth page transitions, cart bounce, status timeline |
| Routing | **React Router v6** | Standard |
| Server state | **TanStack Query** | Caching, retries, offline-friendly |
| Client state | **Zustand** (cart) persisted to localStorage | Tiny and simple |
| Forms | **React Hook Form + Zod** | Validation shared with the backend |
| Backend SDK | **@supabase/supabase-js** | Auth, DB, realtime |
| PWA | **vite-plugin-pwa** (Workbox) | Install on home screen, offline cache, web push |
| i18n | **react-i18next** | Hindi / English |
| Icons | **lucide-react** | Light, consistent |
| Toasts | **sonner** | Pretty notifications in-app |

## 2.3 Backend (Supabase)

| Feature | Used for |
|---------|----------|
| **Postgres** | `profiles`, `products`, `pack_types`, `prices`, `orders`, `order_items`, `addresses`, `push_subscriptions`, `notifications`, `shop_settings` |
| **Row-Level Security** | Customers see only their own rows; staff/admin see all |
| **RPC function `place_order`** | Reads current prices on the server and computes totals, so the client can't fake prices |
| **Realtime** | Admin order queue updates live; customer's order page updates live |
| **Edge Function `notify`** | Triggered on `orders` status change; sends Web Push / Expo Push / email |
| **Auth** | Email OTP (magic code), Google OAuth, optional phone OTP |
| **Storage** | Product images, shop logo |

## 2.4 Proposed repository structure

When we start coding, this should become its own repo, e.g. `jahangir-egg-center`:

```
jahangir-egg-center/
├─ apps/
│  ├─ web/                 # React + Vite website & PWA (customer + /admin)
│  │  ├─ src/
│  │  │  ├─ app/           # routes, layouts, providers
│  │  │  ├─ features/
│  │  │  │  ├─ auth/
│  │  │  │  ├─ catalog/    # products, price display
│  │  │  │  ├─ cart/
│  │  │  │  ├─ orders/
│  │  │  │  └─ admin/      # price board, order queue, customers
│  │  │  ├─ components/ui/ # shadcn components
│  │  │  ├─ lib/           # supabase client, push helpers, formatters
│  │  │  └─ locales/       # en.json, hi.json
│  │  └─ public/           # icons, manifest, service worker
│  └─ mobile/              # (phase 3) Expo app
├─ packages/
│  └─ shared/              # TS types (generated from DB), Zod schemas, cart maths, status labels
├─ supabase/
│  ├─ migrations/          # SQL schema + RLS + functions
│  ├─ functions/notify/    # Edge Function for notifications
│  └─ seed.sql             # default products & pack types
└─ docs/                   # this documentation
```

Monorepo tooling: **pnpm workspaces** (simple, free). Types are generated with
`supabase gen types typescript`, so the frontend always matches the DB.

## 2.5 Why this works well for the later mobile app

- `packages/shared` (types, Zod schemas, cart maths, status texts) is imported by both the web app and the Expo app.
- All business logic sits in Supabase (RPCs, RLS, Edge Functions), so the mobile app only builds new screens.
- Expo gives **free push notifications** and builds Android APK/AAB for free (EAS free tier).
- Alternative shortcut: wrap the PWA with **Capacitor** for a quick Play Store app. It's less native, but the build is almost free.

## 2.6 Security notes

- The **anon key** is public by design. All protection comes from RLS policies (doc 3).
- Prices and totals are **always computed server-side** in `place_order`.
- Admin routes are hidden in the UI **and** blocked by RLS (`role in ('staff','admin')`).
- Rate limiting: Supabase Auth has built-in OTP rate limits, and `place_order` limits each customer to N open orders.
- The **service role key** is used only inside Edge Functions and never shipped to the browser.
