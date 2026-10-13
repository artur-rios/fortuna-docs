---
title: Imports and connections
linkTitle: Imports
weight: 80
description: >
  Connecting an institution, importing a spreadsheet or statement PDF, following the job, and reviewing the records it brought in.
---

Fortuna brings transactions in from outside in two ways: by **connecting an
institution** through an aggregator, and by **importing a file**. Either one starts
a job on the instance, and you follow the job and review what it did. How the
pipeline works, with its duplicate check and its per-row outcomes, is on the
[Architecture]({{< relref "/docs/architecture#the-import-pipeline" >}}) page, and
the rules are in [Concepts]({{< relref "/docs/concepts#imports" >}}).

Three things to know. The interface uploads a file whole and untouched: it does not
parse, transform, filter or repair it
([UI-BR-28](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md)).
A raw imported record can never be edited, because it is the evidence the import is
reconciled against, so you correct the transaction it produced instead. And file
imports are not available in desktop offline mode; connections are not offered there
either.

| Screen | Address |
| --- | --- |
| Data sources | `/sources` |
| Connections | `/connections` |
| Imports (the job list) | `/imports` |
| Import a file | `/imports/new` |
| Imported records (one job) | `/imports/:id/records` |

## UC-30: Connect an Institution

**Screen:** Data sources, `/sources`. Not offered in desktop offline mode.

1. The screen lists the data sources the instance supports. Each shows its name and
   an icon for a file source or a network source. A source that cannot be used in
   the current mode is marked, with the instance's own reason, or "Not available in
   this mode." if it gave none. A network source that is available has a
   **Connect** button.
2. Press **Connect**. If the source needs consent for external processing and you do
   not hold a current one, a disclosure dialog appears. It is titled "Before
   connecting", or "The disclosure has changed" if you agreed to an earlier version.
   It says "Connecting an institution shares your financial data with an external
   service so it can be read on your behalf." and lists what is shared (your account
   and transaction data), who with (the aggregation service this instance is
   configured to use, reached by the instance and never by this application) and why
   (so movements can be imported without being entered by hand). It adds "You can
   withdraw this at any time from Privacy." and the disclosure's version.
3. Choose **I agree** or **Not now**. The decision is recorded against that version.
   Declining starts nothing and discloses nothing.
4. The dialog "Connect …" asks for the **Institution**, "As the instance identifies
   it." or "Required:" followed by what the source needs. Press **Connect**.
5. The dialog then says "Connected" and shows the connection and its state, with
   **Done**.

Notable alternatives:

- **No institution named.** "Name the institution to connect."
- **The API names a missing consent.** The reason appears with **Review the
  disclosure**, which shows the dialog and then retries.
- **The aggregator is unavailable.** The failure shows with **Try again** where
  retrying could help.
- **Already connected.** The dialog says "Already connected" and "This institution is
  already connected. Nothing new was created — here is the existing one."
- **Cancelling** the dialog creates nothing.
- If the instance reports no sources: "This instance supports no data sources."

## UC-31: Synchronize, Reauthenticate or Revoke a Connection

**Screen:** Connections, `/connections`. The top bar has a refresh icon and a
sign-out button.

1. The screen lists your connections as cards, each with its state (**Active**,
   **Needs reauthentication**, **Revoked** or **Unknown**), "Connected" and the date,
   and the institution reference.
2. Press **Synchronize now**. A message says "Synchronization started. It runs on the
   instance — you can follow it under Imports." While it runs the card shows a
   progress bar and "Synchronizing now — N rows read so far", and the button reads
   "Synchronizing…" and is disabled.
3. A connection that needs reauthentication is shown prominently: "This connection
   has stopped bringing data in until you authenticate with the institution again."
   Its button becomes **Reauthenticate**, which asks "Reauthenticate this
   connection?" and says "You will be asked to sign in with the institution again.
   Fortuna never sees those credentials — they stay with the provider." Choose
   **Continue** or **Cancel**.
4. Press **Revoke** to end a connection. The dialog "Revoke this connection?" says
   "It will stop synchronizing, and Fortuna will no longer reach the institution.
   Everything it has already imported stays exactly as it is. Cutting off a bank link
   is not a decision to erase your history." Choose **Revoke** or **Cancel**.

Notable alternatives:

- A revoked connection offers no action: "Revoked. Everything it already imported is
  still here."
- A synchronization already running is shown instead of starting a second one.
- A failure from the aggregator is shown with its reason.
- With none: "No connections", "A connection lets an institution send its
  transactions to your instance. Fortuna never holds the credentials."
- Withdrawing the consent also revokes the connections that depend on it; see
  [Privacy and data rights]({{< relref "/docs/user-guide/data-rights" >}}).

## UC-32: Import a File

**Screen:** Import a file, `/imports/new`. Not available in desktop offline mode,
where the screen is replaced by "Not available offline" and the core's reason: there
is no native workbook or PDF parser.

1. The page says "The file is uploaded exactly as it is. Fortuna reads it; nothing
   here changes it." and offers two sources: **Spreadsheet** ("An Excel workbook
   exported from a bank or kept by hand.", accepts .xlsx or .xls) and **Statement
   PDF** ("A credit card invoice in one of the supported layouts.", accepts .pdf).
   The one supported PDF layout is the Nubank credit-card invoice.
2. Press **Choose file** on one of them and pick a file with the platform's file
   picker.
3. The application checks the type and the size. A name that is not accepted is
   refused: "… is not a file this source reads. Choose a .xlsx or .xls file." A file
   over 25 MB is refused: "… is N MB, and the limit is 25.0 MB." In both cases the
   card "That file was not sent" explains, and nothing leaves your device.
4. A good file shows its name and "N KB, ready to send." with **Import this file**.
5. While it uploads the card reads "Sending …" with a progress bar.
6. When the API accepts the upload it answers with a job, and the card says "The
   import has started": "It runs on the instance. You can follow it under Imports,
   and it keeps going if you leave this screen." **Import another file** starts over.

Notable alternatives:

- Cancelling the picker does nothing and shows no error.
- A failed upload shows "The import did not start" and the reason, with **Try …
  again** for the same file, and no partial import is claimed.
- An unreadable file is refused by the API and its reason is shown; the interface
  does not try to fix it.

**What this screen does not ask for.** It sends the file and nothing else. The
API's import endpoints also take a target and, for a workbook, a column mapping:
a financial account or credit card for a spreadsheet, a credit card for a PDF, and
the date, amount and direction columns. The API's own checks require them (see the
[Excel](https://github.com/artur-rios/fortuna-api/blob/develop/src/Application/ArturRios.Fortuna.Command/Input/Validation/ImportExcelWorkbookCommandValidator.cs)
and
[PDF](https://github.com/artur-rios/fortuna-api/blob/develop/src/Application/ArturRios.Fortuna.Command/Input/Validation/ImportPdfInvoiceCommandValidator.cs)
validators). This version's screen has no field for any of them, so as built, expect
the API to refuse an upload started here, and the refusal appears on this screen as
"The import did not start". The mismatch is between the interface and the API, not
something you can work around on this screen.

## UC-33: Monitor an Import Job

**Screen:** Imports, `/imports`. The top bar has a refresh icon and a sign-out
button. It lists imports and bank synchronizations, newest first, and they stay
after they finish.

1. Each job is a card with a state: **Queued**, **Running**, **Finished**, **Failed**
   or **Unknown**, and its start time.
2. A job that has not finished shows a progress bar and "Starting…" or "N rows read so
   far". The list checks again every three seconds while anything is running.
3. Leave the screen if you like; the job continues on the instance and you can come
   back to it.
4. When it finishes the card shows the per-row outcomes as the API reported them,
   as three counts: **imported**, **skipped as duplicates** and **rejected**. It
   never reduces them to one verdict.
5. **Review the records** opens the imported records (UC-34).

Notable alternatives:

- **A job fails.** The card shows the reason (or "The import failed.") and **Retry
  this import**. In offline mode, where retry is not available, a "Not available
  offline" line shows instead.
- **Some rows were rejected.** The card says "Some rows landed and some did not. The
  rows that failed did not stop the others." A partial import is never reported as a
  plain success or failure.
- **No imports.** "No imports yet", "Imports and bank synchronizations appear here
  while they run, and stay afterwards so you can see what they did."
- **Session ends.** The job belongs to the API, so after you sign in again you find
  it here.
- The interface gives no progress percentage and does not invent one.
- There is no control to dismiss a finished job; it stays in the list.

## UC-34: Review Imported Records

**Screen:** Imported records, `/imports/:id/records`. Open it from **Review the
records** on a finished job.

1. A notice explains: "These are the records exactly as they arrived, and they cannot
   be changed. They are what the import is reconciled against. To correct something,
   open the transaction a record produced and edit that instead."
2. The records are listed in pages. Each shows an outcome chip (**Imported**,
   **Already present**, **Rejected** or **Unknown**), the date, the amount as it
   arrived (as text, with no currency, because a raw record carries none) and a
   "Reference" if there is one.
3. Where a transaction came from the record, **Open the transaction** goes to it.
   **The row as it arrived**, an expandable section, shows the raw row as selectable
   text.
4. When there is more than one page, a control at the bottom shows "Page X of Y".

Notable alternatives:

- **A record that produced no transaction** shows the reason the API recorded, or "This
  row was already present, so no transaction was created from it." for a duplicate,
  or "No transaction was created from this row."
- **The transaction was deleted afterwards:** "The transaction this produced has
  since been deleted."
- **No records.** "This import took in no records." The page control appears only when
  there is more than one page. A failed load shows the reason with **Try again**.

## Sources

This page is written from the interface's
[Use Case Specification](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/requirements/Use%20Case%20Specification%20Document.md)
(UC-30 to UC-34), checked against the code in
[`lib/features/ingestion`](https://github.com/artur-rios/fortuna-ui/tree/develop/lib/features/ingestion)
and the two API validators linked above.
