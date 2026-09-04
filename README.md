# OnlyGames

> A relational database platform designed for digital game distribution, purchasing, and peer-to-peer game exchanges.

---

## Project Overview

OnlyGames addresses flexibility limitations in current digital gaming platforms by extending traditional store functionalities—such as purchasing, reviews, and library management—with a direct peer-to-peer game exchange feature. Built on a normalized 3NF relational schema, the platform enforces data integrity, tracks dynamic game ownership transfers, and maintains distinct purchase histories.

---

## Key System Features

* **User & Wallet Management:** Manages user profiles, authentication, roles, friends lists, and digital wallet balances.
* **Game Catalog & Metadata:** Supports categories, developers, publishers, tags, system requirements, and dynamic discount management.
* **Order & Transaction System:** Implements cart, wishlist, multi-item order processing, and fixed order history logging.
* **Dynamic Library & Access Control:** Separates order logs from active game access (`UserLibrary`) to support dynamic ownership changes.
* **Peer-to-Peer Game Exchange (Trading):** Allows users to initiate and accept game trades when developer permissions permit (`exchange_allowed`).
* **Interaction Features:** Enables user ratings, written game reviews, and social friend connections.

---

## Database Architecture & Relational Schema

The database is built on **PostgreSQL via Supabase** and strictly adheres to Third Normal Form (**3NF**) to eliminate data redundancy and prevent update anomalies.

### Core Relational Entities & Keys

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

## Architectural & Design Decisions

* **Order vs. Library Separation:** Disconnects `orders` from `user_library` to decouple lifetime purchasing records from current game ownership, enabling dynamic trades.
* **Developer Exchange Control:** Includes an `exchange_allowed` boolean flag on game entities to ensure publishers maintain policy control over game trades.
* **Bridge Tables for Many-to-Many Relationships:** Employs `game_tags` and `order_details` bridge tables to maintain strict normalization and handle complex multi-item transactions.
* **Integrity Constraints:** Uses explicit PostgreSQL foreign keys, `NOT NULL`, `UNIQUE`, and `CHECK` constraints to safeguard transactional consistency.

---

## Example SQL Queries Implemented

* **Catalog Lookup:** Joining `games` and `categories` for detailed storefront rendering.
* **User Purchase History:** Querying historical orders and individual purchased items per profile.
* **Spending Analytics:** Aggregating total user expenditures via `SUM()` functions.
* **Popularity Metrics:** Identifying top-performing titles using `COUNT()` aggregations over orders.
* **Trade Management:** Filtering pending peer-to-peer exchange requests between active users.

---

## Tech Stack & Tools

* **Database Engine:** PostgreSQL
* **Database Platform:** Supabase
* **Language & Syntax:** SQL (DDL, DML, Relational Constraints)
* **Design Methodology:** Entity-Relationship (ER) Modeling, 3NF Database Normalization

---

## Project Structure

```text
onlygames/
├── docs/
│   ├── er-diagram.png
│   └── relational-schema.png
├── sql/
│   ├── schema.sql
│   └── queries.sql
└── README.md
