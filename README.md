# SAUNA — Digital Game Store & Publisher Portal

A full-stack **MySQL 8.4** database and **Flask** web frontend built directly from [`steam_er_diagram.png`](steam_er_diagram.png).

## Database Schema (19 Tables)
Defined in [`sql/01_schema.sql`](sql/01_schema.sql) and seeded in [`sql/02_seed.sql`](sql/02_seed.sql) (visualized in [`sauna_er_diagram.png`](sauna_er_diagram.png)):

- **Localization & Multi-Currency**: `CURRENCIES`, `REGIONS`
- **Catalog & Metadata**: `PUBLISHERS`, `DEVELOPERS`, `GAMES`, `TAGS`, `GAME_TAGS`
- **Users & Social**: `USERS`, `FRIENDSHIPS` (with `trg_friendships_no_self` trigger)
- **Sales & Pricing**: `SALES`, `SALE_ITEMS`
- **Commerce & Wallet**: `ORDERS`, `ORDER_ITEMS` (with `trg_order_items_steam_refund_policy` trigger), `WALLET_TRANSACTIONS`
- **Player Activity**: `USER_LIBRARY` (with Play/Exit session timestamps), `WISHLIST`, `REVIEWS`, `ACHIEVEMENTS`, `USER_ACHIEVEMENTS`

## Key Features
- **ID & Password Authentication**: Log in with Member ID (or username/email) + password (`scrypt` password hashes via `werkzeug.security`), register new accounts, and log out. Only the logged-in member's ID is shown.
- **Region-Based Multi-Currency**: Choose your country/region from the header picker dialog (`India · ₹ INR`, `Japan · ¥ JPY`, `USA · $ USD`, `UK · £ GBP`, `Poland · zł PLN`, `Canada · CA$ CAD`, `Australia · A$ AUD`, `Germany/France · € EUR`); converts storefront/wallet amounts dynamically and snapshots the active currency & exchange rate on orders and wallet transactions.
- **Strict Wallet Enforcement**: All game purchases validate `wallet_balance >= total`, deduct from `USERS.wallet_balance`, and record entries in `WALLET_TRANSACTIONS` inside atomic InnoDB transactions (`SELECT ... FOR UPDATE`).
- **Steam 2-Hour Refund Policy**: Players can refund paid games in their library if played for **less than 2 hours (`< 120` minutes)** and purchased **within 14 days**, enforced in both the Flask transaction and the MySQL `trg_order_items_steam_refund_policy` trigger.
- **Storefront, Library, Wishlist, Friends & Catalog Management**: Browse active seasonal sales, add/remove wishlist items, track playtime, unlock achievements, post/vote on reviews, and manage friends by Member ID.
- **DB & SQL Explorer**: Inspect live rows and schema columns across all 17 MySQL tables and run read-only SQL queries (`SELECT`, `SHOW`, `DESCRIBE`, `EXPLAIN`, `WITH`).

## Running Locally

1. **Install Python dependencies**:
   ```bash
   pip install -r requirements.txt
   ```
2. **Load schema and seed data into MySQL**:
   ```bash
   mysql -u root < sql/01_schema.sql
   mysql -u root < sql/02_seed.sql
   ```
   *(Or use `scripts/start_mysql.sh` if using the local MySQL instance configuration in `mysql.cnf`.)*
3. **Start the Flask server**:
   ```bash
   python3 app.py
   ```
   Open `http://127.0.0.1:5050` in your browser.
   Demo Member IDs: `1`–`6` · Password: `password123`.
