---
title: Installments and recurring commitments
linkTitle: Installments and recurring
weight: 50
description: >
  Recording a purchase split into installments on a card, and the recurring commitments screen, which does not exist yet.
---

Two kinds of money movement are spread over time. An **installment purchase** is
one purchase split into several charges on a credit card. A **recurring
commitment** is a rule that creates transactions on a schedule. The rules for both,
including how an installment total is split and what a recurring rule is, are in
[Concepts: Installments]({{< relref "/docs/concepts#installments" >}}) and
[Concepts: Recurring commitments]({{< relref "/docs/concepts#recurring-commitments" >}}).

| Screen | Address |
| --- | --- |
| Record an installment purchase | `/installments/new` |
| Recurring commitments | none: not built |

## UC-22: Record an Installment Purchase

**Screen:** Record an installment purchase, `/installments/new`. In this version a
plan is charged to a **credit card**, not to an account.

1. Enter the **Total purchase**. The currency beside it is the chosen card's.
2. Enter the **Number of installments**. The field says "At least 2".
3. Choose the **Card** and a **Category**. The date button picks the purchase date
   and starts at today. **Counterparty (optional)** can name the merchant.
4. Press **Record**.
5. The screen shows the plan exactly as the API generated it: the **Total
   purchase**, "Split into N charges on" the purchase date, and under
   **Installments** one row per charge, numbered, with its date and amount.
   **Done** goes to the Credit cards screen.

Notable alternatives:

- **Checks in the form.** "That total could not be read. Enter a number.", "The
  total must be greater than zero." and "An installment plan needs at least 2
  charges. A single charge is just a transaction — record it as one instead." Also
  "Choose the card this was purchased on." and "Choose a category."
- **Uneven amounts.** When the total does not divide evenly, one installment carries
  the remainder, and the screen explains: "These charges are not all equal. The
  total does not divide evenly, so one installment carries the remainder. These are
  the amounts the instance generated, shown as it generated them." The interface
  never recomputes or "corrects" them. The API puts the remainder on the first
  installment; for a worked example, see the installments section of
  [Concepts]({{< relref "/docs/concepts#installments" >}}).
- **No card or no category.** The screen says "Set up a credit card first" (or a
  category, or both), "An installment plan is charged to a card and needs a
  category. Create …, then come back.", with **Go to cards** and **Go to
  categories** buttons.
- **The API refuses the plan.** Its reason is shown.
- **Offline.** The whole screen is replaced by "Not available offline" and the
  core's reason: splitting a purchase into installments assigned to billing cycles
  is not implemented natively. See
  [Offline mode]({{< relref "/docs/user-guide/offline-mode" >}}).

## UC-23: Manage Recurring Commitments

**Not built.** This use case is open as issue
[#24](https://github.com/artur-rios/fortuna-ui/issues/24). There is no screen and no
route for it in the application's route table.

What that means for you today:

- You cannot define, change, list or delete a recurring commitment in the
  interface. The API supports recurring transactions; the interface does not yet
  expose them.
- A future-dated movement cannot be recorded as a transaction either. The record
  form refuses a date more than one day ahead and tells you that a future movement
  is "a recurring commitment or a projection, not a transaction that has happened".
- The API builds its projections from recurring commitments, among other things;
  the Projections screen shows the result, not the rules. See
  [Projections]({{< relref "/docs/user-guide/projections" >}}).

The specification describes the intended behavior: a list of rules, each shown as a
rule with its schedule; a form for an amount, a direction, an account, a category, a
frequency, a start date and an optional end date; and statements that editing a rule
affects future occurrences only and that materialized ones keep what they recorded.
None of that can be used yet.

## Sources

This page is written from the interface's
[Use Case Specification](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/requirements/Use%20Case%20Specification%20Document.md)
(UC-22, UC-23), checked against the code in
[`lib/features/transactions`](https://github.com/artur-rios/fortuna-ui/tree/develop/lib/features/transactions)
and the [route table](https://github.com/artur-rios/fortuna-ui/blob/develop/lib/app/routes.dart).
