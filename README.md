# EcoNest – Integrated Java Swing Application

One Maven Java Swing project containing the group modules: Admin Login, Dashboard, Client Management, Products, Inventory, Delivery Agents, Orders and Notifications.

## Integrated modules

- **Admin Login** – database-backed `users` authentication with PBKDF2 password hashing (`econest.LoginForm`, `econest.LoginDAO`, `econest.PasswordUtil`).
- **Dashboard** – live counts/lists from `products`, `clients`, `orders`, `deliveries`, `inventory` (`econest.DashboardFrame`).
- **Client Management** – Member 2 module: add/update/delete/search clients (`com.econest.member2.view.ClientManagementFrame`).
- **Products** – product catalogue CRUD (`econest.ProductForm`, `econest.ProductDAO`).
- **Inventory** – stock levels, suppliers, stock-in, low-stock alerts, monthly sales (`com.econest.member2.view.InventoryManagementFrame`, `InventoryDao`).
- **Delivery Agents** – agent CRUD/deactivate, email capture and order assignment (`com.econest.member2.view.DeliveryAgentsFrame`, `DeliveryAgentDao`).
- **Orders** – order creation, confirmation with transactional stock deduction, order history/details, plus a **Delivery & Assignment** tab for agent assignment, scheduling, dispatch and delivery-status tracking (`econest.member5.OrdersPanel`, `DeliveryPanel`, `DeliveryNotificationDAO`).
- **Notifications** – delivery dispatch, agent-assignment queueing, notification history and SMTP sending (`econest.member5.NotificationsPanel`, `DeliveryNotificationDAO`).
- **Reports** – monthly sales summaries from completed orders and a live inventory stock report with low-stock alerts (`econest.member5.ReportsPanel`, `ReportDAO`).

The whole application runs inside **one shared window** (`com.econest.app.AppShell`): the Dashboard-style sidebar (`econest.UnifiedSidebar`) stays fixed on the left and never reloads; only the right-side content changes through a `CardLayout`. Every module uses the same Dashboard colour palette, fonts, cards and tables. Window size is shared (`AppWindows`): up to `1400x760` constrained by the usable screen, minimum `900x600`, with responsive inner layouts.

## Database setup

1. Start MariaDB/MySQL (XAMPP is fine).
2. For a fresh install, import:

   ```text
   database/econest_db.sql
   ```

   It creates `econest_db` and the shared tables: `users`, `clients`, `products`, `suppliers`, `inventory`, `orders`, `order_items`, `delivery_agents`, `deliveries`, `stock_in`, `notifications`.

3. Configure JDBC in `src/main/resources/db.properties`:

   ```properties
   db.url=jdbc:mariadb://localhost:3306/econest_db
   db.username=root
   db.password=
   ```

Do not re-import the SQL file over an existing database unless you intend to reset it.

## Create the first admin account

After importing the database, run the one-time setup class:

```text
econest.CreateAdmin
```

Enter a password with at least 12 characters. It creates username `admin` with role `ADMIN`. No production password is stored in source code. Log in with username `admin` and the password you entered.

## Build / run

Maven project; main class is `com.econest.member2.Main` (opens the login screen). The project targets Java 25 and uses the local `lib/*.jar` files as Maven `system` dependencies, so keep the `lib` folder with the project.

```bash
mvn -q -DskipTests package
```

### VS Code

1. Open `C:\Users\MY\Downloads\EcoNest_Member2` in VS Code.
2. Install/use the **Extension Pack for Java** and let Maven import the project.
3. Either run `com.econest.member2.Main` from the Java Projects view, or use the integrated terminal (PowerShell):

   ```powershell
   mvn -q -DskipTests compile
   $cp = "target\classes;src\main\resources;lib\flatlaf-3.8.jar;lib\jakarta.mail-api-2.2.0-M1.jar;lib\angus-mail-2.0.5.jar;lib\angus-activation-2.0.3.jar;lib\jakarta.activation-api-2.2.0-M1.jar;$env:USERPROFILE\.m2\repository\org\mariadb\jdbc\mariadb-java-client\3.5.3\mariadb-java-client-3.5.3.jar"
   java -cp $cp com.econest.member2.Main
   ```

### NetBeans

1. Open the project by selecting the `pom.xml` file and choosing **Open Project** as a Maven project.
2. Set the project Main Class to `com.econest.member2.Main`.
3. Run the project with **Run → Run Project** (`F6`).

If NetBeans does not resolve the local jars, verify the `lib` folder contains `flatlaf-3.8.jar`, `jakarta.mail-api-2.2.0-M1.jar`, `angus-mail-2.0.5.jar`, `angus-activation-2.0.3.jar` and `jakarta.activation-api-2.2.0-M1.jar`.

## Notifications email setup

Order/notification records are stored in the database. To actually send queued emails, set these environment variables before running the app:

```text
ECONEST_SMTP_HOST
ECONEST_SMTP_PORT
ECONEST_SMTP_USER
ECONEST_SMTP_PASSWORD
ECONEST_SMTP_FROM
```

Without SMTP settings, notifications remain `Pending` and the send action reports that email is not configured.

## Logo

Sidebar logo resource: `src/main/resources/assets/logo.png`. Replace that file to change the logo; no Java change is needed. If it is missing, the UI falls back to text instead of crashing.

## Verified database tables and relationships

Verified against `econest_db`:

- `users` – admin authentication.
- `clients` – `client_id` PK; used by `orders.client_id`.
- `products` – `product_id` PK; used by `inventory`, `order_items`, `stock_in`.
- `suppliers` – `supplier_id` PK; used by `stock_in.supplier_id`.
- `inventory` – `product_id` PK/FK to `products`; stock levels for orders/dashboard.
- `orders` – `order_id` PK; `client_id` FK to `clients`.
- `order_items` – `order_id` FK to `orders` cascade, `product_id` FK to `products`.
- `delivery_agents` – `agent_id` PK; used by `deliveries.agent_id` and `notifications.agent_id`.
- `deliveries` – `order_id` unique FK to `orders`, `agent_id` FK to `delivery_agents`.
- `stock_in` – `product_id` FK to `products`, `supplier_id` FK to `suppliers`.
- `notifications` – `order_id` FK to `orders`, `agent_id` FK to `delivery_agents` with `ON DELETE SET NULL`.

## Fixes made in this pass

- Rebuilt navigation as a **single shared window** (`AppShell` + `CardLayout`): the sidebar and logo never reload or move; only the content area switches. No more window close/reopen "shake" when navigating.
- Removed the default Swing blue hover/focus styling from the sidebar; hover, pressed, selected and normal states all use the Dashboard green palette.
- Converted Client Management, Inventory and Delivery Agents from separate `JFrame`s into content panels hosted by the shared window; removed their duplicate sidebars.
- Products: cleared design-time sample text, added real search across ID/name/category/price, tightened validation, reported load errors, and made add/delete transactional with `inventory`.
- Products: adding a product creates its `inventory` row; deleting a product removes that inventory row first and still fails safely if order/stock history references it.
- Removed the mid-session FlatLaf switch inside Products so every screen keeps the same look and feel as the Dashboard.
- Fixed the login flow so the Login window closes before the shared application window opens, and Logout returns to Login.
- Delivery Agents: added required email capture/storage so agent-assignment notifications have a recipient; removed the stray glyph before the page title.
- Inventory: added the missing CRUD pieces — text search across product/SKU, low-stock filter, refresh, update of quantity/reorder levels, stock-in recording and safe inventory-record deletion (the product row is kept).
- Orders/Notifications: kept the real DAO workflows; confirmed transactional order confirmation, stock deduction, delivery assignment, dispatch and notification queueing.
- Made all content panels responsive inside the shared window (scroll panes, proportional layouts, minimum window size).
- Products page: removed the small accent bar before the title, removed the icon box before "Product Details", and fixed the Remove button so its text is never clipped.
- Standardised every button across all pages on the Products button style: rounded corners, white text on coloured buttons, dark text on light buttons, consistent padding (member2 `Ui.button` and member5 `EcoTheme.button`, plus the Dashboard profile dialog).
- Orders: added a real **Update Order** workflow — load a Pending order's client/items into the form, edit, and save via a transactional `UPDATE` that rebuilds `order_items` and recomputes `total_amount`. Non-Pending orders are rejected so confirmation/stock/delivery workflows cannot be bypassed.
- Notifications: the history table now shows Sent At and Read columns; added **Mark as Read** persisted via a new additive `notifications.is_read` column; kept dispatch, agent-assignment queueing, pending-email sending and refresh.

## Testing performed

- `mvn -q -DskipTests compile` and `mvn -q -DskipTests package` passed.
- `ShellSmoke`: instantiated the shared `AppShell` and switched through Dashboard → Products → Clients → Inventory → Delivery Agents → Orders → Notifications → Dashboard at two window sizes with no exceptions.
- `DbSmoke`: connected to `econest_db`; current counts `users=1`, `clients=1`, `products=3`, `inventory=3`, `suppliers=1`, `delivery_agents=1`, `orders=0`, `order_items=0`, `deliveries=0`, `stock_in=0`, `notifications=0` (the supplier/agent rows are real user data, not test leftovers).
- `AdminAuthSmoke`: valid admin password accepted (`authenticate=true`) and wrong password rejected (`wrongPasswordRejected=true`).
- `ModuleDaoSmoke` passed:
  - product add/update/delete including inventory row creation/removal
  - client insert/search/update/delete
  - inventory/product/supplier read queries
  - supplier + stock-in transaction
  - agent save with persisted email
  - client -> order -> confirm -> stock deduction -> agent assign -> dispatch -> notification queue flow
  - all test rows were cleaned up and final table counts returned to the values above.
- `OrderUpdateSmoke`: created a Pending order, loaded it via `editableOrder`, updated client/items, verified `total_amount` recompute, confirmed stock deduction still works, verified update rejects a Confirmed order, and cleaned up all test rows.
- Notifications `is_read` column applied to the live database (`ALTER TABLE notifications ADD COLUMN is_read tinyint(1) NOT NULL DEFAULT 0`) and added to `database/econest_db.sql` for fresh installs.

## Remaining limitations

- Actual email sending requires the `ECONEST_SMTP_*` environment variables; otherwise notifications stay `Pending`.
- Dispatch requires the client to have an email address; clients with only a phone number must be updated in Client Management first.
- Deleting a product is blocked by database foreign keys if it already has order or stock-in history; use inventory/order history carefully rather than resetting data.
- Reports and Settings were not supplied as functional modules, so they are not shown as navigation items.
- NetBeans `.form` GUI-builder files were not copied; Maven/runtime execution does not need them.
