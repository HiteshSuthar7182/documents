# Marketplace Platform — Entity & Feature Breakdown

**Purpose:** Working checklist of every entity and feature, split by who controls it. Add, remove, and annotate freely — this is a starting scaffold, not a final spec.

**Legend:**
- **A** = Main Admin
- **V** = Vendor
- **C** = Customer
- **B** = Branch User / POS operator

---

## 1. Identity, access and organisation

| Entity | A | V | C | B | Notes |
|---|:-:|:-:|:-:|:-:|---|
| Admin users | Full | — | — | — | Platform staff |
| Roles & permissions | Full | Own staff only | — | — | Vendor defines roles within their own scope |
| Vendors | Full CRUD, approve/suspend | Own profile only | — | — | Onboarding + KYC status |
| Vendor staff users | View all | Full CRUD | — | — | |
| Branches | View all | Full CRUD (own) | — | View own | Each branch belongs to one vendor |
| Branch users | View all | Full CRUD | — | Own profile | POS operators |
| Customers | Full | Own customers | Own profile | Create at POS | Decide: shared across vendors or per-vendor? |
| Customer groups | Define global | Define own | — | — | Drives pricing and promotions |
| Activity / audit log | Full | Own scope | — | — | **Required for financial records** |
| API tokens | Full | Own | — | — | |

**Open question:** is a customer global to the marketplace, or owned by a vendor? This affects B2B pricing, privacy, and data ownership. Decide before building.

---

## 2. Catalogue

| Entity | A | V | C | B | Notes |
|---|:-:|:-:|:-:|:-:|---|
| Product types | Define | Use | — | — | Tailor, fabric, readymade, raw, material |
| Products | Full, approve | Full CRUD (own) | View | View | Single table + type discriminator |
| Product variants / styles | View | Full CRUD | View | View | Thobe styles, collar, cuff, etc. |
| Attribute definitions | **Define all** | Use, set values | — | — | Admin-controlled, per product type |
| Attribute groups / sets | Define | Assign | — | — | |
| Categories | Define global tree | Assign, own subtree | Browse | Browse | |
| Brands | Define | Assign | Filter | — | |
| Units of measure | Define | Use | — | — | Metre, yard, piece, kg |
| Product images / media | Moderate | Full CRUD | View | View | |
| Product bundles / kits | View | Full CRUD | View | View | Fabric + tailoring as one sellable item |
| Related / cross-sell | — | Full CRUD | View | — | |
| Product approval queue | Approve/reject | Submit | — | — | Optional — decide if vendors self-publish |

---

## 3. Measurements & tailoring

| Entity | A | V | C | B | Notes |
|---|:-:|:-:|:-:|:-:|---|
| Measurement templates | Define | Define own | — | — | Per garment type |
| Measurement fields | Define | Extend | — | — | Chest, sleeve, length, with validation ranges |
| Customer measurement profiles | View | Full CRUD | Own, view/edit | Create/edit | **Must be versioned** |
| Measurement snapshot on order | — | — | — | — | Copy values onto order item at order time |
| Style options | — | Full CRUD | Select | Select | Collar type, cuff, pocket, embroidery |
| Fabric consumption rules | — | Full CRUD | — | — | Qty of fabric per garment/size |
| Pattern / production status | View | Update | View | View | Ties to order status flow |
| Tailor assignment | View | Assign | — | — | Which tailor is working the order |

---

## 4. Inventory

| Entity | A | V | C | B | Notes |
|---|:-:|:-:|:-:|:-:|---|
| Stock per branch | View all | Full | View availability | View own | Branch-level, not global |
| Stock movements ledger | View all | View own | — | — | **Append-only, never mutate a qty column** |
| Stock transfers between branches | View | Approve/create | — | Request | |
| Stock adjustments | View | Full | — | Request | Damage, loss, correction — with reason code |
| Reservations | View | View | — | — | Fabric reserved against an open order |
| Low-stock alerts | Configure | Configure | — | Receive | |
| Batch / lot tracking | — | Optional | — | — | Relevant for fabric rolls |
| Stock take / physical count | View | Full | — | Perform | |

---

## 5. Orders

| Entity | A | V | C | B | Notes |
|---|:-:|:-:|:-:|:-:|---|
| Orders | View all, intervene | Full (own) | Own, place | Create at POS | |
| Order items | View | Full | View | Create | Carries measurement snapshot |
| Order status workflow | Define states | Transition | View | Transition | Use the documented state machine |
| Split orders by vendor | Automatic | View own portion | See as one | — | Multi-vendor cart handling |
| Order notes / internal comments | Full | Own | Customer-visible only | Own | |
| Order history / audit trail | Full | Own | Own | Own | |
| Cancellations | Full | Request/approve | Request | Request | |
| Returns (RMA) | Full | Process | Request | Process | |
| Exchanges | Full | Process | Request | Process | |
| Alterations / rework | View | Full | Request | Create | Tailoring-specific, not the same as a return |
| Order documents | View | Print | Download | Print | Job card, pattern sheet, delivery note |

---

## 6. Payments & money

| Entity | A | V | C | B | Notes |
|---|:-:|:-:|:-:|:-:|---|
| Invoices | View all | Full (own) | Own | Create | |
| Payments | View all | View own | Make | Record | Separate resource, not a field on invoice |
| Partial payments | View | View | Pay balance | Record | Many payments per invoice |
| Payment methods | Define global | **Enable/configure own** | Select | Select | Vendor-level gateway config |
| Refunds / credit notes | Full | Process | Request | Process | |
| Cashbox per branch | View all | Full | — | Open/close | |
| Cashbox sessions | View | View | — | Open/close | Opening float, closing count, variance |
| Bank accounts per branch | View all | Full | — | — | |
| Cash-to-bank transfers | View | Approve | — | Initiate | |
| **Financial ledger** | View all | View own | — | — | **Append-only. Balances derived, never stored mutable** |
| Commission rules | **Define** | View | — | — | Flat, percentage, per-category, tiered |
| Vendor settlements / payouts | **Full** | View own | — | — | Payout cycle, held amounts, adjustments |
| Vendor wallet / balance | View | View | — | — | Derived from ledger |
| Taxes | Define rules | Apply | Pay | Apply | Rates, classes, zones, inclusive vs exclusive |
| Currency | Define | — | — | — | Single or multi — decide early |

---

## 7. Promotions & pricing

| Entity | A | V | C | B | Notes |
|---|:-:|:-:|:-:|:-:|---|
| Price lists | Define | Define own | — | — | B2B vs B2C separation |
| Tier / quantity pricing | View | Full | See applicable | Apply | B2B bulk |
| Customer-group pricing | Define | Define | — | — | |
| Coupons | Platform-wide | Own store | Apply | Apply | Who funds the discount? |
| Cart promotion rules | Platform-wide | Own | — | — | BOGO, spend thresholds |
| Catalogue price rules | Platform-wide | Own | — | — | |
| Gift cards / store credit | Full | Optional | Use | Redeem | |
| Loyalty points | Define | Participate | Earn/redeem | Apply | |
| Campaign scheduling | Full | Own | — | — | Start/end dates |

**Open question:** when an admin-funded coupon applies to a vendor's product, who absorbs the discount? This must be settled in the commission logic.

---

## 8. POS specific

| Feature | Notes |
|---|---|
| Offline-first operation | IndexedDB cache, queued orders, client-generated UUIDs — **design from day one** |
| Sync conflict resolution | Rules for stock conflicts when two branches sell the last item offline |
| Fast product lookup | Barcode scan, SKU, name search |
| Walk-in customer | Order without full registration |
| Split payment tender | Cash + card + wallet on one sale |
| Cash drawer session | Open float, sale log, close and count, variance report |
| Receipt printing | Thermal printer, configurable template |
| Hold / resume sale | Park an order and return to it |
| Returns at POS | With and without a receipt |
| Shift handover | Operator change mid-session |
| Barcode / label printing | For fabric rolls and readymade items |

---

## 9. Shipping & fulfilment

| Entity | A | V | C | B | Notes |
|---|:-:|:-:|:-:|:-:|---|
| Shipping methods | Define global | **Configure own** | Select | Select | |
| Shipping zones / rates | Define | Own rates | — | — | Weight, price, distance based |
| Carriers / integrations | Configure | Enable | Track | — | |
| Shipments | View | Create | Track | Create | |
| Pickup from branch | Enable | Configure | Select | Fulfil | |
| Delivery slots | — | Configure | Select | — | |
| Packing slips / labels | — | Print | — | Print | |

---

## 10. Content & storefront

| Entity | A | V | C | Notes |
|---|:-:|:-:|:-:|---|
| CMS pages | Full | Own store page | View | |
| Banners / sliders | Platform | Own store | View | |
| Vendor storefront profile | Moderate | Full | View | Logo, description, policies |
| Reviews & ratings | Moderate | Respond | Write | Product and vendor level |
| Wishlist | — | — | Own | |
| Blog | Full | — | View | Optional |
| SEO metadata | Full | Own products | — | |

---

## 11. Communications

| Entity | A | V | C | Notes |
|---|:-:|:-:|:-:|---|
| Email templates | Define | Override own | Receive | |
| SMS templates | Define | Override | Receive | |
| WhatsApp templates | Define | Use | Receive | Approved templates only |
| Notification rules | Define | Configure | Preferences | Which event triggers which channel |
| In-app notifications | Send | Send/receive | Receive | |
| Support tickets | Full | Own | Raise | Optional |
| Vendor ↔ customer messaging | Moderate | Full | Full | Optional |

---

## 12. Reporting

| Report | A | V | B | Notes |
|---|:-:|:-:|:-:|---|
| Sales by period | All vendors | Own | Own branch | |
| Sales by product / category | All | Own | Own | |
| Sales by branch | All | Own | Own | |
| Vendor performance | All | Own | — | |
| Commission earned | Full | Own | — | |
| Outstanding / receivables | All | Own | Own | Ties to partial payments |
| Cashbox variance | All | Own | Own | Expected vs counted |
| Stock valuation | All | Own | Own | |
| Fabric consumption vs estimate | All | Own | — | Tailoring-specific |
| Production throughput | All | Own | — | Orders per stage per day |
| Tax report | Full | Own | — | |
| Customer lifetime value | All | Own | — | |

---

## 13. System & configuration

| Entity | A | V | Notes |
|---|:-:|:-:|---|
| Global settings | Full | — | |
| Vendor-level settings | View | Full | Payment, shipping, notification, business hours |
| Branch-level settings | View | Full | |
| Currencies & exchange rates | Full | — | |
| Languages / translations | Full | Contribute | Arabic + English — plan RTL from the start |
| Tax configuration | Full | — | |
| Webhooks | Full | Own | |
| **ERP / external sync** | Full | View status | Outbox pattern, idempotency keys, replay UI |
| Import / export | Full | Own data | Bulk product, customer, stock |
| Backup & restore | Full | — | |
| Feature flags | Full | — | |

---

## Cross-cutting requirements

These apply everywhere and are cheapest to build in at the start:

1. **Vendor scoping enforced at ORM level** — a global scope on every vendor-owned model, not a manual `where` clause per query. One omission leaks data between vendors.
2. **Append-only ledger for all money** — payments, cashbox, settlements, stock. Balances derived, never stored mutable.
3. **Idempotency keys on every write endpoint** — POS retries, network failures, and duplicate submissions are certain.
4. **Soft delete on all financial and order records** — never physically delete.
5. **Audit trail on anything involving money or stock** — who, what, when, previous value.
6. **Exact decimal arithmetic** — never floats for money or fabric quantities.
7. **RTL and Arabic support** — retrofitting this is expensive.
8. **Versioned API** from the first release.

---

## Decisions to make before building

| # | Decision | Why it matters |
|---|---|---|
| 1 | Customers global to the marketplace, or owned per vendor? | Affects pricing, privacy, data ownership, GDPR-style requests |
| 2 | Single currency or multi? | Touches every money table |
| 3 | Who funds admin-issued coupons on vendor products? | Determines commission calculation |
| 4 | Do vendors self-publish products, or is approval required? | Changes the catalogue workflow entirely |
| 5 | Is POS offline-first, or online-only? | Cannot be retrofitted cheaply |
| 6 | Single warehouse per vendor, or true multi-branch stock? | Changes the inventory model fundamentally |
| 7 | Commission model — flat, percentage, tiered, per-category? | Shapes the settlement engine |
| 8 | Does B2B need quotes, credit limits, and purchase orders? | A significant additional module if yes |
