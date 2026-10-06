# OnlyGames — Digital Game Storefront & Analytics Database

[![Supabase](https://img.shields.io/badge/Supabase-PostgreSQL_Backend-3ECF8E?style=for-the-badge&logo=supabase)](https://supabase.com)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15.0-4169E1?style=for-the-badge&logo=postgresql)](https://www.postgresql.org/)
[![License](https://img.shields.io/badge/License-MIT-blue?style=for-the-badge)](LICENSE)

An end-to-end digital video game distribution platform backend built on **PostgreSQL** and **Supabase**. OnlyGames models complex e-commerce and social platform operations, including multi-item checkout, transaction histories, peer-to-peer game trading, user reviews, wishlists, and metadata categorization.

---

## Project Overview

* **3NF Normalization**: Designed a fully normalized relational database schema spanning 16 core entities to eliminate redundancy and maintain data integrity.
* **Authentication Integration**: Linked custom `profiles` directly to Supabase Auth (`auth.users`) for seamless user identity management.
* **Financial & Inventory Rules**: Handled transaction histories (`orders`, `order_details`), calculated item subtotals with discounts, and digital library access control (`user_library`).
* **Peer-to-Peer Trading**: Implemented a dedicated `trades` table supporting game offer/request exchanges between users.

---

## Database Architecture & ER Diagram

The relational schema is hosted on **Supabase** with strict foreign key cascading, check constraints, and indexed join paths.

### Entity Relationship Diagram

<p float="left">
  <img src="./assets/er_diagram_overview.png" width="45%" alt="ER Diagram Overview" />
  <img src="./assets/er_diagram_detail.png" width="52%" alt="ER Diagram Detailed View" />
</p>

* **Supabase Backend Link**: [Access Supabase Dashboard / Project API](https://supabase.com)

---

## Key System Features

1. **Users & Profiles**:
   * Extends `auth.users` into a custom `profiles` table.
   * Tracks user roles (`user`, `admin`) and social connections (`friends`).

2. **Game Catalog & Metadata**:
   * Supports complex game cataloging across `categories`, `developers`, `publishers`, and `tags` (via `game_tags` junction table).
   * Detailed system hardware specs (`system_requirements`) and active price promotions (`discounts`).

3. **Orders & Digital Ownership**:
   * Multi-item order processing using `orders` and `order_details` with auto-calculated subtotals.
   * Clear separation between purchase history (`orders`) and active game entitlement (`user_library`).

4. **Social & Trading Platform**:
   * Friend request state management (`Pending`, `Accepted`, `Blocked`).
   * Direct game-for-game trade proposals between users (`trades`).

5. **User Engagement**:
   * Instant cart queuing (`cart_items`) and saved items (`wishlists`).
   * Verified user game ratings and written reviews (`reviews`).

---

## Database Schema DDL (PostgreSQL / Supabase)

<details>
<summary><b>Click to expand full SQL DDL Script</b></summary>

```sql
CREATE TABLE public.categories (
  category_id integer GENERATED ALWAYS AS IDENTITY NOT NULL,
  category_name text NOT NULL UNIQUE,
  description text,
  CONSTRAINT categories_pkey PRIMARY KEY (category_id)
);

CREATE TABLE public.developers (
  developer_id integer GENERATED ALWAYS AS IDENTITY NOT NULL,
  developer_name text NOT NULL,
  country text,
  website text,
  founded_year integer,
  CONSTRAINT developers_pkey PRIMARY KEY (developer_id)
);

CREATE TABLE public.publishers (
  publisher_id integer GENERATED ALWAYS AS IDENTITY NOT NULL,
  publisher_name text NOT NULL,
  country text,
  website text,
  CONSTRAINT publishers_pkey PRIMARY KEY (publisher_id)
);

CREATE TABLE public.games (
  id uuid NOT NULL DEFAULT gen_random_uuid(),
  title text NOT NULL,
  genre text NOT NULL,
  price text NOT NULL,
  image text,
  created_at timestamp with time zone DEFAULT now(),
  description text,
  category_id integer,
  developer_id integer,
  publisher_id integer,
  release_date date,
  minimum_age integer DEFAULT 0,
  is_active boolean DEFAULT true,
  cover_image_url text,
  CONSTRAINT games_pkey PRIMARY KEY (id),
  CONSTRAINT games_category_id_fkey FOREIGN KEY (category_id) REFERENCES public.categories(category_id),
  CONSTRAINT games_developer_id_fkey FOREIGN KEY (developer_id) REFERENCES public.developers(developer_id),
  CONSTRAINT games_publisher_id_fkey FOREIGN KEY (publisher_id) REFERENCES public.publishers(publisher_id)
);

CREATE TABLE public.profiles (
  id uuid NOT NULL,
  username text NOT NULL UNIQUE,
  role text NOT NULL DEFAULT 'user'::text CHECK (role = ANY (ARRAY['user'::text, 'admin'::text])),
  created_at timestamp with time zone DEFAULT now(),
  CONSTRAINT profiles_pkey PRIMARY KEY (id),
  CONSTRAINT profiles_id_fkey FOREIGN KEY (id) REFERENCES auth.users(id)
);

CREATE TABLE public.cart_items (
  id bigint GENERATED ALWAYS AS IDENTITY NOT NULL,
  user_id uuid NOT NULL,
  game_id uuid NOT NULL,
  created_at timestamp with time zone DEFAULT now(),
  CONSTRAINT cart_items_pkey PRIMARY KEY (id),
  CONSTRAINT cart_items_user_id_fkey FOREIGN KEY (user_id) REFERENCES public.profiles(id),
  CONSTRAINT cart_items_game_id_fkey FOREIGN KEY (game_id) REFERENCES public.games(id)
);

CREATE TABLE public.user_library (
  id bigint GENERATED ALWAYS AS IDENTITY NOT NULL,
  user_id uuid NOT NULL,
  game_id uuid NOT NULL,
  purchased_at timestamp with time zone DEFAULT now(),
  CONSTRAINT user_library_pkey PRIMARY KEY (id),
  CONSTRAINT user_library_user_id_fkey FOREIGN KEY (user_id) REFERENCES public.profiles(id),
  CONSTRAINT user_library_game_id_fkey FOREIGN KEY (game_id) REFERENCES public.games(id)
);

CREATE TABLE public.system_requirements (
  requirement_id integer GENERATED ALWAYS AS IDENTITY NOT NULL,
  game_id uuid NOT NULL UNIQUE,
  min_os text,
  min_cpu text,
  min_ram_gb integer,
  min_gpu text,
  min_storage_gb integer,
  rec_os text,
  rec_cpu text,
  rec_ram_gb integer,
  rec_gpu text,
  rec_storage_gb integer,
  CONSTRAINT system_requirements_pkey PRIMARY KEY (requirement_id),
  CONSTRAINT system_requirements_game_id_fkey FOREIGN KEY (game_id) REFERENCES public.games(id)
);

CREATE TABLE public.discounts (
  discount_id integer GENERATED ALWAYS AS IDENTITY NOT NULL,
  game_id uuid NOT NULL,
  discount_percent numeric NOT NULL CHECK (discount_percent >= 0::numeric AND discount_percent <= 100::numeric),
  start_date date NOT NULL,
  end_date date NOT NULL,
  CONSTRAINT discounts_pkey PRIMARY KEY (discount_id),
  CONSTRAINT discounts_game_id_fkey FOREIGN KEY (game_id) REFERENCES public.games(id)
);

CREATE TABLE public.orders (
  order_id integer GENERATED ALWAYS AS IDENTITY NOT NULL,
  user_id uuid NOT NULL,
  order_date timestamp with time zone NOT NULL DEFAULT now(),
  total_amount numeric NOT NULL CHECK (total_amount >= 0::numeric),
  payment_method text CHECK (payment_method = ANY (ARRAY['CreditCard'::text, 'DebitCard'::text, 'PayPal'::text, 'WalletBalance'::text, 'GiftCard'::text])),
  status text NOT NULL DEFAULT 'Pending'::text CHECK (status = ANY (ARRAY['Pending'::text, 'Completed'::text, 'Refunded'::text, 'Cancelled'::text])),
  CONSTRAINT orders_pkey PRIMARY KEY (order_id),
  CONSTRAINT orders_user_id_fkey FOREIGN KEY (user_id) REFERENCES public.profiles(id)
);

CREATE TABLE public.order_details (
  order_detail_id integer GENERATED ALWAYS AS IDENTITY NOT NULL,
  order_id integer NOT NULL,
  game_id uuid NOT NULL,
  quantity integer NOT NULL DEFAULT 1 CHECK (quantity >= 1),
  unit_price numeric NOT NULL,
  discount_applied numeric NOT NULL DEFAULT 0,
  subtotal numeric DEFAULT (((quantity)::numeric * unit_price) * ((1)::numeric - (discount_applied / 100.0))),
  CONSTRAINT order_details_pkey PRIMARY KEY (order_detail_id),
  CONSTRAINT order_details_order_id_fkey FOREIGN KEY (order_id) REFERENCES public.orders(order_id),
  CONSTRAINT order_details_game_id_fkey FOREIGN KEY (game_id) REFERENCES public.games(id)
);

CREATE TABLE public.tags (
  tag_id integer GENERATED ALWAYS AS IDENTITY NOT NULL,
  tag_name text NOT NULL UNIQUE,
  CONSTRAINT tags_pkey PRIMARY KEY (tag_id)
);

CREATE TABLE public.game_tags (
  game_id uuid NOT NULL,
  tag_id integer NOT NULL,
  CONSTRAINT game_tags_pkey PRIMARY KEY (game_id, tag_id),
  CONSTRAINT game_tags_game_id_fkey FOREIGN KEY (game_id) REFERENCES public.games(id),
  CONSTRAINT game_tags_tag_id_fkey FOREIGN KEY (tag_id) REFERENCES public.tags(tag_id)
);

CREATE TABLE public.wishlists (
  user_id uuid NOT NULL,
  game_id uuid NOT NULL,
  added_date timestamp with time zone DEFAULT now(),
  CONSTRAINT wishlists_pkey PRIMARY KEY (user_id, game_id),
  CONSTRAINT wishlists_user_id_fkey FOREIGN KEY (user_id) REFERENCES public.profiles(id),
  CONSTRAINT wishlists_game_id_fkey FOREIGN KEY (game_id) REFERENCES public.games(id)
);

CREATE TABLE public.reviews (
  review_id integer GENERATED ALWAYS AS IDENTITY NOT NULL,
  user_id uuid NOT NULL,
  game_id uuid NOT NULL,
  rating integer CHECK (rating >= 1 AND rating <= 5),
  comment text,
  created_at timestamp with time zone DEFAULT now(),
  CONSTRAINT reviews_pkey PRIMARY KEY (review_id),
  CONSTRAINT reviews_user_id_fkey FOREIGN KEY (user_id) REFERENCES public.profiles(id),
  CONSTRAINT reviews_game_id_fkey FOREIGN KEY (game_id) REFERENCES public.games(id)
);

CREATE TABLE public.friends (
  friendship_id integer GENERATED ALWAYS AS IDENTITY NOT NULL,
  user_id1 uuid NOT NULL,
  user_id2 uuid NOT NULL,
  status text NOT NULL DEFAULT 'Pending'::text CHECK (status = ANY (ARRAY['Pending'::text, 'Accepted'::text, 'Blocked'::text])),
  created_at timestamp with time zone NOT NULL DEFAULT now(),
  CONSTRAINT friends_pkey PRIMARY KEY (friendship_id),
  CONSTRAINT friends_user_id1_fkey FOREIGN KEY (user_id1) REFERENCES public.profiles(id),
  CONSTRAINT friends_user_id2_fkey FOREIGN KEY (user_id2) REFERENCES public.profiles(id)
);

CREATE TABLE public.trades (
  id uuid NOT NULL DEFAULT gen_random_uuid(),
  sender_id uuid,
  receiver_id uuid,
  offered_game text,
  requested_game text,
  status text DEFAULT 'pending'::text,
  created_at timestamp without time zone DEFAULT now(),
  offered_game_id uuid,
  requested_game_id uuid,
  CONSTRAINT trades_pkey PRIMARY KEY (id),
  CONSTRAINT trades_sender_id_fkey FOREIGN KEY (sender_id) REFERENCES auth.users(id),
  CONSTRAINT trades_receiver_id_fkey FOREIGN KEY (receiver_id) REFERENCES auth.users(id),
  CONSTRAINT trades_offered_game_id_fkey FOREIGN KEY (offered_game_id) REFERENCES public.games(id),
  CONSTRAINT trades_requested_game_id_fkey FOREIGN KEY (requested_game_id) REFERENCES public.games(id)
);
