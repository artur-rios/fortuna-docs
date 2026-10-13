---
title: Privacy and data rights
linkTitle: Data rights
weight: 130
description: >
  Seeing who controls your data, giving and withdrawing consent, taking a complete copy of everything, and erasing your account.
---

Fortuna is built to satisfy the GDPR and the LGPD. All of the controls for that live
on one screen, Privacy, reachable by both roles. The rules (what counts as consent,
what the personal-data archive contains, what erasure removes) are in
[Concepts]({{< relref "/docs/concepts#consent-and-data-rights" >}}).

The interface follows three rules here: consent comes before anything is sent to a
third party, your data can be exported and erased without contacting anyone, and
erasure is irreversible and is not the two-step deletion a single record goes
through ([UI-BR-40, UI-BR-41 and UI-BR-42](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md)).

| Screen | Address |
| --- | --- |
| Privacy | `/privacy` |

The Privacy screen starts with a card, **Who controls this data**. It depends on the
mode:

- In the *Connected* mode (the address came from the build), the card says "This is a
  shared instance. Whoever operates it is the data controller, and their privacy
  notice and rights process apply." and "Ask the instance's operator for their privacy
  notice and for how to make a rights request. Fortuna does not publish one on their
  behalf."
- In the *Self-hosted* and *Desktop offline* modes, it says "You run this instance
  yourself, so you are the data controller. There is no operator involved and no
  third-party privacy notice that applies." The screen decides this from the mode,
  not from who actually runs the instance you typed in.

## UC-42: Review, Give and Withdraw Consents

**Screen:** Privacy, `/privacy`, under **Consents**.

1. Each consent the instance asks for is a card with its name and its state: "Not
   given.", "Given on" a date with the version you agreed to, or, if the text has
   changed, "You agreed to version …, but version … is now in force."
2. For one you have not given, press **Give**. A dialog titled with the consent's name
   says "Giving this consent lets Fortuna send the data this feature needs to an
   external service on your behalf. Nothing is sent until you agree, and you can
   withdraw it at any time." and the version. Choose **I agree** or **Not now**. The
   decision is recorded with its version.
3. For one you have given, press **Withdraw**. The dialog "Withdraw this consent?"
   lists the connections that will be revoked ("These connections will be revoked:"),
   or says "Any connections that depend on this consent will be revoked." It adds "Data
   already imported stays where it is. Withdrawing stops anything further being sent;
   it does not delete what you already have." Choose **Withdraw** or **Keep it**.

Notable alternatives:

- **An outdated consent.** The card says "This needs a fresh decision. The old one no
  longer applies." and the button reads **Review again**. The old decision does not
  carry over.
- **Declining.** Nothing is recorded, and the feature the consent would have enabled
  stays unavailable. For example, connecting an institution asks for consent first;
  see [Imports and connections]({{< relref "/docs/user-guide/imports" >}}).
- **The consent list cannot be read.** The reason is shown with "Nothing that needs a
  consent will run until this can be read." and **Try again**. No feature proceeds on
  the assumption that a consent exists.
- **No consents.** "This instance asks for no consents."

## UC-43: Export All Personal Data

**Screen:** Privacy, `/privacy`, under **Take everything**. This is the portability
right, and it is not the ordinary [export]({{< relref "/docs/user-guide/exports" >}}) of
a view. The screen says so: "This is everything the instance holds about you, in a
machine-readable archive — the portability right. It is not the same as exporting a
data set from the spreadsheet or chart screens, which gives you only what you asked
for."

1. Press **Produce my archive**.
2. A progress bar runs, with "This runs on the instance. You can leave this screen and
   come back to it."
3. When it is ready, press **Save the archive**. The platform's own save dialog opens.
4. The screen then says "Saved to" the place and the size in bytes.

Notable alternatives:

- **The job fails.** The reason is shown with **Try again**.
- **The archive expired** before you saved it. The screen says "That archive expired
  before it was downloaded. Nothing is wrong with your data — the copy was just
  cleaned up." with **Produce a new one**.
- **The save location is refused.** The reason is shown, with "Your archive is still
  ready. Choose somewhere else." and **Choose a location**.
- **This works offline.** The native core builds the archive itself, so this control
  is not marked "Not available offline". See
  [Offline mode]({{< relref "/docs/user-guide/offline-mode" >}}).

## UC-44: Erase the Account

**Screen:** Privacy, `/privacy`, under **Erase this account**. This destroys the
account and everything it owns, and it cannot be undone.

1. The section says "This destroys the account and everything in it. It cannot be
   undone." Press **Erase my account**.
2. The screen warns, before anything is confirmed: "This is irreversible. It is not the
   deletion you can restore a record from — nothing survives it and nobody can put it
   back, including whoever runs this instance. The audit trail is kept, as the law
   requires, but it will no longer identify you."
3. If you have live connections, it lists them: "These connections will be revoked as
   part of it:".
4. It then says "Take a copy first if you want one. After this there is nothing left to
   export." (see UC-43).
5. Type the word **ERASE** exactly in the box ("Type ERASE to confirm."). The button
   **Erase everything, permanently** stays disabled until you have.
6. Press it. The screen reports what was erased, as a count for each kind of record and
   the number of connections revoked, with a **Close** button. Pressing **Close** ends
   the session, clears everything stored locally, and returns you to sign-in.

Notable alternatives:

- **Keep my account** at the confirmation cancels with nothing erased.
- **Refused or failed.** The API's reason is shown followed by "Nothing was erased and
  your account is intact." The API applies erasure as one operation, so you never end
  up with a half-erased account.
- If the session ends during erasure, the outcome is whatever the API decided: on
  signing in again you find either an erased account or an intact one.

## Sources

This page is written from the interface's
[Use Case Specification](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/requirements/Use%20Case%20Specification%20Document.md)
(UC-42, UC-43, UC-44), checked against the code in
[`lib/features/privacy`](https://github.com/artur-rios/fortuna-ui/tree/develop/lib/features/privacy).
