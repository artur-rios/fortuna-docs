---
title: Signing in
linkTitle: Signing in
weight: 10
description: >
  Choosing an instance, signing in with a password or Google, signing up, recovering a password, and how a session starts and ends.
---

This page covers how you get into Fortuna and out again. Online, identity belongs
to Heimdall, and the interface reaches it through Fortuna's own `/api/auth/*`
routes; see [Signing in through Heimdall]({{< relref "/docs/architecture#signing-in-through-heimdall" >}}).
The rules behind session handling are in
[Concepts]({{< relref "/docs/concepts#identity-connected-and-local-accounts" >}}).
The desktop local account used in offline mode is on the
[Offline mode]({{< relref "/docs/user-guide/offline-mode" >}}) page.

Two things to know first. The interface never keeps a password: a credential is
sent once and cleared from the form, and a failed attempt is retyped, not
replayed. And a rejection is shown in the API's own words, the same for an
unknown account and a wrong password, so the screen cannot be used to find out
who is registered.

| Screen | Address |
| --- | --- |
| Set up your instance | `/setup` |
| Sign in | `/sign-in` |
| Create an account | `/sign-up` |
| Two-factor challenge (placeholder) | `/sign-in/two-factor` |
| Recover your password | `/password-recovery` |
| Choose a new password | `/password-reset?token=…` |
| Verify your email address | `/verify-email?token=…` |

## UC-01: Configure the Instance and Mode

The first time the application starts it needs to know which Fortuna instance to
talk to. The setup screen, "Set up your instance", is the only screen reachable
until that is settled.

1. If the build already carries an instance address (a build-time setting; see
   [Installation]({{< relref "/docs/installation" >}})), the application uses it
   and goes on to sign-in. An address you saved on this device earlier replaces it.
2. Otherwise the screen says "Fortuna keeps your data on the instance you name.
   Enter its address to continue." and offers one field, "Instance address"
   (the hint reads `https://fortuna.example`), and a **Connect** button.
3. The application checks that you typed a full `http://` or `https://` address.
   If not, it says "Enter a full address, including http:// or https:// — for
   example https://fortuna.example." and sends nothing.
4. It asks the instance who it is. If the instance cannot be reached, it says so,
   keeps you on the screen and does not save the address. If the instance speaks a
   different contract version than the application was built for, it refuses to
   continue and names both versions: "That instance speaks contract … and this
   application was built for …. Update whichever is older before continuing."
5. On success the address is saved, the matching transport is chosen, and the
   sign-in screen follows.

**Desktop offline mode.** On Windows and Linux, and only when the core library
loads, the screen also offers **Use this device only** under the words "Your data
stays on this computer. No instance, no network." It is not offered on the web or
on Android, nor on a desktop installation without the core library. If the core
library is present but fails to load, the screen says "This installation carries
the Fortuna core but it could not be loaded, so offline mode is unavailable.
Connect to an instance instead." and the connected path stays available.

The three modes are *Connected* (the address came from the build), *Self-hosted*
(you typed the address on this screen) and *Desktop offline*. The difference you
can see is on the Privacy screen, which names the data controller differently
for each; see [Privacy and data rights]({{< relref "/docs/user-guide/data-rights" >}}).

## UC-03: Sign In with Credentials

**Screen:** Sign in, `/sign-in`. Available in the connected and self-hosted modes.

1. Enter your **Email address** and **Password**. The eye icon shows or hides the
   password.
2. Press **Sign in**. If the address is not plausible, the form says "Enter the
   email address for your account."; if the password is empty, "Enter your
   password." Nothing is sent in either case.
3. The application sends the credentials to the API. On success it stores the
   session token in the platform's secure storage, clears the fields, and the
   guard takes you to the screen your role admits: the address you were heading
   for if one was remembered, otherwise `/` for an account owner or `/admin` for
   an administrator.

Notable alternatives:

- **Rejected credentials.** The API's single message is shown, the same for an
  unknown account and a wrong password.
- **The instance cannot be reached.** The message appears with a **Try again**
  button. The password is not kept for the retry, so you type it again.
- **The address is not verified.** The API's reason is shown with a
  **Resend the verification email** button that opens the verification screen
  (see UC-09).
- **Two-factor is active.** The API answers with a challenge instead of a token.
  That continues in UC-04, which is not built.
- Leaving the screen clears the fields.

The same screen carries three more links: **Continue with Google** (UC-05, only
when Google sign-in is configured), **Forgotten your password?** (UC-09) and
**New to Fortuna? Create an account** (UC-47, hidden in offline mode).

## UC-04: Complete a Two-Factor Challenge

**Not built.** This use case is open as issue
[#5](https://github.com/artur-rios/fortuna-ui/issues/5), and the code agrees.

What exists is the route `/sign-in/two-factor`, which is guarded and shows a
placeholder: a screen titled "Two-factor challenge" with the text
"/sign-in/two-factor is routed and guarded. Its screen is UC-04." Nothing in the
application opens that route.

What happens today when you sign in to an account that has two-factor active:

- The API answers with a challenge. The application holds it and treats you as
  signed out, because a challenge grants nothing
  ([UI-BR-19](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md)).
- The sign-in screen stays as it was. No screen asks for the second factor.

So an account with two-factor active cannot finish signing in through the
interface in this version. The API side works; see
[Signing in through Heimdall]({{< relref "/docs/architecture#signing-in-through-heimdall" >}}).
The specification describes the intended screen, where you would supply an
authenticator code, an emailed code or a recovery code, and could ask for a fresh
emailed code. None of that can be used yet.

## UC-05: Sign In with Google

**Screen:** Sign in. The **Continue with Google** button appears only when the
build carries a Google client identifier (`FORTUNA_GOOGLE_CLIENT_ID`; see
[Installation]({{< relref "/docs/installation" >}})). Where it is absent the
option is not offered at all.

1. Press **Continue with Google**. The platform's own Google sign-in opens and
   returns an ID token.
2. The application sends that token to the API, and to no other identity service.
3. The API creates the account if this is a first Google sign-in, and answers with
   a session token, which is stored as in UC-03.

Cancelling the Google sheet returns you to the sign-in form without an error. If
Google returns no usable token, the screen says so and the password path is still
there. If the API refuses the token, for example because the address already has
a password account it will not link, the API's reason is shown.

## UC-47: Sign Up

**Release state:** this use case is on the `develop` branch and listed as
Unreleased in the UI changelog. It is not in the v1.0.0 release.

**Screens:** Create an account, `/sign-up`, then Verify your email address,
`/verify-email?token=…`. Not offered in offline mode, where the local account has
its own creation path.

1. On the sign-in screen press **New to Fortuna? Create an account**.
2. Enter a **Name** (at most 200 characters), an **Email address** (at most 256)
   and a **Password** (at least 8 characters; the field says "At least 8
   characters."), then press **Create account**. A **Continue with Google**
   button appears where Google sign-in is configured, and runs UC-05.
3. The application checks the input against those limits, with the message under
   the field concerned, then sends it. When the API accepts the request, the form
   is replaced by the API's message and a **Back to sign in** button. **No
   account exists yet.**
4. Open the link in the email. It lands on the verification screen, which sends
   the token by itself. The API answers that the sign-up needs the password you
   chose, and the screen asks for it.
5. Enter the same password and press **Create my account**. The account is
   created, the API's message is shown, and a **Sign in** button leads to UC-03.

Notable alternatives:

- **An address that already has an account** gets the same answer as a new one,
  worded the same; the account's holder is emailed by the instance.
- **A wrong confirmation password** is shown under the password field, and you
  can try again.
- **A bad, expired or used link** shows the API's message followed by "If this link
  came from signing up, sign up again — or sign in if you already finished.", with
  **Sign in** and **Sign up again** buttons. The hint appears only for that case.
- **The instance cannot be reached while you confirm** keeps the password form on
  screen with the reason under the field. The link is still good; tap
  **Create my account** again. Nothing is resubmitted without your action.
- **A plain verification link** (an address change rather than a sign-up) needs no
  password: the token is accepted, the API's message is shown, and **Sign in**
  follows.
- A wrong password on an anonymous screen does not show "session expired".

## UC-09: Recover a Password and Verify an Address

**Screens:** Recover your password, `/password-recovery`; Choose a new password,
`/password-reset?token=…`; Verify your email address, `/verify-email?token=…`.
Not available in offline mode, where a local account has no password reset.

1. On the sign-in screen press **Forgotten your password?**.
2. The screen says "Enter the address for your account and we will send a reset
   link." Type the **Email address** and press **Send a reset link**.
3. The API's confirmation is shown unchanged. It is the same whether or not the
   address belongs to anyone, so the screen cannot be used to discover who is
   registered.
4. Open the link in the email. It lands on "Choose a new password".
5. Type the **New password** and **Confirm it**, then press **Reset password**.
   If the two differ, the form rejects them. On success you go back to sign-in.
6. Verification follows the same shape. The link opens "Verify your email
   address", the token is sent at once and the outcome is shown with a **Sign in**
   button. If you arrive without a link, from **Resend the verification email** on
   the sign-in screen, the only action is **Send the verification email again**.

Notable alternatives:

- **A dead token.** The API's reason is shown, with **Request a new link** on the
  reset screen. A refused password cannot be fixed by another link, so nothing is
  offered for that; the API's reason, including any rule the form did not check,
  is shown.
- **Asking too soon.** The API's rate-limit answer is shown as an answer, not as a
  failure.

## UC-11: Restore a Session at Start

When the application starts it reads the stored session token from the platform's
secure storage, the only place a token is ever written, and checks it with the API
before showing any screen that depends on it.

- **No token** is the ordinary first-run case. You go to sign-in.
- **A good token** puts you on the screen your role admits (or the remembered
  destination).
- **The instance cannot be reached.** A screen titled "Could not restore your
  session" shows the reason with a **Retry** button. The token is kept, because it
  may still be valid, and you are not admitted on an unverified token.
- **Secure storage is unavailable**, for example on the web with site data
  blocked: "This device cannot store a session securely, so a previous one could
  not be restored. Please sign in." There is no retry, since it would fail
  identically.
- **The API rejects the token**, or the stored token cannot be read: it is
  discarded and you sign in again, with "The stored session could not be read.
  Please sign in again." for an unreadable one.
- **Offline mode.** The core cannot verify a token at all, so a stored session is
  discarded with the core's reason, and you sign in again.

The token is attached to requests that need it and is never put in an address
([UI-BR-16](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md)).

## UC-12: End a Session

**Control:** an icon button with the tooltip "Sign out" in a screen's top bar.
Only some screens have it: Settings, Categories, Tags and counterparties, Imports,
Import a file, Connections, Audit trail, Instance health, and the placeholder
screens including the home screen. Every other screen, among them accounts, cards,
investments, transactions, budgets, goals, charts, projections and privacy, has
no sign-out button. To sign out from one of those, first open a screen from the
list.

1. Press **Sign out**.
2. The application clears the token from secure storage and every cached value
   and view state, so nothing about you survives into the next session
   ([UI-BR-18](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md)).
3. The guard sends you to sign-in.

Notable alternatives:

- **Clearing storage fails.** A message appears at the bottom of the screen and you
  stay signed in, rather than being shown the sign-in screen while the token is
  still on the device.
- **A token rejected mid-session** ends the session the same way. There is no
  silent refresh, and an action in flight is not replayed after you sign in again
  ([UI-BR-17](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/initial/Business%20Rules.md)).
- A running import or synchronization belongs to the API and carries on there.
- Erasing your account (see
  [Privacy and data rights]({{< relref "/docs/user-guide/data-rights" >}})) also
  ends the session.

## Sources

This page is written from the interface's
[Use Case Specification](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/requirements/Use%20Case%20Specification%20Document.md)
(UC-01, 03, 04, 05, 09, 11, 12, 47), checked against the code in
[`lib/features/setup`](https://github.com/artur-rios/fortuna-ui/tree/develop/lib/features/setup),
[`lib/features/auth`](https://github.com/artur-rios/fortuna-ui/tree/develop/lib/features/auth),
[`lib/features/session`](https://github.com/artur-rios/fortuna-ui/tree/develop/lib/features/session)
and the [router](https://github.com/artur-rios/fortuna-ui/blob/develop/lib/app/router.dart),
and against the UI [changelog](https://github.com/artur-rios/fortuna-ui/blob/develop/CHANGELOG.md).
