---
title: Charts
linkTitle: Charts
weight: 90
description: >
  Reading your spending and earning as a chart, and drilling from a chart element down to the transactions behind it.
---

The Insight screen draws your spending and earning as a chart, grouped the way you
choose. Every element can be opened, and the one behind it opened again, until you
reach the individual transactions. That path is the interface's answer to the
question "where did the money go?".

The chart is not computed in the interface. Each aggregation and each level of the
drill-down is requested from the API
([UI-BR-02](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md),
[UI-BR-14](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md)),
and a chart element's value is read from the data behind it, never from where it was
drawn. The sequence is drawn on the
[Architecture]({{< relref "/docs/architecture#chart-drill-down" >}}) page. Charts are
not available in desktop offline mode.

| Screen | Address |
| --- | --- |
| Insight (charts and drill-down) | `/insight` |

## UC-36: Read an Aggregation as a Chart

**Screen:** Insight, `/insight`.

1. The screen opens grouped **By category**, over a period that runs from a year
   ago to today. Three controls sit at the top:
   - a grouping drop-down: **Over time**, **By category**, **By account** or **By
     counterparty**;
   - for **Over time** only, a granularity drop-down: **Daily**, **Weekly**,
     **Monthly** or **Yearly**;
   - a date-range button showing the period, which opens a range picker.
2. The application asks the API for the aggregation and computes none of it.
3. It draws the result as a line chart for **Over time** and a bar chart for the
   other groupings.
4. Below the chart, a list gives every element as text: its label and its total. The
   total carries its currency, which a chart shape cannot.
5. The chart reads in both the light and the dark theme.

Notable alternatives:

- **Several currencies.** Without a display currency chosen in Settings, the
  figures are not added across currencies. A card says "These figures are in more
  than one currency. They are shown separately rather than added together — choose a
  display currency in settings to have the instance convert them." With one chosen,
  an amount the API could not convert is listed in its own currency under its
  element as "Not converted:", and the card says "Some amounts could not be
  converted into … They are listed in their own currency rather than folded into
  the totals." For how conversion works, see
  [Currencies and PTAX]({{< relref "/docs/concepts#currencies-and-ptax" >}}).
- **Nothing in the period.** "Nothing in this period", "There are no records in the
  range and grouping you chose. Try a wider period." This is not a failure.
- **The request fails.** The reason is shown with **Try again**.
- **Too many elements.** The API rolls the smallest elements into a remainder, and
  the remainder is a drillable element like the others (UC-37).
- **Offline.** The whole screen is replaced by "Not available offline" and the
  core's reason: reports are not computed natively. See
  [Offline mode]({{< relref "/docs/user-guide/offline-mode" >}}).

## UC-37: Drill Into a Chart Aggregation

**Screen:** Insight, `/insight`. The drill-down replaces the chart on the same
screen.

1. Tap an element, either in the chart or in the list below it.
2. The application asks the API for the next level, bounded by what you tapped. While
   it does, the grouping and period controls are hidden, because changing them would
   change the question the path is an answer to.
3. The deeper level is drawn the same way as the chart above it, and a breadcrumb
   appears at the top: the grouping's name ("By category", for example), then each
   element you chose. The last crumb is where you are and is not a button.
4. Keep tapping elements to go deeper. At the finest level the screen shows the
   transactions that produced the figure, as cards with the date, the description or
   category, and the amount.
5. Tap a transaction to open it (see
   [Transactions and transfers]({{< relref "/docs/user-guide/transactions" >}})).
6. To go back, tap an earlier crumb. Tapping the first crumb, the grouping's name,
   returns to the unfiltered chart.

Notable alternatives:

- **The transactions may not add up.** A card says "These rows may not add up to the
  figure above them. The instance says so itself — a conversion or a rounding can
  make the two differ." It appears only when the API says so.
- **Nothing at a level.** "Nothing here", "There is no breakdown behind this one.
  Step back to where you came from." The breadcrumb stays so you can step back.
- **The request fails.** The breadcrumb stays, so the level you came from is one tap
  away, and the reason appears with **Try again**.
- **A remainder element** expands and drills like any other.
- **Leaving and returning.** Leaving the screen and coming back finds the path as you
  left it. Restarting the application starts at the chart again.

## Sources

This page is written from the interface's
[Use Case Specification](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/requirements/Use%20Case%20Specification%20Document.md)
(UC-36, UC-37), checked against the code in
[`lib/features/insight`](https://github.com/artur-rios/fortuna-ui/tree/develop/lib/features/insight).
