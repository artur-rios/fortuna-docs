---
title: Exports
linkTitle: Exports
weight: 110
description: >
  Taking the transactions view out of Fortuna as a CSV, Excel or PDF file.
---

An export takes the data in a view out of Fortuna as a file. In this version the one
view you can export is the transactions spreadsheet. The API builds the file, and the
interface only asks for it and saves it, so what you save is exactly what the
instance holds
([UI-BR-29](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md)).
The rules for exports, including what an export leaves out, are in
[Concepts]({{< relref "/docs/concepts#exports-and-attachments" >}}).

This is not the same as taking all of your data. The complete personal-data archive,
the portability right, is on the
[Privacy and data rights]({{< relref "/docs/user-guide/data-rights" >}}) page.

| Screen | Address |
| --- | --- |
| Export (a sheet over the transactions screen) | none of its own; open it from `/transactions` |

## UC-39: Export a Data Set

**Where:** the **Export** icon in the top bar of the
[Spreadsheet]({{< relref "/docs/user-guide/spreadsheet" >}}).

1. Press **Export**. A sheet titled "Export" opens.
2. It states what the export will cover, under "What this will cover": either
   "Everything in this view. No filters are applied." or one line for each filter in
   words, for example "From 2026-01-01", "Up to 2026-03-31", "One account only", "One
   category only" or "Expenses only".
3. Choose the **Format**: **CSV**, **Excel** or **PDF**.
4. Press **Export**. The sheet reads "Asking the instance to produce it…" and then "The
   instance is producing it.", with a progress bar. It adds "You can close this and
   carry on — the export keeps going, and you will find it with your other jobs." and a
   **Close** button.
   A small export is delivered at once, and the sheet goes straight to the next step.
5. When the file is ready the sheet says "The export is ready." Press **Save it**. The
   platform's own save dialog opens, and then the sheet says "Saved." with the place,
   and **Done**.

Notable alternatives:

- **Closing the sheet** leaves the export running, as the sheet says; the application
  keeps checking on it. The Imports screen lists import jobs only, so an export does
  not appear there. Press **Export** on the spreadsheet again to see where it has got
  to.
- **An empty view.** The sheet says "There is nothing in this view to export. Widen the
  filters, or record something first." and offers only **Close**.
- **The job fails.** The reason is shown with **Try again**.
- **The save location is refused,** or you close the save dialog. The sheet says "The
  file is still here. Choose another place for it." and the button reads **Choose
  another place**. The file is still there to save.
- **The export expired.** The sheet says "That export is no longer available — the
  instance keeps one only for a while. Producing another takes a moment." with
  **Produce another**.
- **A format the API will not produce for this data** is refused by the API in its own
  words.
- **Offline.** The Export icon is disabled, and its tooltip reads "Not available
  offline" followed by the core's reason: there is no data-set export renderer in the
  native core. See [Offline mode]({{< relref "/docs/user-guide/offline-mode" >}}).

## Sources

This page is written from the interface's
[Use Case Specification](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/requirements/Use%20Case%20Specification%20Document.md)
(UC-39), checked against
[`export_sheet.dart`](https://github.com/artur-rios/fortuna-ui/blob/develop/lib/features/insight/ui/export_sheet.dart)
and the filter wording in
[`transaction_table.dart`](https://github.com/artur-rios/fortuna-ui/blob/develop/lib/features/transactions/state/transaction_table.dart).
