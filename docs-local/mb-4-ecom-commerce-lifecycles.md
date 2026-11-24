---
title: "mb-4-ecom-commerce-lifecycles"
updated: "2026-11-24"
---
# Mybizz Shop — Commerce Lifecycles

The states every shop record passes through, what moves the record from one state to the next, and what each state allows. Tables and columns are those of `mb-4-ecom-commerce-schema.md`; feature numbers are those of `mb-4-ecom-commerce-feature-list.md`.

**Part:** trade.

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

## Rules for every lifecycle

- **One server function per transition.** Each transition below is one server function. The function checks the current state, applies the change and writes the `activity_log` entry, all inside one Data Tables transaction. A transition that is not listed is refused with a clear message.
- **Terminal states stay terminal.** A record in a terminal state is never changed again, except where a lifecycle names an exception.
- **Payments come from the payment provider.** A payment is `succeeded` only when the payment provider's signed webhook says so, never because the browser says so. Webhooks follow [[[default:adr-global/adr-webhook-architecture|Webhook Architecture]]](C:/dev/project-library-global/adr-global/adr-webhook-architecture.md): answered at once, processed in a background task, and refused when the webhook's event id has already been processed.
- **Timed changes run as scheduled tasks.** Every "after N minutes" or "when the date passes" below is a scheduled task that runs every 5 minutes, unless the lifecycle states another interval.

---

## 1. Product

`products.status`: `draft`, `published`, `archived`.

- `draft` → `published`: the product has a title, a handle, at least one active variant with a price, and an image. Sets `published_at`.
- `published` → `draft`: hidden from the storefront; carts holding the product's variants show the product as unavailable at the cart's next change.
- `draft` / `published` → `archived`: hidden everywhere except in past orders and reports.
- `archived` → `draft`: brought back for editing.

A product is deleted only when the product has never been in a cart or an order. Otherwise the product is archived.

---

## 2. Stock reservation

`stock_reservations.status`: `held`, `committed`, `released`.

- **Created `held`** when a cart goes to payment, one row per cart line of a variant with `manage_stock` set. `stock_levels.reserved` rises by the quantity. `expires_at` = now + `shop_settings.cart_hold_minutes`.
- `held` → `committed`: the cart's payment succeeds and the order is placed. The row gains `order_item_id`.
- `held` → `released`: the hold expires, the payment fails, or the customer goes back to the cart. `reserved` falls by the quantity.
- `committed` → `released`: the order line is shipped (the shipment also lowers `on_hand`) or the order is cancelled. `reserved` falls by the quantity.

**Choosing the location.** Stock is reserved at `shop_settings.default_stock_location_id` when that location holds enough. Otherwise the stock is reserved at the active location with the most available stock. For a collection order, the stock is reserved at the pickup location.

**Backorder.** A variant with `allow_backorder` set may be reserved beyond the variant's available stock. The order line is then shown to team members as awaiting stock.

**Low-stock alert.** When available stock falls to or below the variant's threshold, the Owner and Managers receive one notification, and `low_stock_alerted_at` is set. The alert is not repeated until stock has risen above the threshold.

---

## 3. Cart

`carts.status`: `active`, `at_payment`, `completed`, `abandoned`, `expired`.

- **Created `active`** when a customer first adds an item. The browser keeps `cart_token`. A signed-in customer's cart also carries `contact_id`, so the cart follows the customer to another device.
- **Currency.** The cart's currency is the currency of the zone of the delivery address, or of the customer's country before an address is given. Prices come from an active price list in that currency, or else from the base price converted at the zone's exchange rate and rounded by the zone's rounding rule. The customer sees, pays and is invoiced in the cart's currency.
- **Every change** (item added, quantity changed, address, delivery option or discount code changed) recomputes the line prices, discounts, delivery charge, tax and totals on the server, in that order. The browser never sends a price.
- **Tax.** With `shop_settings.tax_engine` `manual`, tax comes from the zone's rate for each line's tax class. With `stripe_tax`, the server sends the lines, delivery charge and address to the Stripe Tax calculation service once the address is complete, and uses the tax Stripe Tax returns. A business customer who gives a valid VAT number is charged tax as Stripe Tax returns the tax (reverse charge where reverse charge applies); with `manual`, a business customer abroad with a VAT number is charged no tax. With tax-inclusive prices, the price stays the same and the tax inside the price follows the zone.
- **Who is buying.** When `shop_settings.sells_to` is `both`, checkout asks whether the customer is buying as a private person or for a business. A business customer gives a company name and may give a VAT number. When `sells_to` is `consumers` or `businesses`, the question is not asked.
- `active` → `at_payment`: the customer presses Pay, having accepted the Terms, the Privacy Policy, the Returns Policy and the Delivery Policy, and, for a consumer buying digital items in a zone whose rule set requires the waiver, having ticked the waiver that releases the items before the cancellation period ends. The server checks that the shop is open, that every line is still for sale, that the delivery option applies to the address, and that every discount is still valid; the server then reserves stock (section 2) and opens a payment with the payment provider (section 5). If any check fails, the cart stays `active` and the customer sees what changed.
- `at_payment` → `active`: the customer returns to the cart, or the payment fails. Reservations are released.
- `at_payment` → `completed`: the payment succeeds. The order is placed (section 4) and `order_id` is set.
- `at_payment` → `active` on hold expiry: when `expires_at` passes without a succeeded payment, reservations are released. A payment that succeeds after this runs the late-payment path in section 5.
- `active` → `abandoned`: the cart holds an email and no change for `shop_settings.abandoned_cart_hours`. The abandoned-cart email is sent once, through the common email sequences, with a link that restores the cart. `abandoned` → `active` when the customer returns through that link or changes the cart.
- `active` / `abandoned` → `expired`: no change for 30 days. Terminal.
- `completed` and `expired` are terminal.

---

## 4. Order

An order has three statuses, each answering one question:

| Column | Question | Values |
|---|---|---|
| `status` | Where is the order as a whole? | `draft`, `placed`, `completed`, `cancelled` |
| `payment_status` | Has the order been paid? | `unpaid`, `paid`, `partially_refunded`, `refunded` |
| `fulfilment_status` | Has the order been sent? | `not_shipped`, `partially_shipped`, `shipped`, `delivered` |

### 4.1 Status

- **Created `placed`** when an online cart's payment succeeds, or when gift cards cover the cart's whole total. In the same transaction: the order number is taken from `shop_counters`, the order's currency and the zone's exchange rate are recorded and `total_system` is worked out, the lines are copied with the lines' snapshots, reservations are committed, discount redemptions are recorded and `used_count` raised, the customer is matched to a contact by email or a new contact is created (source `order`), the paid invoice is issued through the common invoicing feature, and the order confirmation email is queued.
- **Created `placed` on invoice** when a signed-in customer with an `active` credit account chooses Pay on invoice and the order total, converted to the system currency at the zone's exchange rate, plus `customer_accounts.balance_outstanding` is within `credit_limit`. No payment provider payment is opened. In one transaction the order is placed as above, with `payment_status` `unpaid` and `pay_on_invoice` set; the invoice is issued with the account's payment terms and sent; and the account's balance rises. The common invoicing feature sends overdue reminders, and the recorded payment of the invoice moves the order to `paid` and lowers the balance. When the total would pass the limit, Pay on invoice is not offered for that order.
- **Created `draft`** when a team member builds an order for a customer (by phone, in person or for a trade customer). A draft holds no stock and can be edited freely.
- `draft` → `placed`: the team member places the order. Stock is reserved and committed at once. The order is either paid by a payment link sent to the customer, or marked paid by cash or bank transfer through the common payments feature.
- `placed` → `completed`: every physical line is delivered or collected, and every digital line has been made available. Sets `completed_at`. Returns remain possible within the return window.
- `draft` → `cancelled`: discarded.
- `placed` → `cancelled`: only while nothing has been shipped. Reservations are released, discount redemptions are reversed and `used_count` lowered, a paid order is refunded in full (section 5), and the customer is emailed. Sets `cancelled_at` and `cancel_reason`.
- `completed` and `cancelled` are terminal.

### 4.2 Payment status

- `unpaid` → `paid`: the order's payments reach the order total.
- `paid` → `partially_refunded`: a refund less than the amount paid succeeds.
- `paid` / `partially_refunded` → `refunded`: refunds reach the amount paid.

### 4.3 Fulfilment status

Worked out from the order's shipments every time a shipment changes:

- `not_shipped`: no line shipped.
- `partially_shipped`: some physical units shipped, some not.
- `shipped`: every physical unit shipped.
- `delivered`: every shipment delivered or collected.

An order with only digital lines goes straight to `delivered` when the order is placed.

### 4.4 Permitted actions in the order viewer

| Status | Edit lines | Place | Create shipment | Mark delivered | Refund | Start return | Cancel | Resend confirmation |
|---|---|---|---|---|---|---|---|---|
| Draft | Yes | Yes | No | No | No | No | Yes | No |
| Placed, not shipped | No | No | Yes | No | Yes | No | Yes | Yes |
| Placed, partly or fully shipped | No | No | Yes, for unshipped units | Yes | Yes | Yes | No | Yes |
| Completed | No | No | No | No | Yes | Yes, within the return window | No | Yes |
| Cancelled | No | No | No | No | No | No | No | No |

---

## 5. Payment

The shop uses the common payments feature and the payment providers Stripe and PayPal. Every payment and refund is a row of the common `payments` table, linked from `orders.payment_ids`.

- **Opening a payment.** When the cart goes to payment, the server creates a checkout with the payment provider for the cart's total, less any gift card amount, in the cart's currency and stores the checkout reference in `carts.gateway_session_ref`. Card details never reach Mybizz ([[[default:adr-global/adr-payment-security-boundary-vault|Payment Security Boundary]]](C:/dev/project-library-global/adr-global/adr-payment-security-boundary-vault.md)).
- **Charged at once.** The payment is captured when the customer pays. Mybizz does not hold authorised-but-uncaptured payments.
- **Tax record.** With `stripe_tax`, the order's tax calculation is recorded as a Stripe Tax transaction when the order is placed, and reversed in proportion with every refund, so the client's tax reports in Stripe match the shop.
- **Succeeded.** The payment provider's webhook marks the payment `succeeded`. The webhook's background task places the order (section 4.1) in one transaction.
- **Failed.** The payment is `failed`, the cart returns to `active` and the customer may try again.
- **Late payment.** If a payment succeeds after the cart's hold has expired, the server tries to reserve the stock again inside the same transaction that places the order. If the stock is still available, the order is placed as normal. If not, the payment is refunded in full at once, and the customer and the Owner are emailed.
- **Refunds.** A refund is made from the order or from a return, for any amount up to the amount paid less earlier refunds. The refund is made in the order's currency, through the payment provider that took the payment, or recorded as a manual refund for cash or bank transfer, and is a `payments` row of `kind` `refund`. `orders.refunded_total` and `payment_status` follow.

---

## 6. Shipment

`shipments.status`: `preparing`, `shipped`, `delivered`, `cancelled`.

- **Created `preparing`** when a team member chooses which unshipped units of an order go in a parcel, and from which location. A packing slip can be printed.
- **Buying the label.** When the order's delivery option uses a courier provider (Sendcloud or Shippo), the team member presses Buy label: the server sends the parcel's weight, size, addresses and service to the courier provider, and stores the returned label, tracking number, tracking link and label cost on the shipment. The label prints from the shipment. The courier provider charges the client's own account. When no courier provider is connected, the team member types the carrier and tracking number.
- `preparing` → `shipped`: the team member enters the carrier and tracking number (or the collection is handed over). In one transaction, `on_hand` falls at the shipment's location, the reservations are released, `stock_movements` rows of reason `shipped` are written, `order_items.shipped_quantity` rises, and the customer receives the shipping email with the tracking link.
- `shipped` → `delivered`: set when the courier provider's tracking update reports delivery; marked by a team member; or automatically 14 days after shipping when neither has happened.
- `preparing` → `cancelled`: the parcel is not sent; the parcel's units become unshipped again.
- `delivered` and `cancelled` are terminal.

**Collection orders** follow the same states: `shipped` means "ready for collection" (the customer is emailed), and `delivered` means collected.

**Digital lines** need no shipment. When the order is placed, the customer receives a download link for each digital line. Each download is served by the server from the client's R2 bucket, counted in `download_count`, and refused after `shop_settings.digital_download_limit` downloads or after `download_expires_at`. A team member can reset the count and the date for a customer.

---

## 7. Return

`returns.status`: `requested`, `approved`, `received`, `closed`, `rejected`, `cancelled`.

- **Created `requested`** by the customer in the customer portal, for delivered units, within `shop_settings.return_window_days` of delivery, with a reason per line; or by a team member at any time.
- **Cancellation by a consumer.** A return requested by a consumer within the `min_cancellation_days` of the order's consumer rule set is a cancellation (`is_cancellation` set). A cancellation needs no reason, cannot be rejected, and is approved automatically with resolution `refund`. The refund covers the items and, when `refund_basic_delivery` is set and every item of the order comes back, the standard delivery charge. `refund_due_by` is set as soon as the goods are received or the customer gives proof of posting, whichever is first; the Owner and Managers are alerted two days before `refund_due_by` passes.
- `requested` → `approved`: a team member approves the return, sets the resolution (`refund` or `replacement`) and, for a refund, the refund amount. The customer is emailed the return address and instructions, and a prepaid return label when `shop_settings.return_shipping_paid_by` is `business` and a courier provider is connected.
- `requested` → `rejected`: a team member declines the return, with a reason emailed to the customer. A cancellation cannot be rejected. Terminal.
- `approved` → `received`: the items arrive. For each line the team member records the quantity received and the condition of the items. `restock` units raise `on_hand` with a `returned` stock movement; `damaged` units are recorded on the return line only and do not change stock. `order_items.returned_quantity` rises.
- `received` → `closed`: the resolution is carried out. For `refund`, the refund is made (section 5) and linked in `refund_payment_id`. For `replacement`, a new order is placed for the same items at a price of zero, linked in `replacement_order_id`, and shipped as normal. The customer is emailed.
- `requested` / `approved` → `cancelled`: withdrawn by the customer or a team member. Terminal.
- `closed`, `rejected` and `cancelled` are terminal.

### 7.1 Permitted actions in the return viewer

| Status | Approve | Reject | Mark received | Close | Cancel |
|---|---|---|---|---|---|
| Requested | Yes | Yes, except a cancellation | No | No | Yes |
| Approved | No | No | Yes | No | Yes |
| Received | No | No | No | Yes | No |
| Closed, Rejected, Cancelled | No | No | No | No | No |

---

## 8. Discount

`discounts.status`: `draft`, `active`, `ended`.

- `draft` → `active`: activated by a team member. A discount with a future `starts_at` is active but does not apply until that time.
- `active` → `ended`: ended by a team member, when `ends_at` passes, or when `used_count` reaches `usage_limit`.
- `ended` → `active`: reactivated by a team member, with a new end date or a raised limit.
- A discount that has been used is never deleted.

**How discounts apply.** On every cart change, automatic discounts whose conditions are met are applied, then entered codes. A discount that is not `combinable` is applied alone: when several apply, the one that saves the customer the most is used. A discount never takes a line or the delivery charge below zero.

---

## 9. Price list

`price_lists.status`: `draft`, `active`, `ended`.

- `draft` → `active`: activated by a team member; the price list applies between `starts_at` and `ends_at`.
- `active` → `ended`: ended by a team member or when `ends_at` passes.
- **Which price applies.** A variant's price in a cart is the lowest of: the variant's base price, and the price in every active price list that applies to the customer (no segment, or a segment the customer belongs to) whose `min_quantity` the line meets.

---

## 10. Credit application and credit account

`credit_applications.status`: `submitted`, `approved`, `declined`, `withdrawn`.
`customer_accounts.status`: `active`, `on_hold`, `closed`.

- **Offered** on the storefront and in the customer portal when `shop_settings.credit_accounts_enabled` is set. Only a signed-in customer may apply; the application form is the standard credit application form of `credit_applications`.
- **Created `submitted`.** The Owner and Managers are notified.
- `submitted` → `approved`: a team member sets the credit limit and payment terms, which may differ from those requested. The customer account is created `active`, and the customer is emailed the terms.
- `submitted` → `declined`: with a note; the customer is emailed. Terminal.
- `submitted` → `withdrawn`: by the customer. Terminal.
- Account `active` → `on_hold`: set by a team member, or automatically when an invoice of the account is 30 days overdue. Pay on invoice is not offered while on hold.
- `on_hold` → `active`: set by a team member, or automatically when no invoice of the account is overdue.
- `active` / `on_hold` → `closed`: set by a team member. Terminal. Unpaid invoices stay due.

---

## 11. Dispute

`order_disputes.status`: `open`, `evidence_submitted`, `won`, `lost`, `accepted`.

- **Created `open`** when Stripe or PayPal reports a dispute or chargeback by webhook. The Owner and Managers are alerted with the evidence deadline, and the order shows a dispute banner.
- **Evidence.** The dispute screen gathers the order, the invoice, the accepted policy versions and acceptance time from the common consents record, the shipment's tracking number and delivery confirmation, and the customer's messages, ready to send to the payment provider.
- `open` → `evidence_submitted`: a team member sends the evidence through the payment provider.
- `open` → `accepted`: a team member accepts the dispute without contest. The disputed amount is recorded as a refund.
- `evidence_submitted` → `won` / `lost`: set by the payment provider's webhook. A lost dispute is recorded as a refund. Terminal.
- The Owner and Managers are reminded two days before `evidence_due_at` while the dispute is `open`.

---

## 12. Tax method

`shop_settings.tax_engine`: `stripe_tax`, `manual`. Only the Owner chooses the tax method, at onboarding or later in Settings.

- **Choosing `stripe_tax`.** The setting shows a link to the Stripe Tax documentation (`https://docs.stripe.com/tax`) beside the choice. The Owner connects Stripe Tax with the client's own Stripe account; the secret key is kept in the Vault. The method takes effect for carts changed after the connection succeeds.
- **Choosing `manual`.** A Material 3 dialog from the m3-custom-theme components opens. The dialog has no close button, does not close on a click outside the dialog or on the Escape key, and has one action: Accept. The dialog states, in plain words, that the client accepts that Mybizz is not responsible for the client's VAT and other tax obligations, and that the client must meet the legal requirements of every transaction the shop makes. The dialog links to the client's legal responsibility acknowledgement; the dialog itself is not a legal document. Accepting sets `tax_responsibility_accepted_at` and `tax_responsibility_accepted_by`, and writes an `activity_log` entry. The method takes effect only after acceptance.
- **Every switch to `manual`** opens the dialog again.

---

## 13. Gift card

`gift_cards.status`: `active`, `used`, `expired`, `disabled`.

- **Sold** as a product of kind `gift_card`, in amounts the client sets. When the order is placed, the gift card is created `active` in the order's currency, and the code is emailed to the recipient, or to the purchaser when no recipient is given. A team member can also issue a gift card, for example as a goodwill gesture; that issue is recorded with the team member.
- **Spent at checkout.** The customer enters the code; the server checks that the gift card is `active`, in the cart's currency and not expired, and applies the lower of the balance and the amount due. The rest is paid through a payment provider, or nothing is due when the gift card covers the whole order. The amount is taken from the balance, with a `spent` transaction, inside the transaction that places the order.
- **Refunds.** A refund of an order paid partly by gift card returns the gift card part to the gift card first, with a `refunded` transaction, and the rest through the payment provider.
- `active` → `used`: the balance reaches zero. `used` → `active` when a refund raises the balance.
- `active` → `expired`: `expires_at` passes; the remaining balance is recorded with an `expired` transaction.
- `active` → `disabled`: a team member stops a lost or misused code. `disabled` → `active`: a team member restores the gift card.
- **Tax.** The sale of a gift card carries no tax: tax is charged on the goods the gift card later pays for. The gift card product's tax class is `exempt`.
- **The money.** A gift card is paid for through the client's own payment provider account, like any order. Mybizz never holds gift card money.

---

## 14. Product feed

- **Built every hour** by a scheduled task, for every enabled feed: one line per published variant of each product with `in_product_feeds` set, with the title, description, link to the product page, image, price and sale price in the feed zone's currency, availability from stock, brand, condition, barcode, Google product category, and the delivery charge and tax of the zone. The file is stored in the client's R2 bucket through `files`.
- **Read by the channel.** The client pastes the feed's web address into Google Merchant Center, Meta Commerce Manager or TikTok Catalog Manager once. Each channel then fetches the feed on the channel's own schedule, and `last_read_at` records each fetch.
- **Selling.** Shoppers who click a product on Google, Facebook, Instagram or TikTok land on the product page of the client's own shop and buy there, through the client's own checkout.
- **Changes reach the channels** within an hour plus the channel's own fetch interval: a price change, a sold-out variant or an archived product.

---

*End of `mb-4-ecom-commerce-lifecycles`*
