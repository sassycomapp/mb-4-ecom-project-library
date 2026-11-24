---
title: "mb-4-ecom-concept"
updated: "2026-11-24"
---
# Mybizz Shop — Concept

**Online shop and back office · mybizz.live**

---

## Purpose of this document

This document is the concept of Mybizz Shop: what the business is, who Mybizz Shop serves, what the product does, the rules Mybizz Shop never breaks, the price and the market position. Every statement in this document is a decision.

The shop features are named and numbered (F-S01 to F-S14) in `mb-4-ecom-commerce-feature-list.md`, specified in `mb-4-ecom-commerce-schema.md` and `mb-4-ecom-commerce-lifecycles.md`, and placed in the architecture by `mb-4-ecom-architecture-report.md`.

---

## 1. What Mybizz Shop Is

Mybizz Shop is an online shop and a complete back office for small product businesses, sold as a subscription. Mybizz Shop is one of the Mybizz businesses, with the slug `mb-4-ecom`.

Mybizz Shop is built from two parts:

- **The common part**, shared by every Mybizz application and carried by the project template: website, blog and landing pages, CRM, email on the client's own domain with a team inbox, marketing sequences, invoicing and payments, built-in accounts, the Clause Builder, reviews, AI assists and voice, the customer portal, the dashboard, backups and data ownership, security, design and support.
- **The shop part** (the trade part): catalogue, pricing, stock, cart, checkout, orders, delivery with courier providers, returns, tax, discounts, zones, credit accounts for business customers, and dispute handling.

**Promise:** one subscription runs the whole shop business: the shop, the customers, the marketing, the email and the books, with nothing extra to buy.

---

## 2. The Problem Mybizz Shop Solves

- A small product business pays for a shop platform, then for a stack of add-on apps: email marketing, reviews, customer service inbox, accounting, legal policies, tax calculation, label printing and business email. The add-ons often cost more than the shop platform.
- The add-ons do not share one customer record, so the business has no single view of a customer.
- Shop platforms add a surcharge on every sale when the business uses Stripe or PayPal instead of the platform's own payment service.
- Business customers who buy on account (30-day terms, credit limits) are poorly served by consumer shop platforms.
- The business does not own the business's data: leaving the platform means losing history.

---

## 3. Who Mybizz Shop Serves

- **Small product businesses** in the UK, Europe, the USA and Canada that sell physical or digital goods online: makers, independent brands, specialist retailers and wholesalers.
- **Size:** 1 to 20 team members, selling roughly £3,000 to £150,000 a month online. Below that range, a free or entry-level shop builder is enough. Above that range, a business needs enterprise platforms.
- **Sales model:** direct to private customers, to business customers on account, or both.
- **Typical client:** an owner-run business that already sells, is tired of paying for and connecting many apps, and wants one system the team can run.

---

## 4. Parties

| Party | Who |
|---|---|
| Mybizz | The business that provides Mybizz Shop to clients |
| Client | The business that subscribes to Mybizz Shop and runs a shop |
| Team member | A person who works for the client and signs in to the client's application |
| Customer | A person or a business that buys from the client's shop; a private customer buys as a private person, a business customer buys for a business |
| Provider | An outside service the client connects with the client's own account: Stripe, PayPal, Stripe Tax, Sendcloud, Shippo, Cloudflare |

---

## 5. Hard Rules

- **One price, everything included.** No add-ons, no feature tiers, no per-user charges.
- **No Mybizz transaction fee.** Mybizz takes nothing from the client's sales; the client pays only the providers' own fees.
- **The client's own provider accounts.** Payments, tax calculation and courier labels run on accounts the client owns. Mybizz never holds the client's money.
- **The client owns the client's data,** kept in the client's own Cloudflare R2 storage, and leaves with all of the data at any time.
- **One client, one instance, one database.** No client shares a database with another client.
- **The client is responsible for the client's legal and tax obligations,** accepted at onboarding through the legal responsibility acknowledgement, and again through the tax responsibility dialog when the client chooses the shop's own tax tables.
- **Consumer protections are never reduced** below the rule set of a private customer's country.

---

## 6. The Product

The product is the common part of every Mybizz application plus the shop part. The shop part is F-S01 to F-S14 of `mb-4-ecom-commerce-feature-list.md`:

| Feature | Name |
|---|---|
| F-S01 | Catalogue |
| F-S02 | Pricing |
| F-S03 | Stock |
| F-S04 | Cart |
| F-S05 | Checkout |
| F-S06 | Storefront |
| F-S07 | Orders |
| F-S08 | Delivery |
| F-S09 | Returns and refunds |
| F-S10 | Tax |
| F-S11 | Discounts |
| F-S12 | Zones and currency |
| F-S13 | Credit accounts |
| F-S14 | Legal terms and disputes |

The common part is defined by the project template when Mybizz CS is finished. The features of the common part that the shop uses are listed in `mb-4-ecom-commerce-feature-list.md`, section "What the shop uses from the common part".

---

## 7. Price and Terms

- **£119 a month, or £1,190 a year** on the annual plan (two months free), charged in the client's own currency at the Mybizz price points.
- Unlimited team members at no extra cost.
- Month-to-month, no contract; cancel at any time, effective at the end of the paid month.
- No Mybizz transaction fee and no add-ons.
- Assisted setup, where Mybizz loads the catalogue and configures the shop, costs one month's subscription, once.
- The same price as Mybizz CS. Every Mybizz business application with the full common part carries the same price, so the Mybizz price list stays one rule: £45 for the front desk (Mybizz FDA), £119 for a full business application.

---

## 8. Market Position

**Position:** the all-in-one shop for small businesses that are paying for a shop platform plus a stack of apps. Mybizz Shop is priced above entry-level shop builders and below the real monthly cost of a shop platform with the apps a working shop needs.

**The market at 2026-11 (UK prices, annual billing):**

| Competitor | Shop plan | What the client adds on top |
|---|---|---|
| Shopify | Basic £19, Grow £49, Advanced £259 a month; monthly billing £25, £65 and £344 | Apps (email marketing, reviews, customer service, accounting, legal policies), business email, and a 2% (Basic) to 0.6% (Advanced) surcharge on every sale paid through Stripe or PayPal |
| Wix | Core £16, Business £25, Business Elite £119 a month | Business email, accounting, CRM depth, apps |
| Squarespace | Core £17, Plus £29, Advanced £79 a month | Business email, accounting, CRM, email marketing at scale |

UK shop owners on Shopify commonly spend a further £50 to £500 a month on apps, themes, domain and business email, beyond the plan itself.

**The comparison Mybizz Shop sells on:** a small UK shop on Shopify Grow (£49 to £65 a month) with an email marketing app, a reviews app, a customer service inbox, accounting software and business email for a small team pays roughly £150 to £250 a month, plus a 1% surcharge on every Stripe or PayPal sale. Mybizz Shop is £119 a month with all of that included and no surcharge.

**Mybizz Shop does not compete** with the £5 to £25 entry plans. A business at that level is not the client.

**What sets Mybizz Shop apart:**

- Everything in one system, with one customer record across shop, email, marketing, invoices and accounts.
- Credit accounts and pay on invoice for business customers, with a standard credit application form.
- Unlimited team members and a team inbox on the client's own domain.
- The client's own data in the client's own storage, kept on leaving.
- AI assists and voice across the business.
- No transaction surcharge for using Stripe or PayPal.

---

## 9. What Mybizz Shop Is Not

- Not a marketplace: each client runs the client's own shop on the client's own domain.
- Not a merchant of record: the client is the seller, collects the client's own payments and accounts for the client's own tax.
- Not an enterprise platform: no headless storefronts, no custom code by clients.
- Not a builder for hobby sellers.

---

## 10. Architectural Parameters

`mb-4-ecom-architecture-report.md` states the architecture: the five-app structure, one instance and one database per client, client data in the client's own R2 storage, the heartbeat to App 5, resilience and Anvil's accepted limits. Client instances (App 4) are hosted as every Mybizz client instance is hosted.

---

## 11. How Mybizz Shop Is Built

1. Mybizz CS is finished through PDLF, and the common part of Mybizz CS is made general in the project template.
2. `mb-4-ecom` is set up from the project template.
3. PDLF builds the shop part from the three shop documents and this concept, and runs `mb-4-ecom` through every phase to release.

---

## 12. Success Measures

- Paying clients, and the share of clients on the annual plan.
- Monthly client churn below 3%.
- Time from sign-up to first order taken in the client's shop: under 14 days.
- Share of clients connecting a courier provider and Stripe Tax.
- Client instance health in App 5: no client instance at the limit of the instance's plan tier without a raise scheduled.

---

## 13. Risks and How Mybizz Shop Answers Them

| Risk | Answer |
|---|---|
| Shopify's brand and app choice draw the market | Mybizz Shop sells on total monthly cost and one system, not on the shop alone; the comparison in section 8 is the core sales message |
| A client's sales spike slows the client's shop | Each client runs in a separate instance; App 5 flags instances near the plan tier's limit, and the tier is raised before high-traffic events |
| Tax and consumer law vary by country | Stripe Tax for automatic rates, consumer rule sets per zone, and the client's acceptance of the client's own legal and tax responsibility |
| Providers change terms or prices | The client's own provider accounts carry the providers' terms; Mybizz Shop supports two payment providers and two courier providers, so no single provider is essential |
| Disputes and chargebacks | Disputes arrive automatically with deadlines, alerts and an evidence pack built from the order, the accepted policies and the tracking record |

---

*End of `mb-4-ecom-concept`*
