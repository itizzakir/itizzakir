# 10. Dealers (Purchases) & Customer Credit (Udhaar) Ledger

This module replaces the paper *khata* (account book). It records:

1. **Dealers / suppliers:** how many cartons we bought, at what price, how much **advance** we paid,
   and how much we **still have to pay**.
2. **Customer udhaar:** customers who take eggs without paying now, how much they owe, and when they paid.

It lives in the admin panel (`/admin/khata`) and works from a phone.

---

## 10.1 One idea for both: the party ledger

Every dealer and every credit customer is a **party** with a running **balance**.
Each entry in their account either **increases** or **decreases** that balance.

| Party | Balance means | Increases balance | Decreases balance |
|-------|---------------|-------------------|-------------------|
| **Dealer** | Amount **we owe** the dealer | Purchase (cartons received) | Payment / advance we paid |
| **Customer** | Amount the customer **owes us** | Sale on credit (udhaar) | Payment received from customer |

- **Positive balance:** dealer → *"To pay ₹12,400"*; customer → *"Due ₹850"*.
- **Negative balance:** money paid in advance → dealer *"Advance ₹5,000"*; customer *"Advance ₹200"*.
- **Zero:** *"Settled ✅"*.

### Example: dealer "Ravi Poultry"

| Date | Entry | Detail | Change | Balance | Shown as |
|------|-------|--------|--------|---------|----------|
| 1 Nov | Advance paid | Cash | −10,000 | −10,000 | Advance ₹10,000 |
| 2 Nov | Purchase | 20 cartons × ₹1,150 | +23,000 | 13,000 | To pay ₹13,000 |
| 2 Nov | Transport | Tempo charge | +400 | 13,400 | To pay ₹13,400 |
| 4 Nov | Payment | UPI | −8,000 | 5,400 | To pay ₹5,400 |
| 5 Nov | Return / damage | 1 carton broken, credit note | −1,150 | 4,250 | To pay ₹4,250 |

### Example: customer "Sharma Hotel" (udhaar)

| Date | Entry | Detail | Change | Balance | Shown as |
|------|-------|--------|--------|---------|----------|
| 3 Nov | Sale on credit | Order #1042: 2 cartons | +2,460 | 2,460 | Due ₹2,460 |
| 4 Nov | Payment received | Cash | −1,500 | 960 | Due ₹960 |
| 6 Nov | Sale on credit | Counter: 3 trays | +540 | 1,500 | Due ₹1,500 |

The balance is **always calculated from the entries**, never typed in by hand, so it can't go wrong.
Mistakes are fixed by adding a correction entry or voiding the entry (with a reason), never
by silently deleting it. That keeps a full audit trail.

---

## 10.2 Dealer features

**Dealer list:** name, firm, phone, and a big balance chip (red *To pay* / green *Advance*).
Sorted by largest amount to pay. The top shows a total: *"Total to pay all dealers: ₹48,300"*.

**Add purchase (stock in)**, one screen:

| Field | Example |
|-------|---------|
| Dealer | Ravi Poultry |
| Date | today (editable) |
| Egg type | White |
| Lines | 20 × **Carton** @ ₹1,150, 10 × **Tray** @ ₹168 |
| Extra charges | Transport ₹400, Loading ₹100 |
| Broken / short on arrival | 30 eggs (optional) |
| Bill / challan no. + photo | optional, photo saved to Storage |
| Paid now | ₹5,000 cash (optional, creates a payment entry too) |

The screen shows *"Total ₹24,580 · 4,500 eggs · cost ₹5.46/egg"* live while typing.

**Add payment / advance:** amount, date, mode (Cash / UPI / Bank / Cheque), reference no., note.
There's no separate "advance" type. A payment made before purchases simply leaves the balance negative,
which shows as *Advance*. You can still tag it "advance" for clarity in the list.

**Dealer statement:** date-range filter, opening balance, all entries, closing balance.
**Share as PDF / WhatsApp** to settle accounts with the dealer.

**Price history per dealer:** see how the carton rate changed over time and compare dealers.

---

## 10.3 Customer udhaar features

**Who can have udhaar:**
- Online customers (linked to their app account).
- **Offline / walk-in customers** with no app account, created by the shop with just name + phone.
  If they sign up later with the same phone, the shop can link the account.

**How credit gets recorded:**

| Situation | What happens |
|-----------|--------------|
| Online order delivered, customer didn't pay | When marking *Delivered*, staff picks **Paid cash / Paid UPI / Udhaar**. Choosing *Udhaar* adds a "sale on credit" entry automatically. Part payment is allowed (e.g. paid ₹1,000 of ₹2,460). |
| Sale at the counter on credit | **Quick entry:** customer, packs and quantity (or just an amount), and the note. |
| Customer pays later | **Receive payment:** amount, mode, date. |
| Customer has an advance | The next order uses it automatically (balance goes down). |

**Credit limit (optional):** for example ₹5,000 for Sharma Hotel. If a new order would push the due above the
limit, staff sees a warning, and online checkout can block "Udhaar" and allow only cash/UPI.

**Due list:** all customers with dues, the biggest first, with **"days since oldest unpaid"**,
e.g. *"₹1,500 · 12 days"*. The top shows *"Total market udhaar: ₹36,750"*.

**Reminders (free):** a **"Send reminder on WhatsApp"** button opens WhatsApp with a ready message
(via a `wa.me` link, which costs nothing):

> नमस्ते शर्मा जी, जहांगीर एग सेंटर में आपका बकाया ₹1,500 है। कृपया जल्द भुगतान करें। धन्यवाद 🙏

**Customer view in the app:** linked customers see **"My Khata"** with their due, entries, and statement.
That builds trust, and reminders can stay polite.

---

## 10.4 Stock (comes free with purchases)

Since purchases (eggs in) and orders/counter sales (eggs out) are both recorded, the app can show
**current stock**:

```
Stock (eggs) = purchased − sold (online + counter) − damaged/broken ± manual adjustment
```

- Shown as cartons + trays + eggs, e.g. *"42 cartons, 3 trays (8,910 eggs)"*.
- **Low-stock alert** below a level set by the shop.
- **Damage entry** for broken eggs. These are common, and it's useful to track the loss.

## 10.5 Reports

| Report | Shows |
|--------|-------|
| Daily summary | Eggs bought, eggs sold, cash in, UPI in, new udhaar, udhaar recovered |
| Profit (approx.) | Sales − average purchase cost of eggs sold − damage, per day or month |
| Dealer payable | Each dealer's balance and total |
| Customer receivable | Each customer's due, age of dues |
| Purchase rate trend | Carton purchase price over time (by dealer) |

All reports can be exported as **CSV / PDF**.

---

## 10.6 Database additions

Add these to the schema in [doc 3](03-database-schema.md) (migration `0002_ledger.sql`):

```sql
create type party_kind  as enum ('dealer', 'customer');
create type ledger_type as enum (
  'opening',            -- opening balance when starting to use the app
  'purchase',           -- dealer: eggs received            (+)
  'purchase_charge',    -- dealer: transport/loading etc.   (+)
  'purchase_return',    -- dealer: damaged/returned credit  (−)
  'payment_to_dealer',  -- dealer: payment or advance       (−)
  'credit_sale',        -- customer: udhaar                 (+)
  'payment_received',   -- customer: payment                (−)
  'adjustment'          -- either: correction, discount     (±)
);
create type pay_mode as enum ('cash','upi','bank','cheque','other');

-- A dealer or a credit customer
create table parties (
  id             uuid primary key default gen_random_uuid(),
  kind           party_kind not null,
  name           text not null,
  firm_name      text,
  phone          text,
  address        text,
  profile_id     uuid unique references profiles,   -- link to app account (customers)
  credit_limit   numeric(12,2),                     -- customers only, optional
  notes          text,
  is_active      boolean not null default true,
  created_at     timestamptz not null default now()
);

-- Purchase bills from dealers (details); the money effect goes into ledger_entries
create table purchases (
  id             uuid primary key default gen_random_uuid(),
  dealer_id      uuid not null references parties,
  purchase_date  date not null default current_date,
  bill_no        text,
  bill_photo_url text,
  extra_charges  numeric(12,2) not null default 0,
  damaged_eggs   int not null default 0,
  total_eggs     int not null,
  total_amount   numeric(12,2) not null,   -- items + extra charges
  note           text,
  created_by     uuid references profiles,
  created_at     timestamptz not null default now()
);

create table purchase_items (
  id             uuid primary key default gen_random_uuid(),
  purchase_id    uuid not null references purchases on delete cascade,
  product_id     uuid not null references products,
  pack_type_id   text not null references pack_types,
  quantity       int not null check (quantity > 0),
  eggs_per_pack  int not null,              -- snapshot
  unit_price     numeric(10,2) not null,    -- price per carton/tray paid to dealer
  line_total     numeric(12,2) not null
);

-- The khata: every money movement for every party
create table ledger_entries (
  id             uuid primary key default gen_random_uuid(),
  party_id       uuid not null references parties,
  entry_date     date not null default current_date,
  type           ledger_type not null,
  amount         numeric(12,2) not null,    -- signed: + increases balance, − decreases
  pay_mode       pay_mode,                  -- for payments
  reference      text,                      -- UPI ref / cheque no.
  purchase_id    uuid references purchases,
  order_id       uuid references orders,
  counter_sale_id uuid,                     -- see counter_sales below
  is_advance     boolean not null default false,  -- just a label for dealer payments
  note           text,
  voided_at      timestamptz,               -- never delete; void with reason
  void_reason    text,
  created_by     uuid references profiles,
  created_at     timestamptz not null default now(),
  check (
    (type in ('opening','adjustment'))
    or (type in ('purchase','purchase_charge','credit_sale') and amount > 0)
    or (type in ('purchase_return','payment_to_dealer','payment_received') and amount < 0)
  )
);
create index on ledger_entries (party_id, entry_date);

-- Sales at the shop counter (cash, UPI or udhaar) so stock and reports are complete
create table counter_sales (
  id             uuid primary key default gen_random_uuid(),
  party_id       uuid references parties,   -- null = anonymous cash customer
  sale_date      date not null default current_date,
  items          jsonb not null,            -- [{product_pack_id, qty, unit_price, eggs}]
  total_eggs     int not null,
  total_amount   numeric(12,2) not null,
  paid_amount    numeric(12,2) not null default 0,
  pay_mode       pay_mode,
  created_by     uuid references profiles,
  created_at     timestamptz not null default now()
);

-- Broken eggs, stock count corrections
create table stock_adjustments (
  id             uuid primary key default gen_random_uuid(),
  product_id     uuid not null references products,
  eggs           int not null,              -- negative = loss
  reason         text not null,             -- 'broken', 'stock count', ...
  adj_date       date not null default current_date,
  created_by     uuid references profiles,
  created_at     timestamptz not null default now()
);

-- Live balances (ignores voided entries)
create view party_balances with (security_invoker = true) as
select p.*,
       coalesce(sum(e.amount) filter (where e.voided_at is null), 0) as balance,
       max(e.entry_date)                                              as last_entry_date
from parties p
left join ledger_entries e on e.party_id = p.id
group by p.id;
```

**Changes to existing tables:**

```sql
create type payment_status as enum ('unpaid','paid','partial','credit');
alter table orders add column payment_status payment_status not null default 'unpaid',
                   add column paid_amount    numeric(10,2) not null default 0;
alter table product_packs add column eggs_low_stock_alert int;  -- optional
```

**Server functions (RPC, all `security definer`, staff/admin only):**

| Function | Does |
|----------|------|
| `record_purchase(dealer_id, items, extra_charges, damaged_eggs, paid_now, pay_mode, …)` | Inserts the purchase + items and adds `purchase` (+ `purchase_charge`) ledger entries. Adds a `payment_to_dealer` entry if `paid_now > 0`. One transaction. |
| `record_payment(party_id, amount, pay_mode, reference, is_advance, note)` | Adds a `payment_to_dealer` or `payment_received` entry depending on the party kind. |
| `mark_order_delivered(order_id, paid_amount, pay_mode)` | Sets status and payment status. Any unpaid remainder becomes a `credit_sale` on the customer's party (created automatically if missing). |
| `record_counter_sale(party_id, items, paid_amount, pay_mode)` | Inserts the counter sale, and a `credit_sale` entry for the unpaid part. |
| `void_ledger_entry(entry_id, reason)` | Admin only. Marks the entry voided, which removes it from the balance. |

**Security (RLS):**
- `parties`, `purchases`, `purchase_items`, `ledger_entries`, `counter_sales`, `stock_adjustments`:
  **staff/admin only**. Dealer purchase prices are visible to **admin only** (see open question 26).
- A customer can **read** their own `ledger_entries` when `parties.profile_id = auth.uid()` (for "My Khata").
- No `update` or `delete` policies on `ledger_entries`. Changes go only through the RPCs above.

---

## 10.7 Admin screens added

```
/admin/khata
 ├─ Dealers          list → dealer page (balance, entries, + Purchase, + Payment, Statement)
 ├─ Customers udhaar list → customer page (due, entries, + Udhaar, + Payment, WhatsApp reminder)
 ├─ Counter sale     quick sale screen (cash / UPI / udhaar)
 ├─ Stock            current stock, damage entry, adjustments
 └─ Reports          daily summary, profit, payables, receivables
```

**Fast entry on phone:**
- A floating **"+"** button with *Purchase · Payment to dealer · Udhaar · Payment received · Counter sale*.
- A number pad UI with recent parties at the top, so most entries take under 15 seconds.
- Every entry shows **who added it and when**, which helps when staff also make entries.

## 10.8 Getting started: moving from the paper khata

1. Add each dealer and customer with an **opening balance** (one `opening` entry on the start date).
2. From that day, record everything in the app.
3. For the first week, keep the paper khata in parallel and compare the totals at the end of each day.
