---
title: Spreadsheet
linkTitle: Spreadsheet
weight: 60
description: >
  Reading your transactions as a filterable, sortable, paged grid.
---

The spreadsheet is the interface's table view. It shows your transactions as a
grid you can filter, sort and page through, and it is where most of your records
are found and opened. Nothing on it changes anything: it only reads.

The grid does not hold your records and filter them itself. Every filter, every
sort and every page is a request to the API, which answers with just that page
([UI-BR-02](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md),
[UI-BR-03](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md)).

| Screen | Address |
| --- | --- |
| Transactions (the spreadsheet) | `/transactions` |

## UC-25: Explore Records as a Spreadsheet

**Screen:** Transactions, `/transactions`.

1. The screen asks the API for the first page and shows a loading spinner while it
   waits. The records are newest first by default.
2. They appear in a grid with five columns: **Date**, **Amount**, **Category**,
   **Account or card** and **Description**. A transfer shows "Transfer" in the
   description column, and a missing value shows "—". Amounts are shown with their
   own currency.
3. Narrow the set with the controls above the grid:
   - a date-range button, which reads "Any period" until you pick a range;
   - **Any account**, a drop-down of your accounts;
   - **Any category**, a drop-down of your categories;
   - **Any direction**, with Expense and Earning.

   **Clear filters** appears once any filter is set. The API receives the filters
   and answers with the matching page.
4. Tap a column heading to sort by it, and again to reverse the order. The API
   sorts.
5. Use the bar at the bottom to page: it shows "N records" (or "1 record"), previous
   and next arrows, and "Page X of Y". A page holds 25 records.
6. Tap a row to open the transaction (see
   [Transactions and transfers]({{< relref "/docs/user-guide/transactions" >}})).
   Going back, you find the filters, sort and page as you left them.

The top bar has an **Export** icon (see [Exports]({{< relref "/docs/user-guide/exports" >}}))
and a **Record** button opens the record form.

Notable alternatives:

- **A backwards date range.** If the start is after the end, the screen says "The
  start of the range is after its end. Nothing was requested." and sends no request.
- **No records.** With no filters, the screen says "No records yet" and "Records you
  enter or import will appear here." With filters, "No records match these filters"
  and "Nothing here matched. The filters may be narrower than you meant.", with a
  **Clear the filters** button.
- **The request fails.** The reason is shown with **Try again**. No earlier page is
  presented as current while a new one loads.
- **A grid wider than the window** scrolls sideways inside its own area; the page
  itself does not.
- **Filters are remembered.** The view (filters, sort and page) is saved in the
  device's preferences and restored when the application starts, so a reload keeps
  your filters.

What the grid does not offer in this version: the filters for tag, counterparty, a
single credit card and an amount range, and free-text search. The code carries them
for later, but no control sets them.

## Sources

This page is written from the interface's
[Use Case Specification](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/requirements/Use%20Case%20Specification%20Document.md)
(UC-25), checked against the code in
[`transactions_screen.dart`](https://github.com/artur-rios/fortuna-ui/blob/develop/lib/features/transactions/ui/transactions_screen.dart)
and
[`transaction_table.dart`](https://github.com/artur-rios/fortuna-ui/blob/develop/lib/features/transactions/state/transaction_table.dart).
