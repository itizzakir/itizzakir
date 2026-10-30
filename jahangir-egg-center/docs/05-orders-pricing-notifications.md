# 5. Orders, Pricing & Notifications

## 5.1 Cart maths (shared code in `packages/shared`)

```ts
type PackType = 'tray' | 'half_carton' | 'carton';

interface CartLine {
  productPackId: string;
  quantity: number;          // number of packs, 1..999
}

// Display only. The server recomputes everything in place_order().
function cartSummary(lines: CartLine[], packs: Record<string, ProductPack>, type: 'retail' | 'wholesale') {
  let subtotal = 0, totalEggs = 0;
  for (const l of lines) {
    const p = packs[l.productPackId];
    const price = type === 'wholesale' ? p.wholesalePrice : p.retailPrice;
    subtotal  += price * l.quantity;
    totalEggs += p.eggsPerPack * l.quantity;
  }
  return { subtotal, totalEggs };
}
```

Example: 2 Cartons + 1 Half Carton + 3 Trays = 420 + 105 + 90 = **615 eggs**.

**UI rules**
- The quantity stepper **+ / –** supports long-press to change quickly. Tapping the number lets you type it (for wholesale).
- A **"Converts to"** hint helps customers, e.g. "7 trays = 1 carton. Switch and save ₹X?"
  (only when the carton price per egg is lower).
- If a price changes while items sit in the cart, the cart shows
  *"Price updated: ₹X → ₹Y"* before checkout.

## 5.2 Price Board (admin)

One screen, one table, and one **Save** button:

| Product | Pack | Retail ₹ | Wholesale ₹ | In stock |
|---------|------|----------|-------------|----------|
| White Egg | Tray (30) | [ 180 ] | [ 170 ] | ✅ |
| White Egg | Carton (210) | [ 1230 ] | [ 1180 ] | ✅ |

- Big number inputs that are easy to tap on a phone.
- **"Change all by ±₹"** quick action, handy when the NECC rate moves.
- Saving calls the `update_prices` RPC, which writes `price_history`. Customers see new prices right away (Realtime).

## 5.3 Status transitions

Allowed transitions are enforced in `set_order_status()`:

| From | To |
|------|----|
| placed | accepted, rejected, cancelled (customer) |
| accepted | out_for_delivery, cancelled (staff only) |
| out_for_delivery | delivered |
| accepted (pickup order) | ready_for_pickup |
| ready_for_pickup | picked_up |

The admin order card shows **one big button for the next step**
(e.g. "Accept" → "Out for delivery" → "Delivered"). That makes it hard to make mistakes.

## 5.4 Notifications, all free

### Channels

| Channel | When | Cost |
|---------|------|------|
| **In-app** (Realtime + `notifications` table) | Always | Free |
| **Web Push** (PWA, VAPID keys) | Customer allowed notifications | Free |
| **Email** | Fallback for important events (accepted / rejected) | Free tier of Resend/Brevo |
| **Expo Push** (mobile app, phase 3) | App installed | Free |
| *SMS / WhatsApp* | Optional later | Paid |

> **iPhone note:** web push works on iOS 16.4+ **only after the user adds the site to the
> Home Screen**. The app shows a small "Add to Home Screen" guide for iPhone users.
> Android Chrome works normally.

### Flow

```
Shop taps "Accept"
   → set_order_status() updates orders.status
   → Database Webhook (on UPDATE of orders) calls Edge Function `notify`
        → inserts a row into notifications (in-app)
        → looks up push_subscriptions for the customer
        → sends Web Push (web-push lib) / Expo push
        → removes expired subscriptions (HTTP 404/410)
   → Customer's open order page also updates instantly via Realtime
```

New order → the same function notifies **all staff/admin** subscriptions, and the admin panel
plays a sound.

### Message templates (Hindi / English)

| Event | English | हिंदी |
|-------|---------|-------|
| accepted | ✅ Order #1042 accepted. We're packing your eggs! | ✅ ऑर्डर #1042 स्वीकार हुआ। आपके अंडे पैक हो रहे हैं! |
| rejected | ❌ Order #1042 couldn't be accepted: {reason} | ❌ ऑर्डर #1042 स्वीकार नहीं हो सका: {reason} |
| out_for_delivery | 🛵 Order #1042 is out for delivery! | 🛵 ऑर्डर #1042 डिलीवरी के लिए निकल गया है! |
| delivered | 🥚 Delivered. Thank you for ordering from Jahangir Egg Center! | 🥚 डिलीवर हो गया। जहांगीर एग सेंटर से ऑर्डर करने के लिए धन्यवाद! |
| new order (staff) | 🔔 New order #1042: 2 Carton, 3 Tray (₹2,999) | 🔔 नया ऑर्डर #1042 |

## 5.5 Payment

- v1: **Cash** and **UPI** on delivery or at pickup (customer scans the shop QR, staff marks it paid).
- **No delivery charge.** Delivery is included in the egg price. Minimum online order is half carton (105 eggs).
- Orders are for **today, tomorrow, or a chosen date** (up to `max_days_ahead`). The admin queue groups orders by date.
- The checkout page can show the shop's UPI QR / UPI ID.
- On **Delivered**, staff records how it was paid: **Cash / UPI / Udhaar / Part payment**.
  Any unpaid amount goes to the customer's khata automatically ([doc 10](10-dealers-and-credit-ledger.md)).
- Customers **cannot** choose udhaar online. Only the shop decides it, when marking delivered/picked up.
- Later: Razorpay (fee per transaction, no monthly cost) for prepaid orders.
