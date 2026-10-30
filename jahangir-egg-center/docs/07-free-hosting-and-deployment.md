# 7. Free Hosting & Deployment

> Free-tier limits change from time to time. Check the provider pricing pages before launch.
> The numbers below are approximate.

## 7.1 Services

| Service | Plan | What we use | Approx. free limits |
|---------|------|-------------|---------------------|
| **Supabase** | Free | DB, Auth, Realtime, Edge Functions, Storage | 500 MB database, 1 GB storage, 50k monthly active users, 2 active projects |
| **Vercel** (or Netlify / Cloudflare Pages) | Hobby / Free | Host the React site + PWA | Plenty of bandwidth for a local shop |
| **Resend** or **Brevo** | Free | OTP + order emails | Resend ≈ 3,000 emails/month; Brevo ≈ 300/day |
| **Google Cloud** | Free | Google sign-in OAuth client | Free |
| **GitHub** | Free | Code + CI (GitHub Actions) | Free for this use |
| **Expo EAS** (phase 3) | Free | Build Android app, push notifications | Limited free builds per month |
| **Domain** *(optional)* | ~₹800–1,000/year | e.g. `jahangireggs.in` | Free alternative: `jahangir-egg-center.vercel.app` |

For a local egg shop (even a few hundred orders a day) we'll stay well inside these limits.
500 MB holds millions of order rows.

## 7.2 ⚠️ Supabase free project pausing

Supabase **pauses free projects after about 7 days with no activity**. With daily orders this
won't happen, but to be safe:
- A free **GitHub Actions cron** (or cron-job.org) pings a tiny endpoint once a day.
- **Backups:** the free plan has no point-in-time recovery. A weekly GitHub Action runs
  `supabase db dump` and stores the file as a private artifact.

## 7.3 Environments

| Env | Frontend | Supabase |
|-----|----------|----------|
| Local dev | `pnpm dev` (localhost:5173) | `supabase start` (local Docker) |
| Production | Vercel production branch `main` | Supabase cloud project |
| Preview | Vercel preview per PR | Production project (read-only testing) or a 2nd free project |

## 7.4 Environment variables

```
# apps/web/.env
VITE_SUPABASE_URL=https://xxxx.supabase.co
VITE_SUPABASE_ANON_KEY=public-anon-key
VITE_VAPID_PUBLIC_KEY=...

# Supabase Edge Function secrets (never in the browser)
VAPID_PRIVATE_KEY=...
VAPID_SUBJECT=mailto:itizzakir@gmail.com
RESEND_API_KEY=...
```

## 7.5 Deployment steps (summary)

1. Create the Supabase project (region: **Mumbai `ap-south-1`**, closest to Bihar).
2. `supabase link` → `supabase db push` (runs migrations) → run `seed.sql`.
3. `supabase functions deploy notify` and set the secrets.
4. Create the Database Webhook: `orders` UPDATE/INSERT → `notify` function.
5. Import the GitHub repo into Vercel, set the `VITE_*` env vars, and deploy.
6. Add the Vercel URL to Supabase Auth redirect URLs and the Google OAuth client.
7. Set the owner's profile `role = 'admin'`.
8. Open the site on the shop phone → "Install app" → allow notifications.

## 7.6 CI (GitHub Actions)

- On PR: type-check, lint, unit tests (cart maths, status transitions), build.
- On merge to `main`: Vercel auto-deploys. Migrations are applied with `supabase db push` in a workflow, using a secret access token.
