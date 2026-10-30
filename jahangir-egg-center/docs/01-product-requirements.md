# 1. Product Requirements

## 1.1 Goal

Let customers of **Jahangir Egg Center** (Muzaffarpur) order eggs online in a few taps,
and let the shop manage prices and orders from a phone, all on free infrastructure.

Success looks like:
- A new customer can sign up and place a first order in **under 2 minutes**.
- The shop can update the day's price in **under 30 seconds**.
- Customers learn about status changes right away, with no phone call needed.

## 1.2 Users & roles

| Role | Description |
|------|-------------|
| `customer` (retail) | Households, small buyers. Usually orders trays. |
| `customer` (wholesale) | Shops, hotels, bakeries, dhabas. Orders cartons. The owner can mark an account as wholesale, and wholesale accounts see wholesale prices. |
| `staff` | Shop helpers who accept orders and update delivery status. |
| `admin` | Owner (your father). Everything staff can do, plus prices, products, staff management, and **dealer accounts / purchase prices**. |
| *Dealer* (not a login) | Supplier we buy eggs from. Exists only as a record in the khata. |

## 1.3 Selling units

Eggs are sold in **packs**. The owner sets each pack's size and price, so no unit is hard-coded.

| Pack | Size (confirmed) | Notes |
|------|---------------------------|-------|
| Tray | 30 eggs | Most common retail unit |
| Half carton | 105 eggs (3½ trays) | **Minimum online order** |
| Carton | 210 eggs (7 trays) | Standard wholesale carton in India |
| Loose eggs | 1 egg | **Counter sales only**, not online |

> Confirmed by the owner. All decisions are listed in [doc 9](09-open-questions.md).

The customer can mix packs in one order, for example **2 cartons + 1 half carton + 3 trays**.
The cart shows the total egg count, e.g. "= 615 eggs".

At launch we sell **white (layer) eggs only**. Products can have **variants** later, such as brown/desi eggs.
Each variant has its own pack prices.

## 1.4 Pricing rules

- Prices change often, sometimes daily (they follow NECC rates and the market).
  The admin updates them from the **Price Board** screen.
- Each pack has a **retail price** and a **wholesale price**.
- The price is **locked when the order is placed** (price snapshot), so a later
  price change never alters an existing order.
- An optional "Today's rate" banner on the home page shows the last update time,
  e.g. *"Updated today 8:15 AM"*.
- **Minimum online order: half carton (105 eggs).**
- **No separate delivery charge.** Delivery is included in the egg price.

## 1.5 Order life-cycle

```
 placed ──► accepted ──► out_for_delivery ──► delivered
                 │
                 └──► ready_for_pickup ──► picked_up   (pickup orders)
   │            │
   ├─► rejected (by shop, with reason)
   └─► cancelled (by customer, only while still "placed")
```

| Status | Who sets it | Customer gets notified |
|--------|-------------|------------------------|
| `placed` | Customer | ✅ "Order received" (in-app) |
| `accepted` | Shop | ✅ Push: "Your order is accepted" |
| `rejected` | Shop | ✅ Push + reason |
| `out_for_delivery` | Shop | ✅ Push: "Your eggs are on the way 🛵" |
| `delivered` | Shop | ✅ Push: "Delivered. Thank you!" |
| `ready_for_pickup` | Shop | ✅ Push: "Your eggs are ready at the shop" |
| `picked_up` | Shop | ✅ Push: "Thank you!" |
| `cancelled` | Customer | Shop gets notified |

The shop also gets a **live alert (sound + push) for every new order**.

## 1.6 Features: v1 (website / PWA)

**Customer**
- [ ] Sign up / log in with **email OTP** or **Google** (verified). Phone number is required on the profile.
- [ ] Home: today's prices, quick "Order again" button.
- [ ] Product page: pick pack type and use the **+ / –** quantity stepper.
- [ ] Cart: total eggs and total (no delivery charge). Blocks checkout below half carton.
- [ ] Checkout: **Home delivery** (Muzaffarpur city, saved addresses with landmark) or **Pickup from shop**; **today, tomorrow, or a chosen date**; note; payment = **Cash / UPI on delivery/pickup**.
- [ ] My Orders: live status timeline.
- [ ] Notifications: push (when allowed) and an in-app notification list.
- [ ] English by default, with a Hindi toggle.

**Shop (admin panel inside the same site at `/admin`)**
- [ ] Price Board: edit all prices on one screen and save once.
- [ ] Live order queue with sound alert: Accept / Reject / Out for delivery / Delivered.
- [ ] Customer list: mark wholesale, block fake accounts.
- [ ] Daily summary: orders, eggs sold, revenue.
- [ ] Shop settings: open/closed toggle, **opening/closing time and weekly off**, how many days ahead orders can be booked, UPI ID/QR.

**Khata / accounts (details in [doc 10](10-dealers-and-credit-ledger.md))**
- [ ] Dealers: record purchases (cartons × price, transport, damage), payments and advances; see *To pay* / *Advance* per dealer.
- [ ] Customer udhaar: credit sales (online or counter), payments received, due list, credit limit, WhatsApp reminder.
- [ ] Counter sales (cash / UPI / udhaar), so stock and reports are complete.
- [ ] Stock: eggs in (purchases) − eggs out (sales) − damage.
- [ ] Reports: daily summary, approximate profit, payables, receivables.

## 1.7 Out of scope for v1 (later phases)

- Online payment gateway (Razorpay); start with Cash/UPI on delivery.
- Delivery-boy app with live GPS tracking.
- Invoices / GST bills as PDF.
- Native mobile apps on Play Store / App Store (phase 3).

## 1.8 Non-functional requirements

- **Mobile-first.** Most users will be on low/mid Android phones over 4G.
- **Fast.** First load under 2 s on 4G, JS bundle under 200 KB gzipped for customer pages.
- **Works offline-ish.** Previously loaded prices and orders still show without network.
- **Secure.** Customers can see only their own data (Postgres Row-Level Security).
- **Free to run** within the free-tier limits in [doc 7](07-free-hosting-and-deployment.md).
