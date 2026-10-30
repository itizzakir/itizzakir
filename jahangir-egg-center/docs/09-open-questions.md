# 9. Decisions & Open Questions

## 9.1 Decisions (answered by the owner, Oct 2026)

### Products & pricing
| Topic | Decision |
|-------|----------|
| Pack sizes | **Tray = 30 eggs, Half carton = 105 eggs, Carton = 210 eggs** |
| Egg types | **White (layer) eggs only** for now (schema still supports adding more later) |
| Loose eggs | **Counter sales only** (by piece). Not sold online. |
| Wholesale | **Shop marks** a customer as wholesale. Those customers always see wholesale prices. |

### Orders & delivery
| Topic | Decision |
|-------|----------|
| Minimum online order | **Half carton (105 eggs)** |
| Delivery charge | **None shown separately.** Delivery is included in the egg price. |
| Delivery area | **Muzaffarpur city only** |
| Pickup | **Yes.** Customer chooses *Home delivery* or *Pickup from shop*. |
| When | **Same day**, or **book for the next day / a chosen date** |
| Shop timings | **Owner sets them in the app** (open/close time, weekly off). Default 7 AM – 9 PM, all days. |
| Payment | **Cash + UPI on delivery/pickup.** UPI ID / QR is shown in the app. No online gateway yet. |
| GST bills | **Not needed.** Simple receipts only. |

### Accounts
| Topic | Decision |
|-------|----------|
| Customer login | **Google + email OTP (free)**. Shop confirms the phone number by a call on the first order. SMS OTP later. |
| Default language | **English** (Hindi toggle available) |

### Khata (dealers & udhaar)
| Topic | Decision |
|-------|----------|
| Build order | **Khata first**, online ordering after |
| Dealer rate | **Per carton** |
| Dealer extras | **Transport charged extra.** Dealers do **not** credit broken eggs, so breakage is our loss and is recorded as stock damage. |
| Who sees dealer prices & profit | **Owner (admin) only** |
| Udhaar control | **Shop decides only.** Customers can't choose udhaar online. |
| My Khata for customers | **Yes.** Linked customers can see their dues. |
| Opening balances | **Yes.** Enter current dealer and udhaar balances on day one (recommended). |

### Branding, hosting & code
| Topic | Decision |
|-------|----------|
| Logo & colours | **Designed by us:** yolk-yellow theme + simple egg logo (doc 6) |
| Domain | **Free `*.vercel.app`** for now |
| Code location | **New GitHub repo `jahangir-egg-center`** |
| Supabase / Vercel | Owner **already has accounts** |

## 9.2 Still needed (small details, not blocking the start)

1. Shop **phone / WhatsApp number** to show in the app and in WhatsApp reminders.
2. **UPI ID** (and a QR image) for payments.
3. **Google Maps link** of the shop.
4. Roughly how many **dealers** and **udhaar customers** you have today (for testing with realistic data).
5. How many **staff** will use the admin panel, and their emails (to give them the `staff` role).
6. The new GitHub repo `jahangir-egg-center`: you create it and give access, or I create it if my access allows.
