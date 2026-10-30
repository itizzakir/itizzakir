# 8. Roadmap

## Phase 0: Documentation & decisions ✅
- Requirements, architecture, schema, auth plan, design.
- Collect answers to [open questions](09-open-questions.md).

## Phase 1A: Khata (built first, about 2–3 weeks part-time)
| Week | Deliverable |
|------|-------------|
| 1 | New repo `jahangir-egg-center`, Vite/React/TS/Tailwind setup, Supabase project + migrations, staff/admin login, dealers & customers (parties), opening balances |
| 2 | Dealer purchases (per carton + transport), payments/advances, customer udhaar, payments received, counter sales (incl. loose eggs), WhatsApp reminder |
| 3 | Stock + damage entries, reports (daily, profit, payables, receivables), statements as PDF / WhatsApp, install as PWA on the shop phone |

**Go-live:** enter opening balances, run the paper khata in parallel for one week, and compare daily totals.

## Phase 1B: Online ordering website (about 3–4 weeks part-time)
| Week | Deliverable |
|------|-------------|
| 1 | Customer auth (Google + email OTP), profile completion, catalogue + Price Board |
| 2 | Product sheet, cart (minimum half carton), checkout (delivery/pickup, today/tomorrow/date), `place_order`, My Orders, My Khata |
| 3 | Admin live order queue grouped by date, status buttons, payment capture on delivery (cash/UPI/udhaar → khata) |
| 4 | Notifications (in-app, web push, email), Hindi toggle, polish & animations, deploy |

**Launch checklist:** test on 2–3 real Android phones and one iPhone, do a trial run with
5 known customers, and put a QR code poster in the shop ("Order eggs online – scan me").

## Phase 2: Grow (after launch)
- Phone OTP verification (MSG91/Twilio with DLT).
- Automatic WhatsApp/SMS due reminders (paid API). The manual WhatsApp button is free and in phase 1.
- Delivery slots and a delivery-boy view (assigned orders, one-tap call/navigate).
- Daily/monthly reports and CSV export.
- WhatsApp order updates (optional, paid).
- Razorpay online payment (optional).

## Phase 3: Mobile app
- React Native + Expo app in `apps/mobile`, reusing `packages/shared` and the same Supabase backend.
- Expo push notifications (free).
- Publish on the Google Play Store (one-time ~$25 developer fee). The iOS App Store costs $99/year, so it's optional and iPhone users can keep using the PWA.
