# Order Status Flow

**Version:** 1.2
**Scope:** Order lifecycle from invoice creation through delivery, return, or closure.
**Changes since 1.1:** Return and Closed entry rules defined. BR-1 rescoped to item level. Measurement Attention loop clarified as status-only. One open question remains (Q1).

---

## 1. Main states

| Main state | Sub-states | Description |
|---|---|---|
| **Processing** | New, Printed, Pattern Done, Measurement Attention | Order placed and invoice created. All pre-shipment operations happen here. |
| **Complete** | Shipped / Partially Shipped, Received In Branch, Delivered / Partially Delivered | Order has left processing. One-way — the order header never returns to Processing. |
| **Return** | — | Customer returned the order or an item. Reachable only after shipment. |
| **Closed** | — | Terminal state. Sub-status is either **Cancelled** or **Refunded**. |

> **Measurement Attention is not a sequential step.** It is an exception branch reachable from Printed or Pattern Done that always loops back to New. See section 3.

---

## 2. Happy path

```
New → Printed → Pattern Done → Shipped / Partially Shipped
    → Received In Branch → Delivered / Partially Delivered
```

| # | State | Trigger | Actor |
|---|---|---|---|
| 1 | **New** | Order placed successfully, invoice generated. | System |
| 2 | **Printed** | Order document / pattern printed. | Vendor |
| 3 | **Pattern Done** | Pattern-making completed. | Vendor |
| 4 | **Shipped / Partially Shipped** | Dispatched from branch. Partial when only some items are ready. | Vendor |
| 5 | **Received In Branch** | Branch user collects the consignment via **Home → Receive Order**. | Branch user |
| 6 | **Delivered / Partially Delivered** | Final handover to customer. | Branch user |

**Received In Branch is mandatory.** An order cannot move from Shipped directly to Delivered. The collection step must be recorded first, and only after it can the order proceed to Delivered or Return.

---

## 3. Exception flow — Measurement Attention

**Entry condition:** the order is in **Printed** or **Pattern Done**.

1. Vendor raises a *Change Measurement* request.
2. Order status moves to **Measurement Attention**.
3. Branch user updates the measurement.
4. On save, the order status is set back to **New** and re-enters the happy path.

**Constraints:**

- The loop is a **status change only**. No new invoice is generated and no existing invoice is regenerated.
- **No retry limit.** An order may enter Measurement Attention any number of times.
- Cannot be triggered from **New** — there is nothing to correct yet.
- Cannot be triggered once the order has reached any **Complete** sub-state.
- **Fabric Consumption** is tracked internally and does **not** trigger any status change.

---

## 4. Partial fulfilment

Status is tracked **per item**; the order header shows the aggregate.

Example — a 2-item order where item A is ready first:

| Step | Item A | Item B | Order header |
|---|---|---|---|
| 1 | Shipped | Pattern Done | **Partially Shipped** |
| 2 | Received In Branch | Pattern Done | **Partially Shipped** |
| 3 | Delivered | Pattern Done | **Partially Delivered** |
| 4 | Delivered | Shipped | **Partially Delivered** |
| 5 | Delivered | Received In Branch | **Partially Delivered** |
| 6 | Delivered | Delivered | **Delivered** |

The header reads **Partially** while any item is still outstanding, and resolves to the full status only when every item reaches it. Item B continues through its own Processing steps while the header already reads Complete — this is expected and does not violate BR-1.

---

## 5. Business rules

**BR-1 — No backward movement, scoped per item.**
Once an *item* reaches any Complete sub-state, that item can never move back to Processing — including for measurement issues. The order header likewise never reverts from Complete to Processing. Items not yet shipped are unaffected and continue through Processing normally.

**BR-2 — Return requires prior shipment.**
A Return can only be accepted once the order has shipped. It is a forward transition out of Complete into a separate branch, not backward movement, and does not violate BR-1.

**BR-3 — Closed is reachable from any state.**
Cancelling an order or refunding it moves it to Closed with sub-status **Cancelled** or **Refunded** respectively. This applies from both Processing and Complete.

**BR-4 — Fabric Consumption is status-neutral.**
Recorded for reporting only; never changes order status.

---

## 6. Transition matrix

| From | Allowed next states |
|---|---|
| New | Printed, Closed |
| Printed | Pattern Done, Measurement Attention, Closed |
| Pattern Done | Shipped / Partially Shipped, Measurement Attention, Closed |
| Measurement Attention | New |
| Shipped / Partially Shipped | Received In Branch, Closed |
| Received In Branch | Delivered / Partially Delivered, Return, Closed |
| Delivered / Partially Delivered | Return, Closed |
| Return | *See Q1* |
| Closed | — (terminal) |

---

## 7. Open question

| # | Question | Why it matters |
|---|---|---|
| Q1 | Is **Return** terminal, or does it move to **Closed / Refunded** once the refund is processed? | BR-3 says a refund sends an order to Closed, and returns normally end in a refund — which suggests Return → Closed. But if Return is meant to be terminal, the refund path needs its own rule. The transition matrix cannot be finalised until this is settled. |

---

## 8. Terminology

Use these exact terms throughout the codebase and UI:

- **Branch user** — the eSeller app user (not "eSeller user", "branch/eSeller user", or "branch staff").
- **Vendor** — the tailoring vendor performing print and pattern work.
- **Main state** — Processing, Complete, Return, Closed.
- **Sub-state** — a status within a main state.
- **Sub-status** — used only for Closed: Cancelled or Refunded.