---
title: "mb-4-ecom-commerce-schema"
updated: "2026-11-24"
---
# Mybizz Shop — Commerce Schema

Every table and column of the shop: the trade part of `mb-4-ecom`. Name, type and allowed values.

**Part:** trade. Every table in this document is a trade table ([[[default:adr-global/adr-common-and-trade-modules|Common and Trade Modules]]](C:/dev/project-library-global/adr-global/adr-common-and-trade-modules.md)). The common tables an `mb-4-ecom` client instance also holds come from the project template, as the common part of every Mybizz application.

**Features:** feature numbers (F-S01 to F-S16) are those of `mb-4-ecom-commerce-feature-list.md`. The step-by-step states of carts, orders, payments, shipments, returns, discounts and stock are in `mb-4-ecom-commerce-lifecycles.md`.

**Total tables:** 38
**Platform:** Anvil Data Tables. Every table has `client: none` and `server: full`: the browser never reads a table directly; server functions return rows and results.

---

## Parties

Every document of the shop names the parties by these terms only.

| Party | Who |
|---|---|
| Mybizz | The business that provides the shop application to clients |
| Client | The business that subscribes to Mybizz and runs the shop |
| Team member | A person who works for the client and signs in to the client's application; the Owner and the Managers are team members |
| Customer | A person or a business that buys from the client's shop. A private customer buys as a private person (a consumer, in law); a business customer buys for a business |
| Provider | An outside service the client connects with the client's own account: the payment providers Stripe and PayPal, the tax provider Stripe Tax, and the courier providers Sendcloud and Shippo |

---

## Rules

- **The common rules apply.** One instance per client, UTC datetimes, money in whole cents of the currency named on the record, credentials only in the Vault, and the naming and standard columns of [[[default:specifications-global/spec-nomenclature|Nomenclature]]](C:/dev/project-library-global/specifications-global/spec-nomenclature.md).
- **One direction.** A shop table may link to a common table. No common table links to a shop table. Where a shop record needs a common record (a payment, an invoice, a contact), the shop table holds the link.
- **Selling currencies.** The client's accounts and reports are in `business_profile.system_currency`. Each sales zone sells in one currency: the system currency, or another currency the client chooses, at an exchange rate the client enters. Customers in a zone see prices, pay, receive invoices and are refunded in the zone's currency. Every cart and order carries the order's currency, and every order stores the exchange rate at placing, so every amount converts to the system currency exactly for the client's accounts and reports.
- **Prices in other currencies.** A variant's price in a zone's currency comes from an active price list in that currency, or else from the base price converted at the zone's exchange rate and rounded by the zone's rounding rule. Base prices, compare-at prices, fixed-amount discounts, minimum order amounts, credit limits and performance figures are in the system currency.
- **Snapshots.** An order line stores the product title, variant title, SKU, unit price and tax rate as these were when the order was placed. Later changes to the product never change a placed order.
- **Never deleted.** Rows of `orders`, `order_items`, `shipments`, `returns`, `stock_movements` and `discount_redemptions` are never deleted; the row's status changes. A product, variant or discount that has been sold or used is archived, never deleted.
- **Stock changes in transactions.** Every change to `stock_levels` runs inside one Anvil Data Tables transaction together with the `stock_movements` or `stock_reservations` row that explains the change, so stock can never be sold twice.
- **Customers are contacts.** A shop customer is one row of the common `contacts` table ([[[default:adr-global/adr-single-contacts-table|Single Contacts Table]]](C:/dev/project-library-global/adr-global/adr-single-contacts-table.md)), created or matched by email at checkout.

### Column types

As in the common schema: `string`, `number`, `bool`, `date`, `datetime`, `media`, `simpleObject`, `link → table` (one row) and `links → table` (several rows).

### Common tables the shop links to

| Common table | Used for |
|---|---|
| `contacts` | The customer of a cart, order, return or review |
| `users` | The team member who acted (created an order, packed a shipment, received a return) |
| `payments` | Every payment and refund of an order, held by the common payments feature |
| `invoice` | The paid invoice (receipt) issued for every order, so accounts and tax reports include shop sales |
| `files` | Product images and digital product files, held in the client's R2 bucket |
| `segments` | A CRM segment that a price list or a discount is limited to |
| `reviews` | A customer's review, attached to a product through `product_reviews` |
| `business_profile` | The system currency, the tax label and registration number, and the home country |
| `vault` | The secret keys of the courier providers and of Stripe Tax |

---

## Table Groups

| Group | Tables | Count |
|---|---|---|
| Shop settings | `shop_settings`, `shop_counters`, `courier_connections`, `product_feeds` | 4 |
| Catalogue | `products`, `product_options`, `product_option_values`, `product_variants`, `product_images`, `product_categories`, `product_collections` | 7 |
| Pricing | `price_lists`, `price_list_prices` | 2 |
| Stock | `stock_locations`, `stock_levels`, `stock_reservations`, `stock_movements` | 4 |
| Zones, tax and delivery | `sales_zones`, `consumer_rule_sets`, `zone_tax_rates`, `shipping_options` | 4 |
| Cart | `carts`, `cart_items` | 2 |
| Orders | `orders`, `order_items`, `order_disputes` | 3 |
| Shipments | `shipments`, `shipment_items` | 2 |
| Returns | `returns`, `return_items` | 2 |
| Discounts | `discounts`, `discount_redemptions` | 2 |
| Credit accounts | `credit_applications`, `customer_accounts` | 2 |
| Gift cards | `gift_cards`, `gift_card_transactions` | 2 |
| Reviews and performance | `product_reviews`, `product_performance` | 2 |
| **Total** | | **38** |

---

## 1. Shop Settings

### 1.1 `shop_settings`
One row: how the client's shop works (F-S01, F-S03, F-S04, F-S05, F-S08, F-S09, F-S10, F-S13, F-S14). Set in onboarding and changed in Settings.

| Column | Type | Notes |
|---|---|---|
| `shop_open` | bool | When False, the storefront shows the catalogue but checkout is closed |
| `sells_to` | string | `consumers` / `businesses` / `both` |
| `governing_law_country` | string | ISO 3166-1 alpha-2; the law of the Terms; set to `business_profile.country` at onboarding |
| `tax_engine` | string | `manual` (the shop's own zone rates) / `stripe_tax` (rates worked out by Stripe Tax) |
| `tax_responsibility_accepted_at` | datetime | When a team member last accepted the tax responsibility dialog for the `manual` tax method |
| `tax_responsibility_accepted_by` | link → users | The team member who accepted |
| `credit_accounts_enabled` | bool | True: customers may apply for a credit account and pay on invoice |
| `prices_include_tax` | bool | True: catalogue prices include tax (usual for consumer shops in the UK and EU). False: tax is added at checkout |
| `guest_checkout` | bool | True: customers may buy without creating an account |
| `default_stock_location_id` | link → stock_locations | Where stock is taken from when a variant has stock in several locations and no other rule applies |
| `low_stock_threshold` | number | Default threshold for the low-stock alert; a variant may set a threshold of the variant's own |
| `cart_hold_minutes` | number | How long stock stays reserved for a cart at payment; default `30` |
| `abandoned_cart_hours` | number | Hours after the last change to a cart with an email before the abandoned-cart email is sent; default `4`; `0` turns the abandoned-cart email off |
| `return_window_days` | number | Days after delivery within which a customer may request a return; default `30`; never below the `min_cancellation_days` of any active zone's consumer rule set |
| `return_shipping_paid_by` | string | `customer` / `business` |
| `digital_download_limit` | number | Downloads allowed per purchased digital item; default `5` |
| `digital_link_days` | number | Days a download link stays valid; default `30` |
| `order_confirmation_note` | string | Text added to the order confirmation email |
| `created_at` | datetime | |
| `updated_at` | datetime | |

### 1.2 `shop_counters`
Sequence numbers for order and return numbers. One row per counter and year.

| Column | Type | Notes |
|---|---|---|
| `name` | string | `order` / `return` |
| `year` | number | |
| `last_value` | number | Raised by one, inside a transaction, for each new number |
| `created_at` | datetime | |
| `updated_at` | datetime | |

### 1.3 `product_feeds`
A product feed: the list of the shop's products that a sales channel reads on a schedule (F-S15). One row per channel and zone.

| Column | Type | Notes |
|---|---|---|
| `channel` | string | `google` (Google Merchant Center: Google Shopping and free listings) / `meta` (Facebook and Instagram shops) / `tiktok` (TikTok catalogue) |
| `zone_id` | link → sales_zones | The zone whose currency, prices, tax and delivery the feed shows |
| `feed_token` | string | Random, unguessable part of the feed's web address |
| `enabled` | bool | |
| `file_id` | link → files | The latest feed file, rebuilt every hour |
| `built_at` | datetime | |
| `last_read_at` | datetime | When the channel last fetched the feed |
| `item_count` | number | Products in the latest feed |
| `created_at` | datetime | |
| `updated_at` | datetime | |

### 1.4 `courier_connections`
The client's courier provider accounts (F-S08). One row per courier provider. Secret keys are in the Vault.

| Column | Type | Notes |
|---|---|---|
| `platform` | string | `sendcloud` / `shippo` |
| `test_mode` | bool | |
| `connected` | bool | |
| `default_parcel_weight_grams` | number | Used when a variant has no weight |
| `label_format` | string | `a4_pdf` / `a6_pdf` / `4x6_pdf` |
| `connected_at` | datetime | |
| `created_at` | datetime | |
| `updated_at` | datetime | |

---

## 2. Catalogue

### 2.1 `products`
What the shop sells (F-S01). A product has one or more variants; the variant is what is priced, stocked and bought.

| Column | Type | Notes |
|---|---|---|
| `title` | string | |
| `handle` | string | Unique; the web address of the product page (`/shop/{handle}`) |
| `subtitle` | string | |
| `description` | string | Rich text |
| `kind` | string | `physical` / `digital` / `gift_card` |
| `status` | string | `draft` / `published` / `archived` |
| `brand` | string | Shown in product feeds |
| `condition` | string | `new` / `used` / `refurbished`; shown in product feeds |
| `google_product_category` | string | Google's product category, for product feeds |
| `in_product_feeds` | bool | True: the product is listed in every enabled product feed |
| `category_ids` | links → product_categories | |
| `collection_id` | link → product_collections | Optional |
| `tags` | simpleObject | List of tags, used for search and discount rules |
| `thumbnail_id` | link → product_images | The image shown in listings |
| `tax_class` | string | `standard` / `reduced` / `zero` / `exempt`; default `standard` |
| `discountable` | bool | False: no discount applies to this product |
| `seo_title` | string | |
| `seo_description` | string | |
| `featured` | bool | Shown in the storefront's featured section |
| `sort_order` | number | Order in listings where no other sort is chosen |
| `published_at` | datetime | |
| `created_at` | datetime | |
| `updated_at` | datetime | |
| `metadata` | simpleObject | Client-level attributes (common aspect 4) |

### 2.2 `product_options`
The choices a product offers, such as Size or Colour (F-S01).

| Column | Type | Notes |
|---|---|---|
| `product_id` | link → products | |
| `title` | string | e.g. `Size` |
| `sort_order` | number | |
| `created_at` | datetime | |
| `updated_at` | datetime | |

### 2.3 `product_option_values`
The values of one option, such as Small, Medium and Large (F-S01).

| Column | Type | Notes |
|---|---|---|
| `option_id` | link → product_options | |
| `value` | string | e.g. `Medium` |
| `sort_order` | number | |
| `created_at` | datetime | |
| `updated_at` | datetime | |

### 2.4 `product_variants`
One buyable version of a product: one value for each of the product's options (F-S01, F-S02, F-S03).

| Column | Type | Notes |
|---|---|---|
| `product_id` | link → products | |
| `title` | string | e.g. `Medium / Blue`; for a product with no options, the product title |
| `option_value_ids` | links → product_option_values | One value per option of the product |
| `sku` | string | Unique when set |
| `barcode` | string | EAN, UPC or other; unique when set |
| `price` | number | The base price, in cents of the system currency; tax included or not as `shop_settings.prices_include_tax` sets |
| `compare_at_price` | number | Optional; shown struck through when higher than the price |
| `manage_stock` | bool | True: stock is counted. False: always available (made to order, services, digital) |
| `allow_backorder` | bool | True: may be sold when no stock is available |
| `low_stock_threshold` | number | Optional; overrides `shop_settings.low_stock_threshold` |
| `requires_shipping` | bool | False for a digital variant |
| `weight_grams` | number | Used by weight-based shipping options |
| `length_mm` | number | |
| `width_mm` | number | |
| `height_mm` | number | |
| `download_file_id` | link → files | For a digital variant: the file delivered |
| `image_ids` | links → product_images | Images shown when this variant is chosen |
| `status` | string | `active` / `archived` |
| `sort_order` | number | |
| `created_at` | datetime | |
| `updated_at` | datetime | |
| `metadata` | simpleObject | Client-level attributes (common aspect 4) |

### 2.5 `product_images`
Images of a product (F-S01).

| Column | Type | Notes |
|---|---|---|
| `product_id` | link → products | |
| `file_id` | link → files | The image, held in the client's R2 bucket |
| `alt_text` | string | |
| `sort_order` | number | |
| `created_at` | datetime | |
| `updated_at` | datetime | |

### 2.6 `product_categories`
The shop's category tree, such as Clothing > Women > Dresses (F-S01).

| Column | Type | Notes |
|---|---|---|
| `name` | string | |
| `handle` | string | Unique; the web address of the category page (`/shop/c/{handle}`) |
| `description` | string | |
| `parent_id` | link → product_categories | Empty for a top-level category |
| `image_id` | link → files | |
| `is_active` | bool | Inactive categories are hidden from the storefront |
| `sort_order` | number | |
| `created_at` | datetime | |
| `updated_at` | datetime | |
| `metadata` | simpleObject | Client-level attributes (common aspect 4) |

### 2.7 `product_collections`
A hand-picked group of products shown together, such as Summer Edit or Gift Ideas (F-S01).

| Column | Type | Notes |
|---|---|---|
| `title` | string | |
| `handle` | string | Unique; the web address of the collection page (`/shop/collection/{handle}`) |
| `description` | string | |
| `image_id` | link → files | |
| `is_active` | bool | |
| `sort_order` | number | |
| `created_at` | datetime | |
| `updated_at` | datetime | |
| `metadata` | simpleObject | Client-level attributes (common aspect 4) |

---

## 3. Pricing

### 3.1 `price_lists`
A set of prices that replaces variant base prices: a timed sale, or prices for one customer segment such as wholesale business customers (F-S02).

| Column | Type | Notes |
|---|---|---|
| `title` | string | |
| `description` | string | |
| `type` | string | `sale` (prices shown as reductions, with the base price struck through) / `override` (prices shown as the normal price) |
| `status` | string | `draft` / `active` / `ended` |
| `starts_at` | datetime | Empty: from activation |
| `ends_at` | datetime | Empty: until ended |
| `currency` | string | ISO 4217; the price list applies only in zones that sell in this currency |
| `segment_id` | link → segments | Optional; empty: every customer |
| `created_at` | datetime | |
| `updated_at` | datetime | |
| `metadata` | simpleObject | Client-level attributes (common aspect 4) |

### 3.2 `price_list_prices`
One price for one variant in one price list (F-S02).

| Column | Type | Notes |
|---|---|---|
| `price_list_id` | link → price_lists | |
| `variant_id` | link → product_variants | |
| `price` | number | In cents of the price list's currency |
| `min_quantity` | number | Optional; the price applies from this quantity in the cart (bulk pricing) |
| `created_at` | datetime | |
| `updated_at` | datetime | |

---

## 4. Stock

### 4.1 `stock_locations`
Places stock is kept: the shop, a store room, a warehouse (F-S03).

| Column | Type | Notes |
|---|---|---|
| `name` | string | |
| `address` | simpleObject | Keys `address_line_1`, `address_line_2`, `city`, `region`, `postal_code`, `country` |
| `is_pickup_point` | bool | Customers may collect orders here |
| `is_active` | bool | |
| `created_at` | datetime | |
| `updated_at` | datetime | |

### 4.2 `stock_levels`
Stock of one variant at one location (F-S03). Available stock is `on_hand` − `reserved`.

| Column | Type | Notes |
|---|---|---|
| `variant_id` | link → product_variants | |
| `location_id` | link → stock_locations | One row per variant and location |
| `on_hand` | number | Units physically held |
| `reserved` | number | Units held for carts at payment and for placed orders not yet shipped |
| `incoming` | number | Units on order from suppliers, entered by the client |
| `low_stock_alerted_at` | datetime | When the last low-stock alert was sent; cleared when stock rises above the threshold |
| `created_at` | datetime | |
| `updated_at` | datetime | |

### 4.3 `stock_reservations`
Units held for one cart line or order line (F-S03, F-S05).

| Column | Type | Notes |
|---|---|---|
| `variant_id` | link → product_variants | |
| `location_id` | link → stock_locations | |
| `quantity` | number | |
| `cart_item_id` | link → cart_items | While the cart is at payment |
| `order_item_id` | link → order_items | Once the order is placed |
| `status` | string | `held` / `committed` / `released` |
| `expires_at` | datetime | For a `held` reservation: when the reservation is released if payment has not succeeded |
| `created_at` | datetime | |
| `updated_at` | datetime | |

### 4.4 `stock_movements`
Every change to `on_hand`, with the reason for the change (F-S03). The stock history of a variant.

| Column | Type | Notes |
|---|---|---|
| `variant_id` | link → product_variants | |
| `location_id` | link → stock_locations | |
| `change` | number | Positive adds stock, negative removes stock |
| `reason` | string | `received` / `shipped` / `returned` / `written_off` (damaged, lost or expired stock) / `count_correction` / `transfer_in` / `transfer_out` |
| `shipment_id` | link → shipments | For `shipped` |
| `return_id` | link → returns | For `returned` |
| `note` | string | |
| `user_id` | link → users | Who made the change; empty for automatic changes |
| `created_at` | datetime | |

---

## 5. Zones, Tax and Delivery

### 5.1 `sales_zones`
A group of countries the shop sells to, with the zone's own tax rates and delivery options (F-S08, F-S10, F-S12). Every country the shop sells to belongs to exactly one zone.

| Column | Type | Notes |
|---|---|---|
| `name` | string | e.g. `United Kingdom`, `European Union`, `Rest of World` |
| `country_codes` | simpleObject | List of ISO 3166-1 alpha-2 codes |
| `is_home` | bool | True for exactly one zone: the zone of `business_profile.country` |
| `consumer_rule_set_id` | link → consumer_rule_sets | The consumer protection rules applied to private customers in this zone |
| `currency` | string | ISO 4217 code of the currency the zone sells in; default `business_profile.system_currency` |
| `exchange_rate` | number | Units of the zone's currency per 1 unit of the system currency, entered by the client; `1` when the zone sells in the system currency |
| `price_rounding` | string | How converted prices are rounded: `none` / `whole` (to the nearest whole unit) / `ninety_nine` (to .99) |
| `is_active` | bool | Inactive: checkout is closed to addresses in the zone |
| `created_at` | datetime | |
| `updated_at` | datetime | |

### 5.2 `consumer_rule_sets`
The minimum consumer protections of a group of countries, applied to private customers in every zone that uses the set (F-S14). Mybizz supplies the sets; the client may make any value more generous, never less.

| Column | Type | Notes |
|---|---|---|
| `name` | string | e.g. `United Kingdom`, `European Union`, `No statutory cancellation right` |
| `supplied_by_mybizz` | bool | True for the sets Mybizz supplies and keeps current with each release |
| `min_cancellation_days` | number | Days after delivery within which a consumer may cancel without reason, e.g. `14` |
| `refund_deadline_days` | number | Days within which a refund is made after the goods come back or proof of posting is given, e.g. `14` |
| `refund_basic_delivery` | bool | True: a cancellation refund includes the original standard delivery charge |
| `digital_waiver_required` | bool | True: a digital item is released before the cancellation period ends only after the customer's waiver |
| `created_at` | datetime | |
| `updated_at` | datetime | |

### 5.3 `zone_tax_rates`
The tax rate for one tax class in one zone, used when `shop_settings.tax_engine` is `manual` (F-S10).

| Column | Type | Notes |
|---|---|---|
| `zone_id` | link → sales_zones | |
| `tax_class` | string | `standard` / `reduced` / `zero` / `exempt` |
| `rate` | number | Percentage, e.g. `20` |
| `applies_to_shipping` | bool | True for the class whose rate is charged on delivery (usually `standard`) |
| `created_at` | datetime | |
| `updated_at` | datetime | |

### 5.4 `shipping_options`
A delivery choice offered at checkout in one zone (F-S08).

| Column | Type | Notes |
|---|---|---|
| `zone_id` | link → sales_zones | |
| `name` | string | Shown at checkout, e.g. `Standard delivery (2–4 days)` |
| `kind` | string | `flat` / `weight_band` / `live_rate` (the courier provider's price for the parcel) / `collection` |
| `price` | number | In cents of the zone's currency; `0` for free; for `live_rate`, an amount added to the courier's price |
| `free_over_amount` | number | Optional, in the zone's currency; the option is free when the cart's items total reaches this amount |
| `min_weight_grams` | number | For `weight_band` |
| `max_weight_grams` | number | For `weight_band` |
| `pickup_location_id` | link → stock_locations | For `collection` |
| `courier_connection_id` | link → courier_connections | For a courier-provider option: the courier provider that buys the label |
| `courier_service_code` | string | The courier provider's code for the courier service, e.g. Royal Mail Tracked 48 |
| `carrier_name` | string | e.g. `Royal Mail`; shown with tracking |
| `tracking_url_template` | string | e.g. `https://track.example.com/{tracking_number}` |
| `is_active` | bool | |
| `sort_order` | number | |
| `created_at` | datetime | |
| `updated_at` | datetime | |

---

## 6. Cart

### 6.1 `carts`
A shopping cart (F-S04, F-S05). A cart becomes an order when the cart's payment succeeds.

| Column | Type | Notes |
|---|---|---|
| `cart_token` | string | Random, unguessable id held in the customer's browser |
| `contact_id` | link → contacts | When the customer is signed in or has given an email |
| `email` | string | |
| `status` | string | `active` / `at_payment` / `completed` / `abandoned` / `expired` |
| `shipping_address` | simpleObject | Keys as `stock_locations.address`, plus `first_name`, `last_name`, `phone` |
| `billing_address` | simpleObject | As `shipping_address`; empty: same as shipping |
| `zone_id` | link → sales_zones | Set from the shipping country |
| `shipping_option_id` | link → shipping_options | |
| `currency` | string | The zone's currency, set from the shipping country |
| `discount_codes` | simpleObject | List of codes entered |
| `gift_card_ids` | links → gift_cards | Gift cards applied to the cart |
| `items_subtotal` | number | Recomputed on every change; all totals in cents of the cart's currency |
| `discount_total` | number | |
| `shipping_total` | number | |
| `tax_total` | number | |
| `total` | number | |
| `gateway_session_ref` | string | The payment provider's checkout or payment intent id while `at_payment` |
| `abandoned_email_sent_at` | datetime | |
| `order_id` | link → orders | Set when the cart is completed |
| `last_activity_at` | datetime | |
| `created_at` | datetime | |
| `updated_at` | datetime | |
| `metadata` | simpleObject | Client-level attributes (common aspect 4); extra checkout fields |

### 6.2 `cart_items`
One line of a cart (F-S04).

| Column | Type | Notes |
|---|---|---|
| `cart_id` | link → carts | |
| `variant_id` | link → product_variants | |
| `quantity` | number | |
| `unit_price` | number | The price that applies now, in the cart's currency, recomputed on every cart change |
| `price_list_id` | link → price_lists | When a price list set the price |
| `created_at` | datetime | |
| `updated_at` | datetime | |

---

## 7. Orders

### 7.1 `orders`
A customer's order (F-S07). Placed when the order's payment succeeds, or created by a team member.

| Column | Type | Notes |
|---|---|---|
| `order_number` | string | `ORD-{year}-{sequence}` |
| `contact_id` | link → contacts | |
| `email` | string | |
| `channel` | string | `online` / `team` |
| `buyer_type` | string | `consumer` / `business` |
| `status` | string | `draft` / `placed` / `completed` / `cancelled` |
| `payment_status` | string | `unpaid` / `paid` / `partially_refunded` / `refunded` |
| `fulfilment_status` | string | `not_shipped` / `partially_shipped` / `shipped` / `delivered` |
| `shipping_address` | simpleObject | Snapshot at placing |
| `billing_address` | simpleObject | Snapshot at placing |
| `zone_id` | link → sales_zones | |
| `currency` | string | The order's currency: the zone's currency at placing |
| `exchange_rate` | number | Snapshot of the zone's exchange rate at placing |
| `shipping_option_id` | link → shipping_options | |
| `shipping_name` | string | Snapshot of the option's name |
| `prices_include_tax` | bool | Snapshot of `shop_settings.prices_include_tax` |
| `items_subtotal` | number | All amounts in cents of the order's currency, fixed at placing |
| `discount_total` | number | |
| `shipping_total` | number | |
| `tax_total` | number | |
| `total` | number | |
| `gift_card_total` | number | Amount paid by gift cards |
| `refunded_total` | number | Sum of succeeded refunds |
| `total_system` | number | `total` converted to the system currency at `exchange_rate`, for the client's accounts and reports |
| `pay_on_invoice` | bool | True: paid against the invoice on the customer's credit account |
| `customer_account_id` | link → customer_accounts | For `pay_on_invoice` |
| `tax_engine` | string | Snapshot of `shop_settings.tax_engine` |
| `tax_reference` | string | The Stripe Tax transaction id, when `stripe_tax` |
| `consumer_rule_set_id` | link → consumer_rule_sets | Snapshot at placing; empty for a business customer |
| `digital_waiver_accepted_at` | datetime | When the customer waived the cancellation right for digital items |
| `discount_ids` | links → discounts | |
| `payment_ids` | links → payments | Every payment and refund of the order |
| `invoice_id` | link → invoice | The paid invoice issued for the order |
| `customer_note` | string | Note from the customer at checkout |
| `internal_note` | string | |
| `created_by` | link → users | For a `team` order |
| `placed_at` | datetime | |
| `completed_at` | datetime | |
| `cancelled_at` | datetime | |
| `cancel_reason` | string | |
| `created_at` | datetime | |
| `updated_at` | datetime | |
| `metadata` | simpleObject | Client-level attributes (common aspect 4); extra checkout fields |

### 7.2 `order_items`
One line of an order, with the line's snapshot (F-S07).

| Column | Type | Notes |
|---|---|---|
| `order_id` | link → orders | |
| `variant_id` | link → product_variants | |
| `product_title` | string | Snapshot |
| `variant_title` | string | Snapshot |
| `sku` | string | Snapshot |
| `kind` | string | `physical` / `digital`; snapshot |
| `quantity` | number | |
| `unit_price` | number | Snapshot, in cents of the order's currency |
| `discount_amount` | number | This line's share of discounts |
| `tax_class` | string | Snapshot |
| `tax_rate` | number | Snapshot |
| `tax_amount` | number | |
| `line_total` | number | |
| `shipped_quantity` | number | |
| `returned_quantity` | number | |
| `download_count` | number | For a digital line |
| `download_expires_at` | datetime | For a digital line |
| `created_at` | datetime | |
| `updated_at` | datetime | |

### 7.3 `order_disputes`
A dispute or chargeback opened against an order's payment (F-S14).

| Column | Type | Notes |
|---|---|---|
| `order_id` | link → orders | |
| `payment_id` | link → payments | The disputed payment |
| `gateway` | string | `stripe` / `paypal` |
| `gateway_dispute_ref` | string | The payment provider's dispute id |
| `reason` | string | `not_received` / `not_as_described` / `unauthorised` / `duplicate` / `other` |
| `amount` | number | In cents of the order's currency |
| `status` | string | `open` / `evidence_submitted` / `won` / `lost` / `accepted` |
| `evidence_due_at` | datetime | The payment provider's deadline for evidence |
| `evidence_note` | string | |
| `handled_by` | link → users | |
| `opened_at` | datetime | |
| `closed_at` | datetime | |
| `created_at` | datetime | |
| `updated_at` | datetime | |

---

## 8. Shipments

### 8.1 `shipments`
One parcel sent, or one collection handed over (F-S08). An order may have several.

| Column | Type | Notes |
|---|---|---|
| `order_id` | link → orders | |
| `location_id` | link → stock_locations | Where the parcel left from |
| `status` | string | `preparing` / `shipped` / `delivered` / `cancelled` |
| `courier_connection_id` | link → courier_connections | When the label was bought through a courier provider |
| `courier_shipment_ref` | string | The courier provider's shipment id |
| `label_file_id` | link → files | The shipping label |
| `label_cost` | number | In cents of the system currency; what the courier provider charged the client |
| `carrier_name` | string | |
| `tracking_number` | string | |
| `tracking_url` | string | Built from the option's template, or typed |
| `packed_by` | link → users | |
| `shipped_at` | datetime | |
| `delivered_at` | datetime | |
| `cancelled_at` | datetime | |
| `created_at` | datetime | |
| `updated_at` | datetime | |
| `metadata` | simpleObject | Client-level attributes (common aspect 4) |

### 8.2 `shipment_items`
The order lines and quantities in one shipment (F-S08).

| Column | Type | Notes |
|---|---|---|
| `shipment_id` | link → shipments | |
| `order_item_id` | link → order_items | |
| `quantity` | number | |
| `created_at` | datetime | |

---

## 9. Returns

### 9.1 `returns`
A return of items from one order (F-S09).

| Column | Type | Notes |
|---|---|---|
| `return_number` | string | `RET-{year}-{sequence}` |
| `order_id` | link → orders | |
| `contact_id` | link → contacts | |
| `status` | string | `requested` / `approved` / `received` / `closed` / `rejected` / `cancelled` |
| `requested_by` | string | `customer` / `team` |
| `resolution` | string | `refund` / `replacement` |
| `return_location_id` | link → stock_locations | Where the items are sent back to |
| `return_tracking_number` | string | |
| `refund_amount` | number | In cents of the order's currency; set at approval, confirmed at receipt |
| `refund_shipping` | bool | True: the original delivery charge is refunded too |
| `is_cancellation` | bool | True: a consumer's cancellation within the statutory period of the order's consumer rule set |
| `return_label_file_id` | link → files | A prepaid return label bought through the courier provider |
| `proof_of_posting_at` | datetime | When the customer gave proof of sending the goods back |
| `refund_due_by` | datetime | For a cancellation: the earlier of received date and proof of posting, plus `refund_deadline_days` |
| `refund_payment_id` | link → payments | The refund, once made |
| `replacement_order_id` | link → orders | For `replacement` |
| `customer_note` | string | |
| `internal_note` | string | |
| `handled_by` | link → users | |
| `requested_at` | datetime | |
| `approved_at` | datetime | |
| `received_at` | datetime | |
| `closed_at` | datetime | |
| `created_at` | datetime | |
| `updated_at` | datetime | |
| `metadata` | simpleObject | Client-level attributes (common aspect 4) |

### 9.2 `return_items`
The order lines and quantities in one return (F-S09).

| Column | Type | Notes |
|---|---|---|
| `return_id` | link → returns | |
| `order_item_id` | link → order_items | |
| `quantity` | number | Requested |
| `received_quantity` | number | |
| `reason` | string | `wrong_size` / `not_as_described` / `damaged` / `faulty` / `changed_mind` / `other` |
| `condition` | string | Set at receipt: `restock` / `damaged` |
| `note` | string | |
| `created_at` | datetime | |
| `updated_at` | datetime | |

---

## 10. Discounts

### 10.1 `discounts`
A discount code or an automatic discount (F-S11).

| Column | Type | Notes |
|---|---|---|
| `code` | string | Unique, upper case; empty for an automatic discount |
| `title` | string | Shown to the customer |
| `is_automatic` | bool | True: applied without a code when the discount's conditions are met |
| `type` | string | `percentage` / `fixed_amount` / `free_shipping` / `buy_x_get_y` |
| `value` | number | Percentage, or amount in cents of the system currency, converted at the zone's exchange rate |
| `applies_to` | string | `order` / `products` / `categories` / `collections` |
| `product_ids` | links → products | For `products` |
| `category_ids` | links → product_categories | For `categories` |
| `collection_ids` | links → product_collections | For `collections` |
| `buy_quantity` | number | For `buy_x_get_y`: units to buy |
| `get_quantity` | number | For `buy_x_get_y`: units given at the `value` percentage off (100 = free) |
| `min_order_amount` | number | Optional, in the system currency; the items total needed, converted at the zone's exchange rate |
| `segment_id` | link → segments | Optional; only customers in this CRM segment |
| `first_order_only` | bool | Only a customer's first order |
| `usage_limit` | number | Optional; total uses allowed |
| `usage_limit_per_customer` | number | Optional |
| `used_count` | number | |
| `combinable` | bool | True: may be used with other discounts |
| `status` | string | `draft` / `active` / `ended` |
| `starts_at` | datetime | |
| `ends_at` | datetime | |
| `created_at` | datetime | |
| `updated_at` | datetime | |
| `metadata` | simpleObject | Client-level attributes (common aspect 4) |

### 10.2 `discount_redemptions`
Each use of a discount by an order (F-S11).

| Column | Type | Notes |
|---|---|---|
| `discount_id` | link → discounts | |
| `order_id` | link → orders | |
| `contact_id` | link → contacts | |
| `amount` | number | In cents of the order's currency |
| `status` | string | `applied` / `reversed` (the order was cancelled) |
| `created_at` | datetime | |
| `updated_at` | datetime | |

---

## 11. Credit Accounts

### 11.1 `credit_applications`
A customer's application for a credit account, made through the standard credit application form (F-S13).

| Column | Type | Notes |
|---|---|---|
| `contact_id` | link → contacts | The applicant |
| `company_name` | string | |
| `registration_number` | string | Company registration number |
| `vat_number` | string | |
| `trading_address` | simpleObject | Keys as `stock_locations.address` |
| `years_trading` | number | |
| `accounts_contact` | simpleObject | Keys `name`, `email`, `phone`: who pays invoices |
| `trade_references` | simpleObject | List of up to three references, each with `company`, `contact`, `email`, `phone` |
| `requested_limit` | number | In cents of the system currency |
| `requested_terms_days` | number | e.g. `30` |
| `status` | string | `submitted` / `approved` / `declined` / `withdrawn` |
| `decision_note` | string | |
| `decided_by` | link → users | |
| `submitted_at` | datetime | |
| `decided_at` | datetime | |
| `created_at` | datetime | |
| `updated_at` | datetime | |
| `metadata` | simpleObject | Client-level attributes (common aspect 4); extra form fields |

### 11.2 `customer_accounts`
A customer's approved credit account: the terms on which the customer pays on invoice (F-S13).

| Column | Type | Notes |
|---|---|---|
| `contact_id` | link → contacts | One account per contact |
| `application_id` | link → credit_applications | |
| `credit_limit` | number | In cents of the system currency |
| `payment_terms_days` | number | |
| `balance_outstanding` | number | In cents of the system currency; unpaid invoices of pay-on-invoice orders, recomputed with each order and payment |
| `status` | string | `active` / `on_hold` / `closed` |
| `hold_reason` | string | |
| `created_at` | datetime | |
| `updated_at` | datetime | |
| `metadata` | simpleObject | Client-level attributes (common aspect 4) |

---

## 12. Gift Cards

### 12.1 `gift_cards`
A gift card sold by the shop or issued by a team member (F-S16).

| Column | Type | Notes |
|---|---|---|
| `code` | string | Unique, random, 16 characters; shown in full only to the purchaser and the recipient |
| `currency` | string | The currency of the order that bought the gift card; the gift card is spent only in zones with this currency |
| `initial_amount` | number | In cents of the gift card's currency |
| `balance` | number | In cents of the gift card's currency |
| `status` | string | `active` / `used` / `expired` / `disabled` |
| `expires_at` | datetime | Optional; empty: no expiry |
| `order_item_id` | link → order_items | The purchase; empty when issued by a team member |
| `purchaser_contact_id` | link → contacts | |
| `recipient_email` | string | |
| `recipient_message` | string | |
| `issued_by` | link → users | For a gift card issued by a team member |
| `created_at` | datetime | |
| `updated_at` | datetime | |

### 12.2 `gift_card_transactions`
Every change to a gift card's balance (F-S16).

| Column | Type | Notes |
|---|---|---|
| `gift_card_id` | link → gift_cards | |
| `change` | number | In cents; negative when spent, positive when issued or refunded |
| `reason` | string | `issued` / `spent` / `refunded` / `adjusted` / `expired` |
| `order_id` | link → orders | For `spent` and `refunded` |
| `user_id` | link → users | For `adjusted` |
| `created_at` | datetime | |

---

## 13. Reviews and Performance

### 13.1 `product_reviews`
Attaches a common review to the product the review is about (F-S06).

| Column | Type | Notes |
|---|---|---|
| `review_id` | link → reviews | The review, with the review's rating, text and approval |
| `product_id` | link → products | |
| `order_item_id` | link → order_items | The purchase reviewed; marks the review as a verified purchase |
| `created_at` | datetime | |

### 13.2 `product_performance`
Daily sales figures per product, written by a nightly task for the dashboard and reports (F-S07).

| Column | Type | Notes |
|---|---|---|
| `date` | date | |
| `product_id` | link → products | |
| `units_sold` | number | |
| `revenue` | number | In cents of the system currency, net of discounts, excluding tax |
| `units_returned` | number | |
| `refunded` | number | In cents of the system currency |
| `views` | number | Product page views |
| `add_to_cart_count` | number | |
| `created_at` | datetime | |
| `updated_at` | datetime | |

---

*End of `mb-4-ecom-commerce-schema`*
