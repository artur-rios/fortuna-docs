---
title: Transactions and transfers
linkTitle: Transactions
weight: 40
description: >
  Recording, correcting and deleting transactions, moving money between your accounts, reconciling, and the attachments screen that does not exist yet.
---

A transaction is an expense or an earning on one of your accounts or cards, and it
is the most frequent thing you do in Fortuna. A transfer moves money between two of
your own accounts and counts as neither. The rules, including that an amount is
always positive and its direction carries the sign, are in
[Concepts]({{< relref "/docs/concepts#money-movement" >}}).

In every form on this page the amount is read as an exact decimal in the locale you
chose in Settings, so `1.234,50` and `1,234.50` are stored identically, and an
action is only reported as done once the API has accepted it
([UI-BR-04](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md)).
When the API refuses, you see its reason and the form keeps what you entered.

| Screen | Address |
| --- | --- |
| Record a transaction | `/transactions/new` |
| Transaction (one transaction) | `/transactions/:id` |
| Record a transfer | `/transfers/new` |

The list of transactions is the [Spreadsheet]({{< relref "/docs/user-guide/spreadsheet" >}}),
which has a **Record** button and opens a transaction when you tap its row.

## UC-19: Record a Transaction

**Screen:** Record a transaction, `/transactions/new`.

1. Choose **Expense** or **Earning** at the top.
2. Enter the **Amount**. The currency shown beside it is the chosen account's or
   card's.
3. The date button shows today. Tap it to pick another day.
4. Choose the **Account or card**. Cards are marked "(card)", and each entry shows
   its currency.
5. Choose a **Category**.
6. Optionally fill in **Description (optional)**, **Counterparty (optional)** and
   **Tags (optional)**, where tags are typed as names, "Separated by commas".
7. Press **Record**. When the API accepts it, the screen becomes "Recorded", showing
   the amount, the direction and the category, with **Record another** (which clears
   the form) and **Done**. **Done** goes to `/`, which is a placeholder; see
   [the guide's start]({{< relref "/docs/user-guide" >}}).

Notable alternatives:

- **Checks in the form.** "That amount could not be read. Enter a number.", "The
  amount must be greater than zero.", "Choose a category." and "Choose the account
  or card this is on." A date more than one day ahead is refused, in the picker as
  well: "That date is too far ahead. A movement in the future is a recurring
  commitment or a projection, not a transaction that has happened."
- **Nothing to record against.** If you have no account or card, or no category, the
  screen says "Set up an account or a credit card first" (or a category, or both)
  and explains "A transaction needs somewhere to sit and something to be called.
  Create …, then come back.", with **Go to accounts** and **Go to categories**
  buttons.
- **A rule the form does not check** is the API's to refuse, and its reason is shown
  under the form.
- **A failure to reach the instance** does not report the transaction as recorded,
  and the form keeps your entry so you can retry.

## UC-20: Update or Delete a Transaction

**Screen:** Transaction, `/transactions/:id`. Reached by tapping a row in the
spreadsheet, a transaction in a chart drill-down, or "Open the transaction" on an
imported record.

1. The screen shows the amount, the direction (or "Transfer"), the date, and any
   category, account or card, description, counterparty and tags.
2. A **Source** card says where it came from: "Entered by hand", "From a connected
   institution", "Imported from a spreadsheet" or "Imported from a statement file".
3. If it was imported, an "The imported record" card shows "Amount as imported" and
   "Date as imported" exactly as they arrived. It says "This is what was imported,
   exactly as it arrived. It cannot be edited: it is the evidence the import is
   reconciled against, and a record that can be changed is evidence of nothing." If
   you corrected a value after import, it adds "A value above was corrected after
   import. The original is kept here."
4. To correct the transaction, press the pencil icon (tooltip "Edit") in the top
   bar. The dialog "Edit transaction" has the direction, amount, date, account or
   card, category, description, counterparty and tags, with **Save**.
5. To delete it, press **Delete** in the same dialog. A second dialog asks "Delete
   this transaction?" and says "It will disappear from every view except deleted
   records, where it can be restored." Choose **Delete** or **Keep it**.

Notable alternatives:

- The same checks as recording apply to a correction.
- The API may refuse a change, for example on a transaction in a settled statement
  or one already reconciled; its reason is shown in the dialog and your edits stay.
- A deleted transaction shows a card, "This transaction is deleted. It no longer
  appears in any view except deleted records, where it can be restored.", and the
  edit icon is not offered. Restoring is in
  [Deleted records and the audit trail]({{< relref "/docs/user-guide/deleted-records-and-audit" >}}).

## UC-21: Record a Transfer

**Screen:** Record a transfer, `/transfers/new`. Not available offline.

1. Choose the origin in **From** and the destination in **To**. Each entry shows the
   account's currency.
2. Enter the **Amount**, which is in the origin's currency, and pick a date.
3. Press **Transfer**.
4. The screen becomes "Transferred". It says "Recorded as a transfer between your
   own accounts. It counts as neither an earning nor an expense, so it does not
   appear in either total." If the accounts hold different currencies and the API
   converted, the screen adds "Converted by the instance at" and the rate.
   **Done** closes it.

Notable alternatives:

- Choosing the same account twice is rejected in the form: "Choose two different
  accounts. Money cannot be transferred to the account it came from." An amount
  that is not greater than zero is rejected too.
- With fewer than two accounts the screen says "A transfer needs two accounts", and
  that you have none or one, with a **Go to accounts** button.
- The API applies both sides of a transfer or neither, and the screen shows only
  what the API confirms. Whether it refuses a currency mismatch or converts is the
  API's call.
- In offline mode the whole screen is replaced by "Not available offline" and the
  core's reason: the paired legs, conversion and cascade are not implemented
  natively. See [Offline mode]({{< relref "/docs/user-guide/offline-mode" >}}).

## UC-24: Reconcile a Transaction

**Screen:** Transaction, `/transactions/:id`. Reconciling confirms that a recorded
transaction matches what an institution reported; the idea is explained in
[Concepts]({{< relref "/docs/concepts#reconciliation" >}}).

1. Open a transaction that is not yet reconciled. A card, "Not yet reconciled",
   appears.
2. If the API has proposed a matching imported record, the card says "The instance
   proposes the imported record below as its match. Reconciling confirms the two
   describe the same movement." and the button reads **Reconcile**.
3. Press it. The transaction's new state appears.

Notable alternatives:

- **No matching record.** The card says "No imported record matches this
  transaction. You can still confirm it yourself — it will be recorded as your own
  confirmation rather than as a match.", and the button reads **Confirm it myself**.
- **Already reconciled.** The action is not offered. The card says "Reconciled" and
  which kind: "Matched to an imported record" or "Confirmed by you".
- **Refused.** The API's reason appears in the card.
- **Offline.** The card says "Not yet reconciled" with a "Not available offline"
  line, because reconciliation needs imported records the native core does not
  create.

## UC-35: Manage a Transaction's Attachments

**Not built.** This use case is open as issue
[#36](https://github.com/artur-rios/fortuna-ui/issues/36), and the code agrees:
nothing in the interface's screens mentions attachments. The transaction screen has
no attachments section, and no control to add, download or remove a file.

The API supports attachments (the endpoints are in the
[OpenAPI document](https://github.com/artur-rios/fortuna-api/blob/develop/docs/openapi/fortuna.json)),
and the interface's generated client includes them, but no screen uses it. The
specification describes the intended behavior: a transaction's attachments shown
beside it, a file chosen with the platform's file picker, a check of its type and
size against the instance's limits before upload, download through the platform's
file saving, and removal after a confirmation. None of that can be used yet.

## Sources

This page is written from the interface's
[Use Case Specification](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/requirements/Use%20Case%20Specification%20Document.md)
(UC-19, 20, 21, 24, 35), checked against the code in
[`lib/features/transactions`](https://github.com/artur-rios/fortuna-ui/tree/develop/lib/features/transactions).
