# Options Learning Platform

A paper trading platform for classrooms. Students practice options and stock trading with a virtual portfolio. Teachers create classes, monitor student performance, and manage accounts.

## Setup

### 1. Supabase

Create a free project at [supabase.com](https://supabase.com) and run the SQL schema below in the SQL Editor.

### 2. Configure the app

Open `index.html` and replace the two placeholder values near the top of the script:

```js
const SUPABASE_URL  = "YOUR_SUPABASE_URL";   // → your project URL
const SUPABASE_ANON = "YOUR_SUPABASE_ANON_KEY"; // → your anon public key
```

Both values are in your Supabase project under **Settings → API**.

### 3. Deploy to Vercel

1. Push this repo to GitHub (must be public)
2. Go to [vercel.com](https://vercel.com) → New Project → import this repo
3. Framework preset: **Other**
4. Click Deploy — no build step needed

Your app goes live at `https://your-repo-name.vercel.app`.

### 4. First login

- Visit your live URL
- Click **Sign Up** → toggle **Teacher**
- Create your first class — a 6-character class code is generated automatically
- Share the code with students so they can sign up

### Setting yourself as Admin

In Supabase → Table Editor → `profiles` → find your row → change `role` to `admin`.

---

## Database Schema

Run this in Supabase → SQL Editor → New Query:

```sql
-- PROFILES
create table profiles (
  id uuid references auth.users on delete cascade primary key,
  email text,
  full_name text,
  role text not null default 'student',
  class_id uuid,
  starting_balance numeric default 25000,
  created_at timestamptz default now()
);

-- CLASSES
create table classes (
  id uuid default gen_random_uuid() primary key,
  teacher_id uuid references profiles(id) on delete cascade,
  name text not null,
  class_code text unique not null,
  starting_balance numeric default 25000,
  created_at timestamptz default now()
);

-- TRADES
create table trades (
  id uuid default gen_random_uuid() primary key,
  user_id uuid references profiles(id) on delete cascade,
  ticker text not null,
  strategy text not null,
  expiry date,
  strike_short numeric,
  strike_long numeric,
  premium numeric,
  contracts integer default 1,
  current_value numeric,
  status text default 'Open',
  open_date date,
  close_date date,
  stock_price numeric,
  cost_basis numeric,
  thesis text,
  notes text,
  div_amount numeric,
  div_ex_date date,
  div_pay_date date,
  div_count integer default 0,
  created_at timestamptz default now(),
  updated_at timestamptz default now()
);

-- STOCKS
create table stocks (
  id uuid default gen_random_uuid() primary key,
  user_id uuid references profiles(id) on delete cascade,
  ticker text not null,
  shares integer not null,
  buy_price numeric not null,
  buy_date date,
  current_price numeric,
  status text default 'Holding',
  sell_price numeric,
  sell_date date,
  notes text,
  created_at timestamptz default now()
);

-- ROW LEVEL SECURITY
alter table profiles enable row level security;
alter table trades   enable row level security;
alter table stocks   enable row level security;
alter table classes  enable row level security;

create policy "own profile" on profiles
  for all using (auth.uid() = id);

create policy "own trades" on trades
  for all using (auth.uid() = user_id);

create policy "own stocks" on stocks
  for all using (auth.uid() = user_id);

create policy "teachers see own classes" on classes
  for all using (auth.uid() = teacher_id);

create policy "students can read their class" on classes
  for select using (id = (select class_id from profiles where id = auth.uid()));

-- Auto-create profile on signup
create or replace function handle_new_user()
returns trigger as $$
begin
  insert into profiles (id, email)
  values (new.id, new.email);
  return new;
end;
$$ language plpgsql security definer;

create trigger on_auth_user_created
  after insert on auth.users
  for each row execute procedure handle_new_user();
```

---

## Updating the App

Make changes → save `index.html` → upload to GitHub → Vercel rebuilds in ~30 seconds. That's it.
