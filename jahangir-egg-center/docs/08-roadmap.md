# 8. Roadmap

## Phase 0: Documentation & setup ✅ (this)
- Requirements, architecture, schema, auth plan, design.
- Collect answers to [open questions](09-open-questions.md).

## Phase 1: Website MVP (about 3–4 weeks part-time)
| Week | Deliverable |
|------|-------------|
| 1 | Repo + Vite/React/TS/Tailwind setup, Supabase project, migrations, seed data, auth (Google + email OTP), profile completion |
| 2 | Catalogue + Price display, product sheet, cart, checkout, `place_order` RPC, My Orders |
| 3 | Admin panel: live order queue, status buttons, Price Board, customers, shop settings |
| 4 | Notifications (in-app, web push, email), PWA install, Hindi/English, polish & animations, deploy |

**Launch checklist:** test on 2–3 real Android phones and one iPhone, do a trial run with
5 known customers, and put a QR code poster in the shop ("Order eggs online – scan me").

## Phase 2: Grow (after launch)
- Phone OTP verification (MSG91/Twilio with DLT).
- **Udhaar / credit ledger** for wholesale customers (balance, payments, statements).
- Delivery slots and a delivery-boy view (assigned orders, one-tap call/navigate).
- Daily/monthly reports and CSV export.
- WhatsApp order updates (optional, paid).
- Razorpay online payment (optional).

## Phase 3: Mobile app
- React Native + Expo app in `apps/mobile`, reusing `packages/shared` and the same Supabase backend.
- Expo push notifications (free).
- Publish on the Google Play Store (one-time ~$25 developer fee). The iOS App Store costs $99/year, so it's optional and iPhone users can keep using the PWA.
