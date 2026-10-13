---
title: Projections
linkTitle: Projections
weight: 100
description: >
  Where you stand now, where you are heading, and what you have already committed to pay, with forecasts that cannot be mistaken for records.
---

The Projections screen answers two questions: where do I stand now, and where will I
be in the coming months? It shows your net position, a projected balance over a
period you choose, and the obligations you have already committed to.

Nothing on it is stored. The API builds each figure on request, and the interface
marks every forecast as a forecast in words and in shape, not by color alone. The
concepts, including what **committed obligations** are, are in
[Concepts]({{< relref "/docs/concepts#projections-and-committed-obligations" >}}).
This screen is not the home screen: the home address `/` is a placeholder (see
[the guide's start]({{< relref "/docs/user-guide" >}})). The net position,
projections and obligations are not available in desktop offline mode.

| Screen | Address |
| --- | --- |
| Projections | `/insight/projections` |

## UC-38: View the Net Position, Projections and Obligations

**Screen:** Projections, `/insight/projections`.

1. The first card, **Net position**, shows the total across your accounts, cards and
   investments as the API reports it, and "As at" and its date.
2. Under "Looking ahead", choose the period in the drop-down: **Next month**, **Next 3
   months**, **Next 6 months** or **Next year**. Both cards below follow it.
3. The **Projected balance** card shows "Starting from" your recorded balance, then
   one row per month with the projected closing balance, and "Through" and the end
   date.
4. The **Already committed** card lists what you have already committed to pay in the
   period: a total, then each obligation with its kind (**Installment**, **Card
   statement** or **Commitment**), its due date and its amount. The API computes the
   obligations as the remaining installments and the statements not yet settled.
5. Each of those two cards carries the word "Projected" in a tag, an outlined border,
   and the note "These figures are forecasts, not records. Nothing here is stored."

Notable alternatives:

- **Too little history.** The projected-balance card is replaced by the API's reason
  or "There is too little history to project from. A forecast built on nothing would
  be a guess dressed as a figure."
- **You hold nothing.** The net position card says "Nothing yet" and "You hold no
  accounts, cards or investments, so there is nothing to add up. This is not a
  failure to read them."
- **Several currencies.** The net position is not summed across them. The card says
  "Your holdings are in more than one currency, so they are shown separately rather
  than added together." and lists a line per currency. If a display currency is
  chosen and some holdings could not be converted, it says "Some holdings could not
  be converted, and are listed in their own currency below."
- **Overdue.** An obligation past due says "Overdue by N days" (or "1 day") in text as
  well as color.
- **Nothing committed.** "Nothing committed in this period."
- **A request fails.** Each card shows the reason with its own **Try again**.
- **Offline.** The screen is replaced by "Not available offline" and the core's
  reason: reports and projections are not computed natively. See
  [Offline mode]({{< relref "/docs/user-guide/offline-mode" >}}).

## Sources

This page is written from the interface's
[Use Case Specification](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/requirements/Use%20Case%20Specification%20Document.md)
(UC-38), checked against
[`projections_screen.dart`](https://github.com/artur-rios/fortuna-ui/blob/develop/lib/features/insight/ui/projections_screen.dart),
and for the definition of committed obligations against the API's OpenAPI
`CommittedObligationKind` and the API System Requirements Document.
