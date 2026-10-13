---
title: Concepts
linkTitle: Concepts
weight: 20
description: >
  The ideas behind Fortuna, with the business rules a user needs to know.
---

This page explains the ideas Fortuna is built on, in plain terms, so that the
rest of the documentation can use them without stopping to define them. Each
section ends with the business rules that govern the idea.

## How the business rules are cited

Each repository keeps a Business Rules document, and both number their rules
`BR-01` onwards, so the numbers collide. On this site a rule is always written
with a prefix.

- **API-BR-xx** is a rule from the
  [Fortuna API Business Rules](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md).
  They define what the domain must enforce, independently of how it is built.
- **UI-BR-xx** is a rule from the
  [Fortuna UI Business Rules](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md).
  They define what the application may display and decide, and where it must
  defer to the API.

The rules are quoted in spirit, not word for word. The linked documents hold the
exact wording and the reason behind each rule. This page lists the rules that
matter to someone using Fortuna, which is a selection and not the full set.

## Ownership and isolation

Every financial record belongs to exactly one user, and only that user can read
or change it. That holds on a shared instance too: other users of the same
instance cannot see your data.

- [API-BR-01](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): every financial record belongs to exactly one user, and only that
  user may read or modify it.
- [API-BR-02](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): an instance administrator manages users, integrations and
  operational health, and has no access to any user's financial records.
- [API-BR-03](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): a record's owner is fixed at creation and never changes.
- [API-BR-04](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): an operation that would touch another user's record is refused as
  if the record did not exist. You get "not found", not "forbidden", so the
  existence of someone else's data is not revealed.
- [UI-BR-21](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md): the interface presents only what the signed-in role may do. An
  owner sees no administrative surface, and an administrator sees no financial
  data anywhere.

## Holdings

A holding is anything that holds value and has a history: a financial account, a
credit card or an investment. Each has exactly one currency, set when it is
created and never changed afterwards.

### Financial accounts

A financial account is a bank account or cash holding. It has a name, an
institution, a type (checking, savings, cash or other), a currency and an
opening balance, which may be zero or negative. Its balance is never stored or
edited: it is the opening balance plus its live transactions, derived every
time.

- [API-BR-08](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): an account, card or investment has one currency, set at creation
  and immutable afterwards.
- [API-BR-11](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): a balance, total or aggregate is always derived from the
  underlying records, never a stored number you can edit.

### Credit cards and statements

A credit card is not a bank account. It has an issuer, a limit above zero, a
currency, and a closing day and a due day between 1 and 31. Its charges
accumulate into **statements**, one per billing cycle, and a statement is
settled by a payment from a financial account.

A statement moves forward through three states and never back: Open (collecting
charges), Closed (past the closing date, total fixed) and Settled (a payment has
cleared it). A charge belongs to the statement whose billing cycle contains its
date, which the card's closing day determines. Once a statement is settled it is
frozen. A record that arrives afterwards and falls in its period goes to the next
open statement and is flagged as late-arriving.

- [API-BR-12](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): a card's charges accumulate into a statement, and settling it is
  a transfer, not an expense. Counting both the purchase and the invoice payment
  as expenses would double that month's spending.
- [API-BR-13](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): a transaction falls into the statement whose billing cycle
  contains its date.
- [API-BR-14](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): a settled statement's composition is frozen, and a later arrival
  goes to the next open statement, flagged.

In the interface (UC-16), closing a statement asks for a confirmation stating
that closing fixes its composition, and settling it is shown as a transfer
between your own accounts, not as an expense. Card statements are one of the
things desktop offline mode does not offer.

### Investments

An investment is a held instrument with a type (fixed income, equity, fund or
other) and a currency. Fortuna records what happened to it: contributions,
withdrawals, yields and fees, and valuations on given dates. A position is worked
out from those movements and the most recent valuation. Fortuna uses no market
data.

- [API-BR-17](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): Fortuna records investment valuations and never prices an
  instrument itself.

## Money movement

### Transactions

A transaction is one movement of money, an expense or an earning, against a
financial account or a credit card. It is the core entity of the system.

A transaction has a date, an amount, a direction, an owning account and a
category. The amount is always positive, and the direction carries the sign. A
date more than one day in the future is rejected, because a future movement is a
recurring rule or a projection, not a transaction. Description, counterparty and
tags are optional.

A transaction's states run from Recorded, to Reconciled, to soft-deleted, to
hard-deleted. See [Reconciliation](#reconciliation) and
[Deleting, restoring and the audit trail](#deleting-restoring-and-the-audit-trail).

- [API-BR-18](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): every transaction has a date, an amount, a direction, an owning
  account and a category.
- [API-BR-19](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): an amount is recorded as a positive value, and the direction
  carries the sign.
- [API-BR-09](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): a transaction is in its account's currency. If the original was
  in another currency, both the original amount and its currency are kept.
- [API-BR-25](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): every transaction records the source it came from and, when it was
  not entered by hand, the imported record it derives from.

### Transfers

A transfer moves money between two of your own accounts. It is recorded once and
produces two paired legs that are applied together or not at all. It is neither
an earning nor an expense, so it is left out of income and expense totals.
Deleting either leg deletes both.

- [API-BR-15](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): a transfer is neither an earning nor an expense, and both sides
  move together or neither does.
- [API-BR-16](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): the origin and the destination must be different accounts.

Transfers are not available in desktop offline mode.

## Installments

An installment plan splits one purchase into a fixed number of future charges,
most often on a credit card. A plan has at least two installments. Each one is
placed in a successive billing cycle of the card it was bought on.

The installments always add up to exactly the purchase total. Any rounding
remainder goes on the first installment, so 100.00 split three ways becomes
33.34, 33.33 and 33.33. Evenly splitting 100.00 would have given 99.99.

- [API-BR-20](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): installment amounts sum exactly to the purchase total, with any
  rounding remainder on the first installment.

Installment purchases are not available in desktop offline mode.

## Recurring commitments

A recurring commitment is a rule that generates transactions on a schedule, such
as a salary, rent or a subscription. It has a frequency (weekly, monthly,
quarterly or yearly), a start date, and an optional end date that is not before
the start.

The rule is not a movement. Only the occurrences that have been materialized,
turned into real transactions, count as transactions. Occurrences still in the
future exist only inside projections, so next year's rent is not in this month's
balance. Materialization is safe to repeat: running it twice for the same period
does not create the same occurrence twice. If a materialized occurrence matches a
transaction already imported from a bank, it is created and flagged as a possible
duplicate for you to resolve.

- [API-BR-21](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): a recurring transaction is a rule, not a movement, and only
  materialized occurrences are real transactions.
- [API-BR-22](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): editing a rule affects future occurrences only.

The interface does not yet cover this. Managing recurring commitments (UC-23)
has no screen and is an open issue; see the [Overview]({{< relref "/docs/overview" >}}).
The API supports them, and recurring commitments are not materialized in desktop
offline mode.

## Projections and committed obligations

A projection is a forward-looking figure. The API builds it on request from
recurring commitments, unbilled installments and statements due. It is never
stored as if it were history, and it is recomputed from current data each time.
A projection can also include a component estimated from your own historical
averages, but only when you ask for it, and it is labelled as estimated and kept
separate from the committed amounts.

**Committed obligations** are the figures the API reports for money you already
owe: the remaining installments and the statements that are not yet settled, with
the dates they fall due.

- [API-BR-31](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): a projection is derived on demand and never persisted as history.
- [API-BR-32](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): projected figures are always distinguishable from recorded ones.
- [UI-BR-11](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md): the interface shows a projected or scheduled figure as visibly
  distinct from a recorded one, in every view that mixes them.

Net position and projections are not available in desktop offline mode.

## Categories, tags, counterparties, budgets and goals

These are the ways you organize and plan.

**Categories** form a tree. A category has at most one parent and the hierarchy
cannot contain a cycle. Names are unique among siblings. You can reassign all
the transactions of one category to another in a single operation, and a
category cannot be hard-deleted while live transactions still use it.

**Tags** are free-form labels, unique per user, that you can attach to a
transaction in any number. They are separate from the category tree.

**Counterparties** are the merchant, payer or payee on the other side of a
transaction. An incoming counterparty name is matched to an existing one by its
normalized form before a new one is created, and Fortuna suggests a category from
the ones you most recently used for the same counterparty.

**Budgets** set a ceiling over one or more categories for a period (monthly,
quarterly or yearly), with an amount above zero. The API reports how much has
been spent, whether the budget is exceeded, and by how much. Transfers and
soft-deleted records are not counted.

**Goals** are savings targets: a target amount above zero, a target date that is
in the future when the goal is created, and the accounts or investments linked to
it. Progress is measured from the current balances of the linked accounts.

- [API-BR-23](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): a category has at most one parent, and the hierarchy has no cycle.

The Business Rules documents number no rule for budgets or goals. Their
constraints are in the validation tables of the
[API](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md)
and [UI](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md)
Business Rules documents. Budget consumption and goal progress are not available
in desktop offline mode.

## Reconciliation

Reconciling confirms that a recorded transaction matches what the institution
reported. You match a transaction to an imported record, or accept a match the
system proposed, and the transaction is marked reconciled with a link to the
record that matched it.

- A record that is already linked to another transaction is refused (`409`).
- A transaction that is already reconciled is refused (`409`). Unreconcile it
  first, which removes the link and frees the record to be matched again.
- If the amounts or dates differ beyond a tolerance, the match is accepted but
  flagged as a discrepancy. The tolerances are an operator setting; their
  defaults are 0.01 for the amount and 1 day for the date.

The raw imported record is never edited. Corrections are made on the transaction
derived from it ([API-BR-24](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md)). Reconciliation is not available in desktop
offline mode.

## Currencies and PTAX

Every amount carries its currency, and money is held as an exact decimal, never
as binary floating point. The API publishes decimals as strings in JSON so no
precision is lost on the way. The reason is simple: `0.1 + 0.2` must not become
`0.30000000000000004` in a finance system.

- [API-BR-05](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): every monetary amount is an exact decimal carrying its currency,
  and binary floating point is never used for money.
- [API-BR-06](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): amounts in different currencies are never summed, compared or
  netted without an explicit conversion.
- [API-BR-07](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): every conversion records the rate applied and the date it was
  taken from.
- [API-BR-10](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): rounding is applied only where a result is presented or
  converted, to the currency's own minor-unit precision. The API rounds half away
  from zero.
- [UI-BR-06](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md): the interface holds an amount as an exact decimal and never
  converts it to floating point.
- [UI-BR-07](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md): the interface shows every amount with its own currency and never
  visually sums amounts in different currencies unless the API converted them.
- [UI-BR-09](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md): amounts, dates and numbers follow your chosen locale, and the
  locale changes only how a value is rendered, never the value.

### PTAX rates

Exchange rates come from the Banco Central do Brasil's PTAX publication, read
through its free Olinda service. PTAX publishes each currency against the
Brazilian real and against the US dollar, so Fortuna can derive a cross rate for
a pair the source does not quote directly.

- Rates are synchronized on a schedule. If the source publishes nothing for a
  weekend or a holiday, the most recent earlier publication is used, attributed
  to its own date.
- If the rate source cannot be reached, Fortuna keeps working with the most
  recent known rate and marks the result with that rate's date.
- A rate can also be entered by hand. A manual rate takes precedence over the
  published one for that pair and date, and it is global: it applies to every
  user. Recording a manual rate, and triggering a synchronization, need a system
  administrator or the local installation owner, and others are refused with
  `403`.
- A rate must be above zero, and its two currencies must differ.

### Display currency

You can read figures in a display currency. Fortuna groups figures by their own
currency, converts each group's total once with the rate on the figure's date,
and reports the rate and its date for every source currency. If no rate has ever
been stored for a pair, the figures come back split by currency, unconverted,
with the reason.

## Imports

An import brings records in from outside. Fortuna has one contract that every
source implements, which is why adding a source does not disturb the others
([API-BR-30](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md)).

| Source | What it is |
| --- | --- |
| Manual entry | Records you type in. |
| Pluggy | Open-banking synchronization through a connection. Available only when the instance is configured for it and the network is reachable. |
| Excel workbook | A `.xlsx` file, with a mapping from its columns to transaction fields that you supply. |
| Nubank credit card invoice | A PDF. This is the only supported PDF layout today. |

An import is a job: it is accepted, queued and run in the background, and moves
through Pending, Running and then Completed or Failed. A completed job keeps the
outcome of every row.

- [API-BR-24](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): imported raw records are immutable. They are the evidence an
  import can be checked against.
- [API-BR-26](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): a record that matches an existing transaction on account, date,
  amount and, where the source gives one, the source's own identifier is not
  imported a second time.
- [API-BR-27](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): a job reports each row as imported, skipped as a duplicate, or
  rejected with a reason. One bad row never aborts the rows around it.
- [API-BR-28](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): Fortuna never stores, logs or transmits a bank credential.
  Open-banking access lives with Pluggy and is referenced by token only.
- [API-BR-29](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): revoking a connection stops future synchronization and leaves
  what was already imported in place.
- [UI-BR-28](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md): the interface uploads an import file whole and unmodified, and
  never parses or repairs it.
- [UI-BR-31](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md): the interface shows each row's outcome as the API reported it,
  never summarized into one success or failure.

A few details are worth knowing.

- An unrecognized PDF layout is rejected, naming the layouts that are supported,
  and nothing is imported.
- For the Nubank invoice, the summary (previous balance, payments, purchases,
  tax on international transactions and amount due) must reconcile, or nothing is
  imported. A PDF with no text layer fails, because Fortuna does no OCR.
- A failed Excel or Pluggy import commits none of its rows. This behaviour was
  fixed in API 2.0.0.
- A Pluggy connection needs your current external-data-processing consent, or
  the request is refused with `403`. A connection is Active, may need
  reauthentication, and can be revoked, which is permanent for synchronization.
- File imports and import retry are not available in desktop offline mode.

## Exports and attachments

### Exports

You can export a table query to CSV, Excel or PDF. A large result runs as a job
and you collect the file when it is ready, and the file is kept for a limited
time. An export contains only your records, leaves out soft-deleted ones, and
writes every amount as an exact decimal with its currency, formatted for the
locale you asked for.

- [API-BR-33](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): aggregations, exports and balances exclude soft-deleted records.
- [UI-BR-29](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md): the API produces an export and the interface only saves it. The
  client never renders a CSV, spreadsheet or PDF of its own.
- [UI-BR-30](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md): asynchronous work such as an import or an export never blocks the
  interface, survives navigating away, and its outcome stays findable.

Data-set export is not available in desktop offline mode.

### Attachments

An attachment is a receipt or document filed against a transaction. The
instance's operator sets limits on size and content type. The stored file is
kept when the transaction is soft-deleted and removed when it is hard-deleted.
The API supports attachments, but the interface does not yet: managing a
transaction's attachments (UC-35) is an open issue.

## Identity: connected and local accounts

In connected mode, Heimdall owns identity, credentials and recovery. Fortuna
forwards sign-in exchanges to Heimdall and validates the token it issues, and
it never keeps a password.

- [API-BR-35](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): in connected mode Fortuna never stores a password. It may carry
  one only in transit, to forward to Heimdall.
- [UI-BR-20](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md): the application authenticates only through the Fortuna API. The
  one exception is obtaining a Google ID token, which it hands to the API.

A desktop local account is the exception, because offline there is no network to
reach Heimdall over. It has no password reset and no email recovery. Recovery
codes are issued once at creation, can each be used once, and cannot be shown
again, so losing every code means losing the account.

- [API-BR-36](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): a desktop local account has no password reset and no email
  recovery, and its recovery codes are the only way back in.
- [API-BR-37](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): local-account data belongs to that installation and is never
  synchronized to a shared instance implicitly.
- [UI-BR-25](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md): desktop offline mode is available on Windows and Linux only, and
  only where the installation provides a local core. It is never offered on the
  web or on Android.
- [UI-BR-26](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md): outside desktop offline mode, a lost connection is reported as a
  lost connection and never disguised by a cache.

How the interface treats the session:

- [UI-BR-16](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md): the session token is kept only in the platform's secure storage.
- [UI-BR-17](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md): when the token expires or is rejected, the session ends and you are
  asked to sign in again. There is no silent refresh and no automatic replay of
  the interrupted action.
- [UI-BR-18](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md): signing out clears the token and all cached state.
- [UI-BR-19](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md): a two-factor challenge grants nothing until it is completed.

## How the interface treats figures

The interface is a client. It displays what the API says and decides nothing
about money.

- [UI-BR-01](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md): the API is the sole authority on every fact the interface shows.
- [UI-BR-02](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md): the interface never computes a balance, total, aggregation or
  projection. It requests them.
- [UI-BR-03](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md): client-side validation is feedback only. Every submission is sent,
  and every refusal is honored.
- [UI-BR-04](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md): an action is not reported as successful until the API confirms it.
- [UI-BR-05](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md): when the API refuses, the interface shows the API's reason.
- [UI-BR-13](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md): every aggregate the interface draws can be drilled into down to the
  records behind it, and the drill-down can be reversed.
- [UI-BR-14](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md): every drill-down level is requested from the API. The interface
  never subdivides an aggregate it already holds.
- [API-BR-34](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): any aggregate a client can drill into resolves, at its finest
  level, to the individual transactions that produced it.

## Deleting, restoring and the audit trail

Deleting a single record is two-stage. First it is soft-deleted: it drops out of
every balance, aggregation, projection and export, but you can still find and
restore it. Only a record that was soft-deleted can then be permanently removed,
and that is refused while any live record still references it. Deleting an
account soft-deletes its transactions with it, and an account cannot be
hard-deleted while any live transaction remains.

Erasing your whole account is a separate operation. It never goes through this
lifecycle. See [Consent and data rights](#consent-and-data-rights).

The audit trail records significant actions. It is append-only, and each entry
identifies its subject by an opaque reference rather than by name or address.

- [API-BR-38](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): deleting a record is two-stage, soft first and hard only from
  soft-deleted. Whole-account erasure is separate.
- [API-BR-39](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): a soft-deleted record is excluded from every balance,
  aggregation, projection and export, but stays retrievable and restorable.
- [API-BR-40](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): a hard delete is refused while any live record still references
  the target.
- [API-BR-41](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): audit entries are append-only.
- [API-BR-42](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): an audit entry identifies its subject by an opaque reference, and
  erasing a user destroys the mapping from that reference to the person.
- [UI-BR-12](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md): the interface shows a soft-deleted record only where you have asked
  to see deleted records, and marks it as deleted.

## Consent and data rights

Fortuna is built to satisfy the GDPR and the LGPD. Who counts as the data
controller depends on the deployment. In a self-hosted or desktop offline
installation you are your own controller, and on a shared instance the operator
is the controller. The interface reflects that ([UI-BR-38](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md)).

**Consent.** Before anything is handed to an external processor, such as the
Pluggy open-banking service, Fortuna needs your explicit decision for a
recognized purpose, recorded against the current version of the disclosure. A
decision applies only to the version you saw, so a revised disclosure needs a new
decision. Withdrawing consent revokes the dependent connections immediately and
keeps the history already imported. Importing from a file or entering data by
hand is never gated by consent.

- [API-BR-43](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): consent is an explicit decision naming a recognized purpose and
  the current disclosure version. Using a feature never implies it.
- [API-BR-44](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): a consent decision applies only to the exact disclosure version
  recorded with it.
- [API-BR-45](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md): withdrawing external-processing consent revokes dependent
  connections immediately and keeps the imported history. Local and file
  ingestion are never gated by it.
- [UI-BR-40](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md): the interface presents the disclosure and records your decision
  before any action that hands data to an external processor, and offers
  withdrawal wherever granting was offered.

**Taking your data.** You can ask for a complete export of your personal data.
It runs as a background job and produces a ZIP file with a manifest, versioned
JSON data, a schema for each kind of record, and your attachments as files. It
leaves out passwords, recovery codes, hashes and connection tokens, and it
expires. Only you can fetch it.

**Erasing your account.** Erasure needs an exact confirmation. It removes your
profile, local credentials, imported records and the rest of what you own in one
step, and it is irreversible. What survives is the audit trail, which no longer
identifies you.

- [UI-BR-41](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md): the interface offers a way to take all your data and a way to erase
  your account, each reachable without contacting anyone.
- [UI-BR-42](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md): the interface presents erasure as irreversible, confirms it
  explicitly, and states what survives.
- [UI-BR-39](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md): no data leaves the interface except to the Fortuna API. There is no
  analytics, crash reporting or telemetry.

**Logs.** The hosted API logs the start of every request with the caller's IP
address, which is personal data under the GDPR and the LGPD
([API-BR-46](https://github.com/artur-rios/fortuna-api/blob/develop/docs/initial/Business%20Rules.md)). The address goes to the console and the log files, and never to
the database, an audit entry or an export.
