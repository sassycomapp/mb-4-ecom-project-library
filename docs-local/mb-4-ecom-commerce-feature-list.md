---
title: "mb-4-ecom-commerce-feature-list"
updated: "2026-11-24"
---
# Mybizz Shop — Commerce Feature List

Every feature of the shop: the trade part of `mb-4-ecom`, the part that makes a Mybizz application an online shop. Feature numbers F-S01 to F-S16 are used by `mb-4-ecom-commerce-schema.md` and `mb-4-ecom-commerce-lifecycles.md`; the `S` marks a shop feature.

**Part:** trade. Everything else a client of `mb-4-ecom` receives (website, team and roles, CRM, email, marketing, invoicing and payments, accounts, AI and voice, customer portal, dashboard, data ownership, security, design and support) is the common part of every Mybizz application, carried by the project template.

**Model.** The shop's domain and workflows follow Medusa (`medusajs.com`, v2.22.1), a proven open-source commerce platform. Mybizz takes the commerce structure that suits small and growing businesses and builds that structure fresh in Python on Anvil, under the common aspects of every Mybizz application.

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

## Products

- **F-S01 Catalogue:** physical and digital products, each with up to three options (such as Size and Colour) and a variant for each combination, with the variant's own SKU, barcode, price, weight, images and stock. Products sit in a category tree and in hand-picked collections, carry tags, and are draft, published or archived. Each product has a web address and search-engine title and description. Products import and export by CSV.
- **F-S02 Pricing:** a base price per variant, with an optional compare-at price shown struck through. Price lists set timed sale prices, or special prices for one CRM segment such as wholesale business customers, with optional bulk prices from a minimum quantity. The customer always gets the lowest price that applies.
- **F-S03 Stock:** stock counted per variant at each location (shop, store room, warehouse). Stock is held for a customer during payment, so the last item is never sold twice. Backorders are allowed per variant. A low-stock alert goes to the Owner and Managers. Every stock change has a reason and a history: goods received, shipped, returned, written off, counted or moved between locations.

## Buying

- **F-S04 Cart:** a cart that follows a signed-in customer across devices, with prices, discounts, delivery and tax worked out on the server at every change. An abandoned cart with an email address triggers one recovery email, through the common email sequences, with a link that restores the cart.
- **F-S05 Checkout:** one page: email, delivery address, private or business customer, delivery option with live courier prices, discount code, order summary, acceptance of the Terms, Privacy, Returns and Delivery policies, and payment through the client's own Stripe or PayPal account, or Pay on invoice for a customer with a credit account, with gift cards accepted as part or full payment. Guest checkout can be switched on or off. Card details never touch Mybizz. Every customer becomes, or is matched to, a contact in the CRM.
- **F-S06 Storefront:** shop home with featured products, category and collection pages with filters (price, option values, tags) and sorting, search, product pages with an image gallery and variant picker, cart and checkout. Verified-purchase product reviews, approved by the client before the reviews show, through the common reviews feature. Prices shown in the customer's currency, with a country selector.

## After the sale

- **F-S07 Orders:** every order with the order number, customer, lines, totals, payments and history, shown with three clear statuses: the order, the payment and the delivery. Team members create orders for phone and in-person customers and for business customers, paid by a payment link, cash or bank transfer. Orders are cancelled, never deleted. Each paid order issues an invoice through the common invoicing feature, so the accounts and tax reports include every sale. The dashboard shows units sold, revenue, returns, views and add-to-cart figures per product.
- **F-S08 Delivery:** delivery options per zone: a flat rate, weight bands, free over a set order value, and collection from the client's own location. Courier providers built in: Sendcloud for the UK and Europe and Shippo for the USA and Canada, each on the client's own account, switched on if the client chooses. With a courier provider connected, checkout shows live courier prices, team members buy and print labels from the order, tracking updates arrive automatically, and prepaid return labels are sent to customers. Royal Mail, Evri, DPD, DHL, UPS, FedEx and many more are reached this way. Without a courier provider, team members type the carrier and tracking number. Orders ship in one parcel or several, a tracking link is emailed to the customer, and packing slips print from the order. Digital products are delivered as download links with a download limit and an expiry date.
- **F-S09 Returns and refunds:** customers request a return in the customer portal within the client's return window, with a reason per item. Team members approve the return, receive and inspect the items, return good stock to the shelf, and close the return with a refund or a free replacement. Full and partial refunds go back through the payment provider that took the payment. A private customer's cancellation within the statutory period of the customer's country needs no reason and cannot be refused, and the refund deadline the law sets is tracked and alerted.

## Rules of sale

- **F-S10 Tax:** two ways to work out tax, chosen at onboarding and changeable in Settings: the shop's own rate tables, or Stripe Tax on the client's own Stripe account, which works out the correct rate for every address and records every sale and refund for the client's tax returns, whichever payment provider took the payment. VAT numbers of business customers are accepted, with reverse charge where reverse charge applies. Choosing Stripe Tax shows a link to the Stripe Tax documentation. Choosing the shop's own tables opens a dialog that cannot be closed until the Owner clicks Accept, confirming that Mybizz is not responsible for the client's VAT obligations; the dialog links to the client's legal responsibility acknowledgement. Prices are entered with tax included or excluded, as the client chooses. Each product has a tax class (standard, reduced, zero or exempt); with the shop's own tables, each zone has a rate for each class, including the rate charged on delivery. With tax included, the customer pays the same price in every zone, and the tax inside the price is worked out at the zone's rate. Tax is shown on every order and invoice with the client's tax label and registration number.
- **F-S11 Discounts:** codes and automatic discounts: a percentage off, a fixed amount off, free delivery, or buy X get Y. A discount can apply to the whole order or to chosen products, categories or collections, with a minimum order value, a CRM segment, first orders only, start and end dates, a total usage limit and a per-customer limit. Team members choose whether a discount combines with others.
- **F-S12 Zones and currency:** the client groups the countries the shop sells to into zones, each zone with the zone's own tax rates and delivery options. The client prices every product in the system currency, and on the Settings page turns on a local currency for any market, such as US dollars for the USA, with local prices the client types or works out at the client's own rate. Customers in that market see, pay and are refunded in that currency, at exactly the price shown; every other customer pays in the system currency. Each sale enters the client's books at the system-currency amount Stripe or PayPal pays in. An address outside every active zone cannot check out.

## Selling further

- **F-S15 Product feeds:** the shop's products listed on Google Shopping, Facebook, Instagram and TikTok through product feeds, one per channel and target country, rebuilt every hour with prices, stock, images and delivery charges. The client pastes each feed's web address into the channel once. Shoppers who click a product on any channel buy in the client's own shop.
- **F-S16 Gift cards:** gift cards sold in the shop in amounts the client sets, emailed to the recipient with a message, and spent at checkout as part or full payment. Team members can issue gift cards and see every gift card's balance and history. Gift card money is paid into the client's own payment provider account, like any sale.

## Business customers and the law

- **F-S13 Credit accounts:** customers apply for a credit account through a standard credit application form: company, registration and VAT numbers, trading address, accounts contact, trade references, and the credit limit and payment terms requested. The client approves or declines, and sets each customer's limit and terms. Approved customers pay on invoice at checkout, within the customer's credit limit; invoices carry the customer's payment terms, overdue reminders go out automatically, and an account with an invoice 30 days overdue is put on hold. Credit accounts are switched on or off by the client.
- **F-S14 Legal terms and disputes:** at onboarding the client chooses who the shop sells to (private customers, business customers or both), and the Terms are governed by the client's own country's law. Private customers in each zone receive at least the consumer protections of the customer's own country, from rule sets Mybizz supplies for the UK, the EU and other regions; the client may be more generous, never less. The Clause Builder adds a Returns Policy and a Delivery Policy to the client's legal documents, linked at checkout and in the payment providers' checkout settings. Digital items carry the customer's waiver of the cancellation period where the law requires the waiver. Stripe and PayPal disputes arrive automatically, with the evidence deadline, an alert, and an evidence pack built from the order, the accepted policies and the tracking record. The client accepts responsibility for the client's own policies through the common legal acknowledgement at onboarding.

---

## What the shop uses from the common part

| Common feature | What the shop uses |
|---|---|
| Contacts and CRM | Every customer is a contact; orders count towards lifetime value and total transactions; segments limit price lists and discounts |
| Email and sequences | Order confirmation, shipping, return and abandoned-cart emails, sent from the client's own domain |
| Invoicing and payments | Stripe and PayPal, payments and refunds, the invoice issued for every order, payment terms and overdue reminders |
| Reviews | Product reviews, approved by the client |
| Legal documents and consents | The Clause Builder, the legal acknowledgement at onboarding, and the record of every acceptance at checkout |
| Customer portal | My Orders, order tracking, downloads and return requests |
| Dashboard and reports | Shop figures alongside the common revenue and customer figures |
| Files and R2 storage | Product images and digital product files |
| Sample data | A shop sample set: a catalogue, stock, customers and three months of orders |

---

## Outside the shop part

- **Marketplace selling** (Amazon, eBay, Takealot), **company buying teams** (several people buying on one business customer's account, with spending limits and approvals) and **built-in demand forecasting** are product directions of `mb-4-ecom` (`mb-4-ecom-architecture-report.md`, section 9).
- **Tax filing.** The shop works out and records tax; the client files the client's own tax returns, or has a filing service or an accountant file the returns.
- **Selling inside other platforms' own checkouts** (TikTok Shop, marketplace checkouts): customers buy in the client's own shop.
- **Subscriptions and repeat orders,** which belong to a separate Mybizz business.

---

*End of `mb-4-ecom-commerce-feature-list`*
