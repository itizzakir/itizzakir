# 6. UI / UX Design

**Feel:** warm, fresh, friendly, and fast. Big buttons and few steps.
It should be easy enough for someone who has never shopped online.

## 6.1 Brand

| Token | Value | Use |
|-------|-------|-----|
| Primary (yolk) | `#F5A623` | Buttons, highlights |
| Primary dark | `#D98A0B` | Pressed state, text on light |
| Shell cream | `#FFF8EC` | Page background |
| Brown (desi egg) | `#8B5A2B` | Headings, accents |
| Fresh green | `#2E9E5B` | Success, "Delivered", "In stock" |
| Ink | `#2B2B2B` | Body text |
| Danger | `#D64545` | Reject / errors |

- **Font:** "Poppins" for headings and "Inter" for body. Both support Devanagari
  ("Noto Sans Devanagari" for Hindi).
- **Logo idea:** a simple egg outline with "JEC" or 🥚 + "Jahangir Egg Center". We'll replace it with a real logo if you have one.
- Dark mode supported (automatic).

## 6.2 Screens: customer

1. **Splash / Welcome:** logo, tagline "Fresh eggs, delivered", Google button, email button.
2. **Home**
   - Top: shop name, open/closed chip, "Today's rate · updated 8:15 AM".
   - Product cards with egg illustration, price per tray / carton, and an **Add** button.
   - "Order again" card with the last order, re-ordered in one tap.
3. **Product sheet** (bottom sheet, slides up): segmented control **Tray | Half Carton | Carton**,
   big quantity stepper, live "= 210 eggs · ₹1,230".
4. **Cart:** list of lines, egg count, subtotal, delivery, total, and a sticky **Place order** button.
5. **Checkout:** **Delivery / Pickup** toggle, address (saved, or add with landmark and optional map pin), **Today / Tomorrow / Pick a date** chips, payment Cash/UPI, note.
6. **Order success:** animated egg crack 🐣 then "Order placed!" and a live status timeline.
7. **My Orders:** cards with coloured status chips. Tap for the timeline:
   `Placed ● ─ Accepted ● ─ Out for delivery ○ ─ Delivered ○`.
8. **My Khata** (only for customers with an account in the khata): due amount, entries, statement.
9. **Profile:** name, phone, addresses, language, notification toggle, logout.

Bottom navigation: **Home · Cart (badge) · Orders · Profile**.

## 6.3 Screens: admin (`/admin`, mobile-first too)

1. **Orders (live):** tabs New / Accepted / On the way / Done. Each card has customer, phone (tap to call),
   address (tap to open Google Maps), items, total, and one big **next-step button**.
2. **Price Board:** the editable table from doc 5.
3. **Customers:** search, mark wholesale, verify phone, block.
4. **Dashboard:** today's orders, eggs sold, revenue, and a 7-day chart.
5. **Khata:** dealers (*To pay* / *Advance*), customer udhaar due list, counter sale, stock, and reports, with a floating **+** for quick entries. See [doc 10](10-dealers-and-credit-ledger.md#107-admin-screens-added).
6. **Settings:** shop open/closed, minimum order, delivery charge, UPI ID, staff.

## 6.4 Motion & "smoothness"

- Page transitions: short fade and slide (150–200 ms) with Framer Motion.
- Add to cart: button morphs into a stepper and the cart icon bounces.
- Bottom sheets with drag-to-close.
- Skeleton loaders instead of spinners.
- Optimistic UI: cart and status changes show immediately.
- Respect `prefers-reduced-motion`.
- Keep total JS small and lazy-load the admin pages, so it stays smooth on budget phones.

## 6.5 Language & accessibility

- **English default** (switchable to Hindi), with numbers shown as ₹1,230.
- Minimum tap target 44 px and font size ≥ 16 px.
- Good colour contrast. Icons always have text labels.
