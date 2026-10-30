# 4. Authentication & Customer Verification

**Goal:** sign-up must be **very easy**, but only **genuine customers** should get in
(no fake orders).

## 4.1 Options compared

| Method | Proves | Cost | Ease for customer | Notes |
|--------|--------|------|-------------------|-------|
| **Email OTP (6-digit code)** | Real email | **Free** | Easy | Supabase built-in. Its default mailer allows only a few emails per hour, so add a free SMTP provider (Resend / Brevo) for real use. |
| **Google sign-in** | Real Google account (email already verified) | **Free** | **Easiest**: one tap | Most Android users already have a Google account. Recommended as the main button. |
| **Phone SMS OTP** | Real mobile number | **Paid** (roughly ₹0.15–0.50 per SMS in India; check current rates) | Very easy | India requires **DLT registration** for transactional SMS (sender ID + template). Providers: MSG91, Twilio, Fast2SMS. Supabase supports Twilio, MessageBird, Vonage, and Textlocal, and a custom provider via its Send SMS hook. |
| **WhatsApp OTP** | Real WhatsApp number | Paid per conversation (Meta pricing) | Very easy | Needs a WhatsApp Business API provider. Good later. |
| **Firebase Phone Auth** | Real mobile number | Small free quota, then paid; billing account needed | Very easy | Adds a second auth system next to Supabase, so we avoid it for now. |

> Free SMS OTP that keeps working at scale doesn't really exist in India. Every reliable route
> costs a little per SMS.

## 4.2 Recommended plan

**Phase 1 (free):**
1. **"Continue with Google"** (one tap) and **"Login with email code"** (6-digit OTP).
2. After login, a short **"Complete profile"** step: name + mobile number (required) + address.
3. **Anti-fake measures that cost nothing:**
   - The first order from a new account shows a badge **"New customer – call to confirm"** in the admin panel. The shop calls once, then ticks **"Phone verified"** on the profile.
   - Cash/UPI **on delivery** means fake orders don't lose money.
   - The admin can **block** an account.
   - Limit of N open orders per customer (in `place_order`).
   - Optional Cloudflare Turnstile (free CAPTCHA) on the login form.

**Phase 2 (when order volume grows):** add **SMS OTP for the mobile number** via MSG91 or Twilio,
using Supabase's phone auth or a custom SMS hook. Once verified, `phone_verified = true` is set
automatically and the manual phone call is no longer needed.

## 4.3 Sign-up flow (phase 1)

```
[Welcome]
   ├─ (G) Continue with Google ───────────────┐
   └─ ✉ Enter email → 6-digit code → verify ──┤
                                              ▼
                         [Complete profile] name, mobile, address
                                              ▼
                                       [Home / Order]
```

- Returning users stay logged in (Supabase refresh tokens). There's no password to remember.
- Login screen text in Hindi and English.

## 4.4 Staff / admin accounts

- The admin logs in the same way with their own email or Google account.
- Their `profiles.role` is set to `admin` once, by hand in the Supabase dashboard (or by SQL).
- The admin can promote helpers to `staff` from the admin panel.

## 4.5 Supabase settings checklist

- Auth → Providers: enable **Email** (OTP) and **Google** (OAuth client from Google Cloud Console, free).
- Auth → Email templates: change the magic link to a **{{ .Token }}** code template, branded "Jahangir Egg Center".
- Auth → SMTP: connect Resend or Brevo (free tier) with a sender like `orders@<your-domain>`.
- Auth → URL configuration: add the production site URL and `localhost` for development.
- Rate limits: keep the defaults, and add Turnstile if bots appear.
