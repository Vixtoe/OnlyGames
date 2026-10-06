# OnlyGames

![PostgreSQL](https://img.shields.io/badge/PostgreSQL-Database-336791?style=flat-square&logo=postgresql&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase-Platform-3ECF8E?style=flat-square&logo=supabase&logoColor=white)
![SQL](https://img.shields.io/badge/SQL-DDL%20%2F%20DML-4479A1?style=flat-square)
![3NF](https://img.shields.io/badge/Design-3NF%20Normalized-blue?style=flat-square)

> A relational database design for digital game distribution, purchasing, and peer-to-peer game exchanges.

---

## Project Overview

OnlyGames addresses flexibility limitations in current digital gaming platforms by extending traditional store functionality (purchasing, reviews, library management) with a peer-to-peer game exchange feature. The database is a normalized 3NF relational schema designed to enforce data integrity, track game ownership transfers, and keep purchase history separate from current ownership.

This was a university database course project.

---

## My Contribution

* Designed the relational schema and normalized it to 3NF, including the separation of `orders`, `user_library`, and `trades`.
* Wrote the SQL DDL scripts, constraints, and the main analytical queries for PostgreSQL on Supabase.
* Developed the project documentation and presentation structure.

---

## ER Diagram

erDiagram
    auth_users ||--|| profiles : "extends"
    profiles ||--o{ orders : places
    profiles ||--o{ reviews : writes
    profiles ||--o{ user_library : owns
    profiles ||--o{ cart_items : has
    profiles ||--o{ wishlists : saves
    profiles ||--|| wallets : has
    profiles ||--o{ friends : "user_id1 / user_id2"
    profiles ||--o{ trades : "sender_id / receiver_id"
    orders ||--o{ order_details : contains
    games ||--o{ order_details : "ordered in"
    games ||--o{ reviews : receives
    games ||--o{ user_library : "owned in"
    games ||--o{ cart_items : "added to"
    games ||--o{ wishlists : "saved in"
    games ||--o{ discounts : has
    games ||--o{ system_requirements : requires
    games ||--o{ game_tags : tagged
    tags ||--o{ game_tags : labels
    games ||--o{ trades : "offered_game / requested_game"
    categories ||--o{ games : groups
    developers ||--o{ games : develops
    publishers ||--o{ games : publishes
---

## Key System Features

* **Users and Wallets:** Models user profiles, roles, friends lists, and wallet balances.
* **Game Catalog and Metadata:** Supports categories, developers, publishers, tags, system requirements, and discounts.
* **Orders and Transactions:** Models carts, wishlists, multi-item orders, and fixed order history.
* **Library and Access Control:** Separates order records from active ownership (`user_library`) so ownership can change over time.
* **Peer-to-Peer Game Exchange:** Models trade requests between users, allowed only when a game's `exchange_allowed` flag permits it.
* **Interaction:** Supports ratings, written reviews, and friend connections.

---

## Database Architecture and Relational Schema

The database runs on **PostgreSQL via Supabase** and follows Third Normal Form (3NF) to reduce redundancy and prevent update anomalies.

### Core Entities and Keys

* `profiles` (`id` PK, `username`, `email`, `role`)
* `games` (`id` PK, `title`, `genre`, `price`, `category_id` FK, `developer_id` FK, `publisher_id` FK, `exchange_allowed`)
* `categories` (`category_id` PK, `category_name`)
* `developers` (`developer_id` PK, `developer_name`)
* `publishers` (`publisher_id` PK, `publisher_name`)
* `orders` (`order_id` PK, `user_id` FK, `order_date`, `total_amount`, `status`)
* `order_details` (`order_detail_id` PK, `order_id` FK, `game_id` FK, `quantity`, `unit_price`)
* `user_library` (`id` PK, `user_id` FK, `game_id` FK, `purchased_at`)
* `reviews` (`review_id` PK, `user_id` FK, `game_id` FK, `rating`, `comment`)
* `cart_items` (`id` PK, `user_id` FK, `game_id` FK)
* `wishlists` (`user_id` PK/FK, `game_id` PK/FK)
* `tags` (`tag_id` PK, `tag_name`)
* `game_tags` (`game_id` PK/FK, `tag_id` PK/FK)
* `trades` (`trade_id` PK, `sender_id` FK, `receiver_id` FK, `offered_game_id` FK, `requested_game_id` FK, `status`)
* `discounts` (`discount_id` PK, `game_id` FK, `discount_percent`, `start_date`, `end_date`)
* `friends` (`friendship_id` PK, `user_id1` FK, `user_id2` FK, `status`)
* `system_requirements` (`requirement_id` PK, `game_id` FK)
* `wallets` (`user_id` PK/FK, `balance`)

---

## Design Decisions

* **Order vs. Library separation:** `orders` keep the permanent purchase record, while `user_library` holds current ownership, which allows trades without rewriting history.
* **Developer exchange control:** An `exchange_allowed` flag on each game lets publishers control whether it can be traded.
* **Bridge tables:** `game_tags` and `order_details` handle many-to-many relationships and multi-item orders.
* **Price snapshot:** `order_details.unit_price` stores the price at purchase time so later price changes or discounts do not alter history.
* **Integrity constraints:** Foreign keys with `NOT NULL`, `UNIQUE`, and `CHECK` constraints [confirm each type in schema.sql].

---

## Example SQL Queries

```sql
-- Total spending per user
[PASTE a real query from sql/queries.sql, e.g. SUM over orders grouped by user]

-- Top-selling games
[PASTE a real COUNT query]
```

Other queries in `sql/queries.sql`: catalog lookup (games joined with categories), user purchase history, and pending trade requests between users.

---

## Tech Stack

* **Database:** PostgreSQL on Supabase
* **Language:** SQL (DDL, DML, constraints)
* **Design:** ER modeling, 3NF normalization

---

## Run the Schema

1. Create a free Supabase project.
2. Open the SQL Editor and run `sql/schema.sql`.
3. Run the queries in `sql/queries.sql` (add sample data first if the file does not include inserts).

---

## Documentation

* **Presentation:** [View slides (PDF)](docs/presentation.pdf) [confirm path and remove any slide showing student IDs]

---

## Project Structure

```text
onlygames/
|-- docs/
|   |-- er-diagram.png
|   |-- relational-schema.png
|   `-- presentation.pdf
|-- sql/
|   |-- schema.sql
|   `-- queries.sql
`-- README.md
```
