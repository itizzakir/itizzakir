# 9. Open Questions: please answer before we build

Short answers are fine. The ones marked ⭐ block development. The others can use the default shown.

## Products & pricing
1. ⭐ **Pack sizes:** Is your carton **210 eggs (7 trays)** and tray **30 eggs**? Do you sell **half cartons**, and how many eggs are in one? *(default: 30 / 105 / 210)*
2. ⭐ **Egg types:** Only white (layer) eggs, or also **brown / desi / duck** eggs?
3. Do you sell **loose eggs** (by piece or dozen) online, or only trays and cartons? *(default: trays & cartons only)*
4. ⭐ **Wholesale vs retail:** Different prices for wholesale customers? Who decides who is wholesale, the shop or a minimum quantity (e.g. ≥ 1 carton gets the wholesale rate automatically)?
5. Roughly how often do prices change? Daily? *(affects how prominently we show "updated at")*

## Delivery & orders
6. ⭐ **Delivery area:** Only Muzaffarpur city / specific pincodes? A maximum distance?
7. **Delivery charge** and **minimum order**? Free delivery above some amount?
8. **Pickup from shop** as an option too? *(default: yes, "Self pickup")*
9. **Shop timings:** Opening and closing hours, and weekly off? Should ordering stop when the shop is closed?
10. Same-day delivery only, or can customers choose a delivery **time slot / date**?
11. Who delivers: your own staff or helpers? Do they need their own login now or later?

## Payments
12. Cash on delivery + UPI on delivery for v1. OK? Do you want to show your **UPI ID / QR** in the app?
13. Do wholesale customers buy on **credit (udhaar)** today? (We'd plan the ledger for phase 2.)

## Accounts & verification
14. ⭐ OK to start with **Google sign-in + email OTP (free)** plus a manual phone check on the first order, and add paid SMS OTP later? Or do you want SMS OTP from day one (about ₹0.15–0.50 per SMS + DLT registration)?
15. How many **staff** will use the admin panel?

## Branding & content
16. Do you have a **logo**, shop photos, or preferred colours? *(default: yolk-yellow theme from doc 6)*
17. Shop **phone / WhatsApp number** to show on the website?
18. Default language: **Hindi** or **English**?
19. Domain name: buy one (e.g. `jahangireggs.in`, about ₹800–1,000/year) or use the free `*.vercel.app` URL?
20. Is there a Google Maps location for the shop we can link?

## Technical
21. Shall I create a **separate GitHub repository** `jahangir-egg-center` for the code (recommended), instead of this profile repo?
22. Do you already have a Supabase / Vercel account, or should the setup guide include creating them?
