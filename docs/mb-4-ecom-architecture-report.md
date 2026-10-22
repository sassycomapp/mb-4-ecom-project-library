---
title: "mb-4-ecom-architecture-report"
updated: "2026-10-22"
doc-id: "mb-c54ed60b"
---
# mb-4-ecom — Architecture Report

## What this document is

This report states the architecture decisions and facts that govern `mb-4-ecom`, Mybizz's e-commerce application. It is the context `mb-4-ecom` brings into PDLF. Everything `mb-4-ecom` shares with other Mybizz applications is governed by [[[default:policy-global/pol-common-aspects-for-mybizz-anvil-applications|Common Aspects for Mybizz Anvil Applications]]](C:/dev/project-library-global/policy-global/pol-common-aspects-for-mybizz-anvil-applications.md); this report states only how those aspects apply to e-commerce and what is particular to it.

## 1. What mb-4-ecom is

`mb-4-ecom` is an e-commerce platform that Mybizz provides to businesses that sell products. Each client business gets its own online store and back office: catalogue, cart, checkout, orders, fulfilment, customers, promotions and payments. It is built on Anvil.works, in Python, by PDLF.

It shares its architecture and infrastructure with `mb-3-cs` (Mybizz Consulting and Services). The two differ in their business domain: `mb-3-cs` centres on scheduling and bookings, `mb-4-ecom` on catalogue, cart and orders. Whatever can be common between them is kept common, in both directions ([[[default:adr-global/adr-common-architecture-across-mybizz-applications|Common Architecture Across Mybizz Applications]]](C:/dev/project-library-global/adr-global/adr-common-architecture-across-mybizz-applications.md)).

## 2. The reference platform: Medusa

`mb-4-ecom` takes Medusa (`medusajs.com`) as the model for its business domain and workflows.

**What Medusa is:**

- An open-source, headless commerce platform: its storefronts and admin are separate clients of its API.
- Written in TypeScript on Node.js, with an Express-based server. It uses PostgreSQL for its data and Redis for its cache, event bus, locking and workflow engine.
- Built in four layers: API routes receive requests, workflows hold the business logic, modules manage each commerce domain, and the database stores the data.
- Its commerce domains are its commerce modules: product, pricing, cart, order, payment, fulfilment, inventory, stock location, tax, region, currency, customer, promotion, sales channel, store, store credit, user, auth, API key, settings and translation.
- Its core is under the MIT licence, which allows it to be studied, modified and used commercially, keeping the original copyright notice. Its enterprise features (role-based access, single sign-on and some audit features) are under a separate commercial licence and are not used; Mybizz builds its own.

**How Medusa is used:**

1. Medusa is analysed with REA (`morluto/rea`), a reverse-engineering tool, outside the Mybizz process. Its output is a description of Medusa's domain, data models, workflows and API.
2. That description is the context material `mb-4-ecom` brings into PDLF at Step 03.
3. PDLF adapts it to Python for Anvil, under the common aspects. Only the domain and the workflows are taken; Medusa's technology is not carried over.

**How Medusa's layers map to Anvil:**

| Medusa | In `mb-4-ecom` |
|---|---|
| API routes | Anvil server functions called by the app's forms; HTTP endpoints for webhooks and the management endpoints |
| Workflows | Server functions built on pure-logic modules, so the business rules are tested on their own |
| Commerce modules and data models | Anvil Data Tables with model classes; behaviour that belongs to one table lives on its model class |
| Event bus and subscribers | Anvil background tasks |
| Scheduled jobs | Anvil scheduled tasks |
| Locking | Anvil Data Tables transactions |
| File storage | The client's own R2 buckets |
| Notifications | Anvil email and Brevo |
| Analytics | The application's own metrics tables, reported to App 5 |
| Admin dashboard | The application's admin forms, in Material 3 |

## 3. The five-app structure for e-commerce

`mb-4-ecom` uses the five-app structure exactly as every Mybizz application does ([[[default:specifications-global/spec-five-app-architecture-model|Five-App Architecture Model]]](C:/dev/project-library-global/specifications-global/spec-five-app-architecture-model.md)).

- **Never multi-tenant.** Each client store is its own Anvil app with its own database, on its own Anvil account held by Mybizz and named after the client's business.
- **Load stays in each client instance.** A client's transactions run in its own instance. App 2 is a code dependency and carries no client traffic, so 100 clients running 100 transactions each place 100 transactions on each instance and none on App 2.
- **Isolation.** One client's traffic, fault or heavy query cannot slow down or expose another client.
- **Plan tiers.** Every client starts on Anvil's Hobby tier. A client whose volume grows has its own tier raised, or server resources added, on its own account, with no change to App 2.

## 4. Data design

- The schema is designed in App 1, published through App 2 and carried by App 3. No code creates or alters tables.
- The schema is designed up front for the whole commerce domain, with room to grow:
  - every core table (products, variants, orders, customers and so on) carries a metadata column for client-level attributes, such as extra checkout fields;
  - client-defined attributes (for example colour or size) use attribute tables: an attribute table and a value table linking each product to its attribute values.
- A later schema change adds only new tables or nullable columns. It is applied by hand to App 3 and to every client instance from a migration plan written before release ([[[default:specifications-global/spec-deployment|Deployment]]](C:/dev/project-library-global/specifications-global/spec-deployment.md)).
- Lifecycles (cart, order, payment, fulfilment, return) are designed as explicit state machines and proven before the design is locked.

## 5. Storage and client exit

- Each client has one or two Cloudflare R2 buckets of its own, set up at onboarding, in a Cloudflare account dedicated to that client. Their credentials are in the client's Vault.
- Raw webhook payloads (payments, shipping), execution logs and backups go to the client's bucket, under keys by type and date. Data Tables keep an index row for each item; index rows past the retention period are pruned.
- When the client exits Mybizz, all its data is written to its bucket and the account holding it is transferred to the client, who leaves in possession of it ([[[default:adr-global/adr-client-r2-storage-and-exit-handover|Client R2 Storage and Exit Handover]]](C:/dev/project-library-global/adr-global/adr-client-r2-storage-and-exit-handover.md)).

## 6. Management and health

- Each client instance pushes a heartbeat of its health and resource use (load, errors, order volume, storage) to App 5 on a schedule. App 5 never polls the instances, so App 5 does not slow down as the number of clients grows.
- App 5 flags any client approaching the limits of its plan tier, so its tier is raised before its customers notice.

## 7. Resilience

- **Webhooks** (payments, shipping) return 200 at once, process in a background task, and are idempotent by transaction id, so a gateway's retry never charges or fulfils an order twice.
- **Outbound calls** to gateways, shipping services and marketplaces retry with backoff when rate-limited. Rate limits apply per client instance.
- **Background tasks** (inventory syncs, reports, imports) stay within the background-task limits of the client's plan tier.
- **Traffic spikes.** An instance that receives a sudden spike (for example a flash sale) can respond slowly at first. A client that plans a high-traffic event has its tier raised before the event.

## 8. Anvil's limits, accepted

- No push connections to the browser: near-real-time updates use polling.
- Background-task concurrency is limited per plan tier.
- The application is bound to Anvil: its forms and server-call layer would be rewritten to move elsewhere. Anvil is chosen for its managed hosting, its Python throughout and its dependency model, which makes the five-app structure possible.

## 9. Product directions

These are candidate features that use the five-app structure and Python to set `mb-4-ecom` apart. Which of them enter the application is decided at PDLF Steps 12 and 14.

- **B2B commerce:** company accounts with several buyers, spending limits and approvals; customer-specific catalogues and price lists; payment terms (30 or 60 days) and purchase-order checkout.
- **Built-in analytics:** demand forecasting and reorder alerts, and product recommendations, computed in each client's own instance with Python, so customer data never leaves it.
- **No-code store configuration:** store owners arrange home-page sections, colours, navigation and features, and add custom checkout fields stored in the metadata columns, all through settings.
- **Multi-channel selling:** inventory and orders synchronised with external marketplaces (for example Amazon, eBay and Takealot) by background tasks.

## 10. What is particular to mb-4-ecom

Everything in sections 3 to 8 applies the common aspects. What is particular to `mb-4-ecom` is its commerce domain (section 2's modules, mapped to Anvil), its order and fulfilment lifecycles, its schema, and its product directions (section 9).
