---
title: "mb-4-ecom-architecture-report"
updated: "2026-11-24"
doc-id: "mb-c54ed60b"
---
# mb-4-ecom — Architecture Report

## What this document is

This report states the architecture decisions and facts that govern `mb-4-ecom`, the e-commerce application of Mybizz. This report is part of the context `mb-4-ecom` brings into PDLF. Everything `mb-4-ecom` shares with other Mybizz applications is governed by [[[default:policy-global/pol-common-aspects-for-mybizz-anvil-applications|Common Aspects for Mybizz Anvil Applications]]](C:/dev/project-library-global/policy-global/pol-common-aspects-for-mybizz-anvil-applications.md); this report states only how those aspects apply to e-commerce and what is particular to e-commerce.

The shop itself is specified in three documents: `mb-4-ecom-commerce-feature-list.md`, `mb-4-ecom-commerce-schema.md` and `mb-4-ecom-commerce-lifecycles.md`.

## Parties

| Party | Who |
|---|---|
| Mybizz | The business that provides the application to clients |
| Client | The business that subscribes to Mybizz and runs a shop |
| Customer | A person or a business that buys from a client's shop |
| Provider | An outside service the client connects with the client's own account: Stripe, PayPal, Stripe Tax, Sendcloud, Shippo, Cloudflare |

## 1. What mb-4-ecom is

`mb-4-ecom` is Mybizz Shop, the e-commerce application that Mybizz provides to businesses that sell products. The concept, price and market position are in `mb-4-ecom-concept.md`. Each client receives an online shop and a back office: catalogue, cart, checkout, orders, delivery, returns, customers, discounts and payments. `mb-4-ecom` is built on Anvil.works, in Python, by PDLF.

`mb-4-ecom` shares the architecture and infrastructure of `mb-3-cs` (Mybizz Consulting and Services). The two applications differ in the trade part: `mb-3-cs` centres on scheduling and bookings, `mb-4-ecom` on the shop. Whatever can be common between the two applications is kept common, in both directions ([[[default:adr-global/adr-common-architecture-across-mybizz-applications|Common Architecture Across Mybizz Applications]]](C:/dev/project-library-global/adr-global/adr-common-architecture-across-mybizz-applications.md)).

## 2. The reference platform

The shop's commerce structure follows Medusa (`medusajs.com`, v2.22.1), an open-source commerce platform written in TypeScript on Node.js. Mybizz takes only the commerce structure, the domain and the workflows; no Medusa code is used. The three shop documents hold the full design, so Medusa is not part of the context.

**How Medusa's layers map to Anvil:**

| Medusa | In `mb-4-ecom` |
|---|---|
| API routes | Anvil server functions called by the application's forms; HTTP endpoints for provider webhooks and the management endpoints |
| Workflows | Server functions built on pure-logic modules, so the business rules are tested separately |
| Commerce modules and data models | Anvil Data Tables with model classes; behaviour that belongs to one table lives on the table's model class |
| Event bus and subscribers | Anvil background tasks |
| Scheduled jobs | Anvil scheduled tasks |
| Locking | Anvil Data Tables transactions |
| File storage | The client's own R2 buckets |
| Notifications | Anvil email and Brevo |
| Analytics | The application's own metrics tables, reported to App 5 |
| Admin dashboard | The application's admin forms, in Material 3 |

## 3. The five-app structure for e-commerce

`mb-4-ecom` uses the five-app structure exactly as every Mybizz application does ([[[default:specifications-global/spec-five-app-architecture-model|Five-App Architecture Model]]](C:/dev/project-library-global/specifications-global/spec-five-app-architecture-model.md)).

- **Never multi-tenant.** Each client shop is a separate client instance (App 4) with a separate database ([[[default:specifications-global/spec-client-instance-hosting|Client Instance Hosting]]](C:/dev/project-library-global/specifications-global/spec-client-instance-hosting.md)).
- **Where a client instance runs.** Each client instance is a Docker container running the Anvil App Server on a Mybizz Hetzner Cloud application server (`CPX42`: 8 vCPU, 16 GB memory), with the client's own database in Mybizz's Ubicloud managed PostgreSQL service. An application server holds 12 places; a standard container has 1 GiB of memory and takes one place, and App 5 raises a busy container to 2 GiB (two places) automatically. Each client database user holds at most 3 database connections, with a 60-second statement timeout.
- **Load stays in each client instance.** A client's transactions run in the client's own instance. App 2 is a code dependency and carries no client traffic, so 100 clients running 100 transactions each place 100 transactions on each instance and none on App 2.
- **Isolation.** One client's traffic, fault or heavy query cannot slow down or expose another client.
- **Mybizz Shop Enterprise.** A client whose shop needs more than a standard container runs on an application server of the client's own, with the container's memory and the client's database connections sized to the shop. The client instance is built from the same release image as every other client instance, so App 2 and every release stay the same.

## 4. Data design

- The schema is designed in App 1, published through App 2 and carried by App 3. No code creates or alters tables.
- The schema is designed up front for the whole commerce domain (`mb-4-ecom-commerce-schema.md`), with room to grow:
  - every core table (products, variants, orders and so on) carries a metadata column for client-level attributes, such as extra checkout fields;
  - product choices such as colour or size use option tables: an option table and an option value table, with each variant linked to the option values the variant carries.
- A later schema change adds only new tables or nullable columns. The change is applied by hand to App 3 and to every client instance from a migration plan written before release ([[[default:specifications-global/spec-deployment|Deployment]]](C:/dev/project-library-global/specifications-global/spec-deployment.md)).
- Lifecycles (cart, order, payment, shipment, return, dispute, credit account) are designed as explicit state machines in `mb-4-ecom-commerce-lifecycles.md`.

## 5. Storage and client exit

- Each client has one or two Cloudflare R2 buckets, set up at onboarding, in a Cloudflare account dedicated to that client. The credentials of the buckets are in the client's Vault.
- Raw webhook payloads (payments, shipping), execution logs and backups go to the client's bucket, under keys by type and date. Data Tables keep an index row for each item; index rows past the retention period are pruned.
- When the client leaves Mybizz, all of the client's data is written to the client's bucket, and the Cloudflare account holding the bucket is transferred to the client ([[[default:adr-global/adr-client-r2-storage-and-exit-handover|Client R2 Storage and Exit Handover]]](C:/dev/project-library-global/adr-global/adr-client-r2-storage-and-exit-handover.md)).

## 6. Management and health

- Each client instance pushes a heartbeat of the instance's health and resource use (load, errors, order volume, storage) to App 5 on a schedule. App 5 never polls the instances, so App 5 does not slow down as the number of clients grows.
- App 5 raises a container to 2 GiB when the container's memory use passes 80%, and messages the developer. A client instance that stays near the 2 GiB limit or the database connection limit is a candidate for Mybizz Shop Enterprise.

## 7. Resilience

- **Webhooks** from providers (payments, shipping, tax, disputes) return 200 at once, are processed in a background task, and are idempotent by event id, so a provider's retry never charges or ships an order twice.
- **Outbound calls** to providers retry with backoff when rate-limited. Rate limits apply per client instance.
- **Background tasks** (stock alerts, product feeds, reports, imports) run inside the client's container and share the container's memory and database connections with the shop, so heavy tasks run in the quiet hours.
- **Traffic spikes.** Product images and digital files are served from the client's R2 bucket, outside the container. A sudden spike of shoppers (for example a flash sale) queues on the container's 3 database connections, so the shop responds slowly at the peak. A client who plans large sales events moves to Mybizz Shop Enterprise before the event.

## 8. Anvil's limits, accepted

- No push connections to the browser: near-real-time updates use polling.
- Each standard client instance shares an application server with up to 11 other client instances, with the memory and database connection limits of section 3.
- The application is bound to Anvil: the forms and the server-call layer would be rewritten to move elsewhere. Anvil is chosen for Python throughout and the dependency model that makes the five-app structure possible; the Anvil App Server runs each client instance on Mybizz's own servers.

## 9. Product directions

These are candidate features that use the five-app structure and Python to set `mb-4-ecom` apart. Which of these features enter the application is decided at PDLF Steps 12 and 14.

- **Company buying teams:** several people buying on one business customer's account, with spending limits and approvals; customer-specific catalogues.
- **Built-in analytics:** demand forecasting, reorder alerts and product recommendations, computed in each client's own instance with Python, so customer data never leaves the client's instance.
- **No-code shop configuration:** the client arranges home-page sections, colours, navigation and features, and adds custom checkout fields stored in the metadata columns, all through settings.
- **Multi-channel selling:** stock and orders synchronised with external marketplaces (for example Amazon, eBay and Takealot) by background tasks.

## 10. What is particular to mb-4-ecom

Everything in sections 3 to 8 applies the common aspects, except Mybizz Shop Enterprise, which is particular to `mb-4-ecom`. What is particular to `mb-4-ecom` is the shop: the commerce domain mapped to Anvil (section 2), the schema, the lifecycles and the features of the three shop documents, and the product directions (section 9).
