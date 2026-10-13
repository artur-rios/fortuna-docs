---
title: Deleted records and the audit trail
linkTitle: Deleted records and audit
weight: 120
description: >
  Bringing back something deleted by mistake, removing it for good, and reading the record of significant actions on your data.
---

Deleting a record in Fortuna is the first of two steps. A deleted record leaves every
balance, chart, projection and export, but it can still be found and restored. Only a
record that has been deleted can then be removed permanently. The audit trail is the
append-only record of significant actions on your records. The rules are in
[Concepts]({{< relref "/docs/concepts#deleting-restoring-and-the-audit-trail" >}}).

Erasing your whole account is a different and irreversible operation, on the
[Privacy and data rights]({{< relref "/docs/user-guide/data-rights" >}}) page.

| Screen | Address |
| --- | --- |
| Deleted records | `/deleted` |
| Audit trail | `/audit` |

## UC-40: Restore or Permanently Remove a Deleted Record

**Screen:** Deleted records, `/deleted`. This is the only view in which deleted
records appear
([UI-BR-12](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md)).
In this version it lists deleted **transactions, accounts and categories**.

1. Each deleted record is a card with a tag such as "Deleted transaction", its name,
   its amount and date where it has them, and a second line such as an institution or
   currency. If other records still refer to it, a line says "N other records still
   refer to this one."
2. Press **Restore** to bring the record back into the views it belongs to. Nothing
   asks you to confirm; restoring can be undone by deleting again.
3. To remove a record for good, press **Remove permanently**. It is a separate
   control, in the warning color.
4. A dialog, "Remove permanently?", names the record ("… will be removed for good.")
   and says "This cannot be undone. Unlike deleting, there is no way to bring it back
   afterwards." If other records still refer to it, it adds how many, and that "The
   instance may refuse to remove it, or may take them with it." Choose **Remove
   permanently** or **Keep it**.
5. The result is reported on the card.

Notable alternatives:

- **Removal refused.** The API's reason, which names what still refers to the record,
  appears on the card. The control is not hidden.
- **Restoring refused.** The API's reason appears on the card.
- **Nothing is deleted.** "Nothing is deleted", "Records you delete appear here, where
  they can be brought back or removed for good."
- **A live record** is never offered permanent removal. Deleting always comes first.
- A failed load shows the reason and **Try again**.

## UC-41: Read the Audit Trail

**Screen:** Audit trail, `/audit`. The top bar has a refresh icon and a sign-out
button.

1. The screen lists the entries for your own records, newest first. Each shows the
   action and the kind of record, as "Create · Transaction" for example, with a result
   icon (succeeded, refused or unknown), the time in your time zone, the refusal reason
   if there was one, and "Record" followed by the record's reference.
2. A notice at the bottom says "The audit trail is append-only. Entries cannot be
   changed or removed, including by you — a trail that can be pruned is not one."
   There are no edit or delete controls.
3. Narrow the list with **Any period**, a range picker, and **Action**, which offers
   **Any action**, **Created**, **Updated**, **Deleted**, **Imported** and
   **Exported**. **Clear filters** appears when either is set.

Notable alternatives:

- **No entries.** "Nothing recorded yet", "Significant actions on your records appear
  here as they happen." With filters: "Nothing in that period" and "No entries match
  the filters you have set.", with **Clear filters**.
- **An entry for a record since removed** is shown with what it recorded, and its
  reference is plain text, not a link, so it does not pretend the record can still be
  opened.
- A failed load shows "The audit trail could not be read." or the reason, with
  **Retry**.

## Sources

This page is written from the interface's
[Use Case Specification](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/requirements/Use%20Case%20Specification%20Document.md)
(UC-40, UC-41), checked against the code in
[`lib/features/lifecycle`](https://github.com/artur-rios/fortuna-ui/tree/develop/lib/features/lifecycle)
and
[`lib/features/audit`](https://github.com/artur-rios/fortuna-ui/tree/develop/lib/features/audit).
