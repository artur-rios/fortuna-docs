---
title: Accounts, cards and investments
linkTitle: Holdings
weight: 30
description: >
  Managing financial accounts, credit cards and their statements, and investments with their movements and valuations.
---

A holding is anything that has value and a history: a financial account, a credit
card or an investment. This page covers the screens for creating and reading
them. The rules behind them, including that a holding has one currency fixed for
life and that a balance is always derived, never stored, are in
[Concepts]({{< relref "/docs/concepts#holdings" >}}).

Every figure on these screens comes from the API. The interface never works out a
balance, a used limit or a position itself
([UI-BR-02](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md)),
and when the API refuses something, the screen shows the API's reason. Each
screen below is for an account owner. Creating and editing happen in a small dialog
on the same screen.

| Screen | Address |
| --- | --- |
| Accounts | `/accounts` |
| Credit cards | `/credit-cards` |
| Statements (one card's billing cycles) | `/credit-cards/:id/statements` |
| Statement | `/credit-cards/:id/statements/:statementId` |
| Investments | `/investments` |
| Investment | `/investments/:id` |

## UC-14: Manage Financial Accounts

**Screen:** Accounts, `/accounts`.

1. The screen lists your accounts. Each row shows the name, then the type, the
   institution (if any) and the currency, with the balance the API reports on the
   right.
2. Press **New account**. The dialog "New account" asks for a **Name**, an
   **Institution (optional)**, a **Type** (Checking, Savings, Cash or Other), a
   **Currency** from the list the API supports, and an **Opening balance**. The
   opening balance may be left empty (it is sent as 0) or be negative.
3. Press **Create**. The account appears in the list.
4. To change an account, tap its row. The dialog "Edit account" lets you change the
   name, institution and type. Press **Save**.
5. **Delete**, in the same dialog, removes the account.

Notable alternatives:

- **Checks in the form.** "Give the account a name.", "Choose the currency this
  account is held in." and "That opening balance could not be read. Enter a
  number." Anything else is the API's to decide, for example a duplicate name,
  and its refusal is shown in the dialog.
- **The currency cannot be edited.** In the edit dialog the field is disabled with
  the reason: "Fixed once the account exists. Every figure in it was recorded in
  this currency; changing it would relabel that history rather than convert it."
- **Deleting is refused** while live records still reference the account; the API's
  reason is shown in the dialog. Deleting asks no extra confirmation; a deleted
  account can be restored from the Deleted records screen (see
  [Deleted records and the audit trail]({{< relref "/docs/user-guide/deleted-records-and-audit" >}})).
- **No balance.** If the balance cannot be read, the row shows "No balance" in the
  error colour, with the reason in a tooltip. The interface never fills the gap
  with a computed number.
- **No accounts yet.** The screen says "No accounts yet", "Add the accounts and
  cash you want to track. Everything else in Fortuna hangs off them.", with an
  **Add an account** button. A failed load shows the reason and **Try again**.
- **There is no account detail or history screen.** The row opens the edit dialog.
  To see an account's transactions, filter the
  [Spreadsheet]({{< relref "/docs/user-guide/spreadsheet" >}}) by that account.

## UC-15: Manage Credit Cards

**Screen:** Credit cards, `/credit-cards`.

1. The screen lists your cards as cards. Each shows the name, the last four digits
   as "•••• 1234" if you gave them, the issuer, and three figures from the API:
   **Used**, **Available** and **Limit**. A line "Closes on day N · due on day M"
   follows.
2. Press **New card**. The dialog "New card" asks for a **Name**, an **Issuer
   (optional)**, **Last four digits (optional)**, a **Currency**, a **Credit
   limit**, a **Closing day** and a **Due day**.
3. The form checks that the limit is greater than zero and that both days are
   between 1 and 31: "The credit limit must be greater than zero." and "The closing
   day must be 1 to 31." (the due day likewise). Whether a closing and due day
   combination makes sense is the API's to answer.
4. Press **Create**. To change a card, tap it for "Edit card" (the currency is not
   offered there), and **Delete** removes it.

Notable alternatives:

- A card over its limit gets its own line with a warning icon: "Over the limit by"
  followed by the amount.
- A duplicate name, or a deletion refused because records still reference the card,
  shows the API's reason in the dialog.
- With no cards, the screen says "No cards yet" and "Add a card to track its
  statements and what you owe.", with an **Add a card** button.
- Each card has a **Statements** button, which is UC-16. In offline mode it is
  disabled with the reason "Not available offline".

## UC-16: Review and Settle a Credit Card Statement

**Screens:** Statements, `/credit-cards/:id/statements`, and Statement,
`/credit-cards/:id/statements/:statementId`. Reached from the **Statements**
button on a card.

1. "Statements" lists the card's billing cycles. Each row shows the period, "Due"
   and the due date, a state chip (**Open**, **Closed** or **Settled**) and the
   amount due. A small clock icon with the tooltip "Contains a late-arriving
   charge" marks a cycle that received one.
2. Tap a cycle to open "Statement". It shows the period, "Closes … · due …", and a
   summary: **Previous balance**, **Purchases**, **Payments received**, **Foreign
   tax**, **Other entries** and **Amount due**. Below, **Charges** lists each
   charge with its date. A charge that arrived late carries a "Late arrival" tag. A
   charge made in another currency shows "Charged as" with the original amount and
   the rate.
3. For an open statement, press **Close this statement**. The dialog "Close this
   statement?" says "Closing fixes what this statement contains. Charges that
   arrive afterwards go to the next cycle, and this cannot be undone." Choose
   **Close it** or **Keep it open**.
4. For a closed statement, press **Settle this statement**. A sheet shows the amount
   due and asks for **Pay from**, one of your financial accounts, and a payment date
   ("Paid on …"). Press **Settle**.
5. The sheet says plainly that the payment is "Recorded as a transfer between your
   accounts, not as an expense — the charges were already counted when they were
   made."

Notable alternatives:

- **Closing too early.** Close is offered on any open statement. Whether the closing
  date has passed is the API's judgment; it refuses if not, and the reason appears
  at the bottom of the screen.
- **A settled statement** shows a "Settled" card: "This statement is paid and its
  composition is fixed. It can no longer be closed or settled."
- **A closed statement is not offered Close, and an open one is not offered
  Settle.** An open statement shows "Still open": "Close this statement before
  settling it. Settling is offered once the composition is fixed."
- **Paying from an account in another currency** is the API's to refuse or convert,
  and its reason appears in the sheet. Nothing is converted locally.
- **No cycles yet.** "No statements yet", "Billing cycles appear here once the card
  has charges."
- **Offline.** Both screens are replaced by "Not available offline" with the core's
  reason, and closing or settling is a separate "Not available offline" line. See
  [Offline mode]({{< relref "/docs/user-guide/offline-mode" >}}).

## UC-17: Manage Investments

**Screens:** Investments, `/investments`, and Investment, `/investments/:id`.

1. "Investments" lists your investments. Each row shows the name, the type and
   institution, and the position the API reports. A row with no valuation says "No
   valuation recorded" under it.
2. Press **New investment**. The dialog asks for an **Instrument**, an
   **Institution (optional)**, a **Type** (Fixed income, Equity, Fund or Other) and
   a **Currency**. The form checks "Name the instrument." and the currency; the
   API decides the rest. Press **Create**.
3. Tap a row to open the investment. It shows the type, currency and institution,
   the **Position**, and the **Latest valuation** with "Recorded for" its date. The
   edit (pencil) icon in the top bar opens the same dialog, where **Delete** is also
   available.
4. The position is "What has been contributed and withdrawn, as the instance
   reports it." The interface never prices an instrument.

Notable alternatives:

- **No valuation recorded.** The page says "No valuation has been recorded for this
  investment. The position above is what was contributed and withdrawn — it is not a
  current value, and this application does not price instruments."
- **No investments.** "No investments yet", "Add an investment to record what you
  put in, take out and value it at.", with **Add an investment**.
- A duplicate name, or a deletion refused because movements still reference it,
  shows the API's reason.

## UC-18: Record an Investment Movement or Valuation

**Screen:** Investment, `/investments/:id`.

1. On the investment, press **Record movement** or **Record valuation**.
2. A movement asks for a **Kind** (Contribution, Withdrawal, Yield or Fee), an
   **Amount** in the investment's currency, a date, and optionally **From account
   (optional)**, which "Links this movement to where the money came from or went." A
   valuation asks for a **Value** and a date.
3. The form checks that the amount or value is greater than zero ("The amount must
   be greater than zero." or "The value must be greater than zero.") and that the date
   is not in the future ("That date is in the future. Record it on or before today.").
   The amount is held as an exact decimal.
4. Press **Record**. The page shows the updated position as the API reports it.

Under **Recorded valuations** the page lists each valuation, newest first, by date
with its value. This history is read-only. Movements are recorded but not listed on
this screen; they show in the position. A valuation for a date that already has one,
or an amount in a different currency, is answered by the API and its reason is shown.

## Sources

This page is written from the interface's
[Use Case Specification](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/requirements/Use%20Case%20Specification%20Document.md)
(UC-14 to UC-18), checked against the code in
[`lib/features/holdings`](https://github.com/artur-rios/fortuna-ui/tree/develop/lib/features/holdings).
