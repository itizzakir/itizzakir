# 3. Database Schema (Supabase / Postgres)

## 3.1 Entity diagram

```
auth.users 1──1 profiles 1──* addresses
                   │
                   1──* orders 1──* order_items *──1 product_packs *──1 products
                   │                                      │                 
                   1──* push_subscriptions                *──1 pack_types   
                   1──* notifications

shop_settings (single row)      price_history (audit of every price change)
```

**`product_packs`** is the sellable item: *product × pack type*, e.g. "White Egg – Tray".
It holds the current retail and wholesale price.

> Dealer, purchase, udhaar and stock tables (`parties`, `purchases`, `ledger_entries`,
> `counter_sales`, `stock_adjustments`) are described in [doc 10](10-dealers-and-credit-ledger.md#106-database-additions).

## 3.2 Tables (SQL draft)

This will become `supabase/migrations/0001_init.sql`.

```sql
-- ---------- Enums ----------
create type user_role    as enum ('customer', 'staff', 'admin');
create type customer_type as enum ('retail', 'wholesale');
create type order_status as enum ('placed','accepted','rejected','out_for_delivery','delivered','cancelled');
create type payment_method as enum ('cash','upi');

-- ---------- Profiles (1 row per auth user) ----------
create table profiles (
  id             uuid primary key references auth.users on delete cascade,
  full_name      text not null,
  phone          text,                       -- +91XXXXXXXXXX
  phone_verified boolean not null default false,
  role           user_role not null default 'customer',
  customer_type  customer_type not null default 'retail',
  is_blocked     boolean not null default false,
  preferred_lang text not null default 'hi',  -- 'hi' | 'en'
  created_at     timestamptz not null default now()
);

create table addresses (
  id          uuid primary key default gen_random_uuid(),
  user_id     uuid not null references profiles on delete cascade,
  label       text default 'Home',            -- Home / Shop / Hotel
  line1       text not null,
  landmark    text,
  area        text,
  city        text not null default 'Muzaffarpur',
  pincode     text not null default '842001',
  lat         double precision,
  lng         double precision,
  is_default  boolean not null default false
);

-- ---------- Catalogue ----------
create table products (
  id          uuid primary key default gen_random_uuid(),
  name_en     text not null,                  -- 'White Egg'
  name_hi     text not null,                  -- 'सफ़ेद अंडा'
  image_url   text,
  is_active   boolean not null default true,
  sort_order  int not null default 0
);

create table pack_types (
  id          text primary key,               -- 'tray' | 'half_carton' | 'carton' | 'piece'
  name_en     text not null,
  name_hi     text not null,
  eggs_per_pack int not null check (eggs_per_pack > 0),
  sort_order  int not null default 0
);

create table product_packs (
  id               uuid primary key default gen_random_uuid(),
  product_id       uuid not null references products on delete cascade,
  pack_type_id     text not null references pack_types,
  retail_price     numeric(10,2) not null check (retail_price >= 0),
  wholesale_price  numeric(10,2) not null check (wholesale_price >= 0),
  in_stock         boolean not null default true,
  updated_at       timestamptz not null default now(),
  unique (product_id, pack_type_id)
);

create table price_history (
  id               bigint generated always as identity primary key,
  product_pack_id  uuid not null references product_packs on delete cascade,
  retail_price     numeric(10,2) not null,
  wholesale_price  numeric(10,2) not null,
  changed_by       uuid references profiles,
  changed_at       timestamptz not null default now()
);

-- ---------- Orders ----------
create table orders (
  id               uuid primary key default gen_random_uuid(),
  order_no         bigint generated always as identity unique,   -- shown as #1042
  customer_id      uuid not null references profiles,
  status           order_status not null default 'placed',
  address_snapshot jsonb not null,           -- copy of address at order time
  customer_type    customer_type not null,   -- which price list was used
  subtotal         numeric(10,2) not null,
  delivery_charge  numeric(10,2) not null default 0,
  total            numeric(10,2) not null,
  total_eggs       int not null,
  payment_method   payment_method not null default 'cash',
  customer_note    text,
  reject_reason    text,
  placed_at        timestamptz not null default now(),
  accepted_at      timestamptz,
  out_for_delivery_at timestamptz,
  delivered_at     timestamptz,
  updated_at       timestamptz not null default now()
);

create table order_items (
  id               uuid primary key default gen_random_uuid(),
  order_id         uuid not null references orders on delete cascade,
  product_pack_id  uuid not null references product_packs,
  product_name     text not null,            -- snapshot
  pack_name        text not null,            -- snapshot
  eggs_per_pack    int not null,             -- snapshot
  quantity         int not null check (quantity > 0),
  unit_price       numeric(10,2) not null,   -- PRICE SNAPSHOT
  line_total       numeric(10,2) not null
);

-- ---------- Notifications ----------
create table push_subscriptions (
  id          uuid primary key default gen_random_uuid(),
  user_id     uuid not null references profiles on delete cascade,
  kind        text not null check (kind in ('webpush','expo')),
  endpoint    text not null unique,          -- web push endpoint OR expo token
  keys        jsonb,                         -- p256dh/auth for web push
  created_at  timestamptz not null default now()
);

create table notifications (
  id          uuid primary key default gen_random_uuid(),
  user_id     uuid not null references profiles on delete cascade,
  title       text not null,
  body        text not null,
  order_id    uuid references orders on delete cascade,
  read_at     timestamptz,
  created_at  timestamptz not null default now()
);

-- ---------- Shop settings (single row) ----------
create table shop_settings (
  id                  int primary key default 1 check (id = 1),
  is_open             boolean not null default true,
  min_order_value     numeric(10,2) not null default 0,
  delivery_charge     numeric(10,2) not null default 0,
  free_delivery_above numeric(10,2),
  closed_message      text,
  updated_at          timestamptz not null default now()
);
```

## 3.3 Helper & security

```sql
create or replace function is_staff() returns boolean
language sql stable security definer set search_path = public as $$
  select exists (select 1 from profiles where id = auth.uid() and role in ('staff','admin'));
$$;

alter table profiles            enable row level security;
alter table addresses           enable row level security;
alter table products            enable row level security;
alter table pack_types          enable row level security;
alter table product_packs       enable row level security;
alter table price_history       enable row level security;
alter table orders              enable row level security;
alter table order_items         enable row level security;
alter table push_subscriptions  enable row level security;
alter table notifications       enable row level security;
alter table shop_settings       enable row level security;

-- Catalogue & settings: everyone can read, only staff can write
create policy "read catalogue"  on products      for select using (true);
create policy "read packs"      on pack_types    for select using (true);
create policy "read prices"     on product_packs for select using (true);
create policy "read settings"   on shop_settings for select using (true);
create policy "staff write products" on products      for all using (is_staff()) with check (is_staff());
create policy "staff write packs"    on product_packs for all using (is_staff()) with check (is_staff());
create policy "staff write settings" on shop_settings for all using (is_staff()) with check (is_staff());

-- Profiles: own row (cannot change own role/type), staff all
create policy "own profile read"   on profiles for select using (id = auth.uid() or is_staff());
create policy "own profile update" on profiles for update using (id = auth.uid())
  with check (id = auth.uid());   -- role/customer_type changes blocked by trigger below
create policy "staff profile update" on profiles for update using (is_staff());

-- Addresses, subscriptions, notifications: own rows only
create policy "own addresses" on addresses          for all using (user_id = auth.uid()) with check (user_id = auth.uid());
create policy "own subs"      on push_subscriptions for all using (user_id = auth.uid()) with check (user_id = auth.uid());
create policy "own notifs"    on notifications      for select using (user_id = auth.uid());
create policy "mark read"     on notifications      for update using (user_id = auth.uid());

-- Orders: customers read their own; inserts ONLY via place_order() RPC
create policy "read own orders"   on orders      for select using (customer_id = auth.uid() or is_staff());
create policy "read own items"    on order_items for select using (
  exists (select 1 from orders o where o.id = order_id and (o.customer_id = auth.uid() or is_staff())));
create policy "staff update orders" on orders for update using (is_staff());
-- no insert policy on orders/order_items => clients cannot insert directly
```

A trigger stops customers from changing their own `role`, `customer_type`, `is_blocked`,
or `phone_verified`. Only staff (or the service role) can change those.

## 3.4 Key server functions (RPC)

| Function | What it does |
|----------|--------------|
| `place_order(items jsonb, address_id uuid, payment payment_method, note text)` | `security definer`. Checks the user isn't blocked and the shop is open. Loads **current** prices for the user's `customer_type`, computes totals, applies minimum order and delivery charge, inserts the order + items with price snapshots, and returns the order. |
| `cancel_order(order_id)` | Customer can cancel only while `status = 'placed'`. |
| `set_order_status(order_id, status, reason)` | Staff only. Validates allowed transitions (e.g. you can't go `placed → delivered`) and sets timestamps. |
| `update_prices(changes jsonb)` | Staff only. Bulk-updates the Price Board in one transaction and writes `price_history`. |
| `handle_new_user()` trigger | Creates the `profiles` row when someone signs up. |

## 3.5 Seed data (example; confirm prices with the shop)

```sql
insert into pack_types values
 ('tray',        'Tray (30)',          'ट्रे (30)',            30,  1),
 ('half_carton', 'Half Carton (105)',  'आधा कार्टन (105)',     105, 2),
 ('carton',      'Carton (210)',       'कार्टन (210)',         210, 3);

insert into shop_settings (id) values (1);
```
