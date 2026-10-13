---
title: Categories, tags, budgets and goals
linkTitle: Organizing
weight: 70
description: >
  Grouping spending with categories, tags and counterparties, setting budget ceilings, and tracking savings goals.
---

These screens are how you give your records meaning. A **category** says what kind
of spending or earning a transaction is, and categories form a tree. **Tags** are
free labels and **counterparties** are the shops, employers and people on the
other side. A **budget** is a ceiling for some categories over a period, and a
**goal** is a savings target with a date. The rules are in
[Concepts]({{< relref "/docs/concepts#categories-tags-counterparties-budgets-and-goals" >}}).

Spent, remaining and progress figures are the API's. The interface never works them
out, and shows the ceiling or target on its own, with the reason, when it cannot
read them.

| Screen | Address |
| --- | --- |
| Categories | `/categories` |
| Tags and counterparties | `/labels` |
| Budgets | `/budgets` |
| Goals | `/goals` |

## UC-26: Manage the Category Tree

**Screen:** Categories, `/categories`. The top bar has a sign-out button.

1. The screen shows your categories as an indented tree. A category with children
   has a folder icon.
2. Press **New category**. The dialog "New category" asks for a **Name** ("A
   category needs a name.") and a **Parent**, where the first choice is "None — a
   top-level category". Press **Create**.
3. The parent list offers only categories that would not create a cycle, so a
   category cannot be placed under itself or one of its own descendants.
4. To change a category, press its pencil icon (tooltip "Edit" and the name) for the
   dialog "Edit …", then **Save**. The trash icon deletes it.
5. Deleting asks "Delete …?" and says "It can be restored afterwards." or, for a
   category with children, "It has N categories beneath it. It can be restored
   afterwards." Choose **Delete** or **Cancel**.

Notable alternatives:

- **Deletion refused.** If transactions still use the category, the API's reason
  appears at the bottom of the screen with a **Reassign** action. It opens "Move
  transactions out of …", which says "Every transaction in this category, and in the
  categories beneath it, moves to the one you choose." Pick the destination in
  **Move them to** (the same category is not offered) and press **Move**. Then
  delete again.
- **A duplicate sibling name** is refused by the API and its reason is shown in the
  dialog.
- **No categories yet.** "No categories yet", "Categories are how spending is
  grouped and budgeted.", with **Create the first one**. A failed load shows "The
  categories could not be loaded." and **Retry**.

## UC-27: Manage Tags and Counterparties

**Screen:** Tags and counterparties, `/labels`, with two tabs, **Tags** and
**Counterparties**, and a sign-out button.

1. Each tab lists your tags or counterparties by name. An empty tab explains what
   they are: "Tags are free-form labels. A transaction can carry several." or "A
   counterparty is who was on the other side — a shop, an employer, a person."
2. Press **New tag** or **New counterparty**, type the **Name** ("A tag needs a
   name." or "A counterparty needs a name.") and press **Create**.
3. To rename, press the pencil icon (tooltip "Rename …") and **Save**. The trash
   icon asks "Delete …?" with "It can be restored afterwards."; choose **Delete**
   or **Cancel**.
4. In the record and edit forms you attach them by typing names; see
   [Transactions and transfers]({{< relref "/docs/user-guide/transactions" >}}).

Notable alternatives:

- A duplicate name, or a deletion refused because transactions still use it, shows
  the API's reason.
- **None yet.** "No tags yet" or "No counterparties yet", with the explanation
  above and **Create the first tag** (or counterparty). A failed load offers
  **Retry**.
- The Deleted records screen lists transactions, accounts and categories only, so
  although the dialog says a deleted tag or counterparty can be restored, that screen
  is not where.

## UC-28: Manage Budgets

**Screen:** Budgets, `/budgets`.

1. The screen lists your budgets. Each card names its categories (or
   "Uncategorized"), shows the period (Monthly, Quarterly or Yearly) and, if it was
   set so, "Includes sub-categories".
2. Press **New budget**. The dialog "New budget" asks for a **Ceiling**, a
   **Currency** (chosen when creating, not editable afterwards), a **Period**, a
   start date ("Choose when the period starts", then "Starts …"), **Categories** as
   checkboxes, and **Include sub-categories**.
3. The form checks "The budget must be greater than zero.", "Choose the date the
   period starts on." and "Choose at least one category for this budget to cover."
   Press **Create**.
4. Each card then shows three figures from the API: **Spent**, **Remaining** (or
   **Over by** when exceeded) and **Ceiling**, with the period's dates beneath. An
   exceeded budget also carries a warning and the words "Over the budget".
5. Tap a card to edit it. **Delete** is in that dialog.

Notable alternatives:

- **Consumption cannot be read.** The card says "What has been spent could not be
  read. The ceiling is shown on its own." and shows only the ceiling.
- **A budget already covering that category and period** is refused by the API and
  its reason appears in the dialog.
- **No categories.** The screen says "Create a category first", "A budget is a
  ceiling for a category, so there needs to be one to budget for.", with a **Go to
  categories** button. With no budgets: "No budgets yet" and **Add a budget**.
- **Offline.** Each card shows its ceiling with a "Not available offline" line in
  place of the spent figure, because consumption is not computed natively. See
  [Offline mode]({{< relref "/docs/user-guide/offline-mode" >}}).

## UC-29: Manage Goals

**Screen:** Goals, `/goals`.

1. The screen lists your goals. Each card shows the name, "By" and the target date,
   and the progress the API reports: a bar and **Saved**, **Remaining** and
   **Target**. A goal that has reached its target is marked "Reached".
2. Press **New goal**. The dialog "New goal" asks for a **Name**, a **Target**, a
   **Currency** (chosen when creating) and the date, "Choose the date to reach it
   by". The date picker starts tomorrow.
3. The form checks "Give the goal a name.", "The target must be greater than zero."
   and "The target date must be in the future. A goal is something to reach, not
   something already past." Press **Create**.
4. Tap a card to edit it; **Delete** is in that dialog.

Notable alternatives:

- **The date has passed.** The goal stays, marked with a history icon and "The date
  has passed. This is how far it got." or, if the target was met, "The date has
  passed, and the target was met." It is never silently removed.
- **Progress cannot be read.** The card shows the target alone, with the reason.
- **Linked accounts.** If the API reports accounts or investments linked to a goal,
  the card adds "From" and their names. This version's goal form has no control to
  link accounts or investments, so a goal made here has none to show.
- **Offline.** Progress shows a "Not available offline" line, because it is not
  computed natively.
- With no goals: "No goals yet", "Set a target and a date, and track what you have
  put aside toward it.", with **Add a goal**.

## Sources

This page is written from the interface's
[Use Case Specification](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/requirements/Use%20Case%20Specification%20Document.md)
(UC-26 to UC-29), checked against the code in
[`lib/features/categories`](https://github.com/artur-rios/fortuna-ui/tree/develop/lib/features/categories),
[`lib/features/labels`](https://github.com/artur-rios/fortuna-ui/tree/develop/lib/features/labels)
and
[`lib/features/planning`](https://github.com/artur-rios/fortuna-ui/tree/develop/lib/features/planning).
