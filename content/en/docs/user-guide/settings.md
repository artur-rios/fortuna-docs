---
title: Settings, two-factor and instance health
linkTitle: Settings and health
weight: 20
description: >
  Choosing theme, language and display currency, managing two-factor authentication, and the administrator's instance health screen.
---

This page covers three screens that are about the account and the instance rather
than about money: Settings, which both roles use; two-factor management, which is
written but not reachable; and Instance health, which only the instance
administrator can open.

| Screen | Address | Who |
| --- | --- | --- |
| Settings | `/settings` | Both roles |
| Two-factor authentication | none | No route opens it |
| Instance health | `/admin` | Instance administrator only |

## UC-13: Choose Theme, Locale and Display Currency

**Screen:** Settings, `/settings`. There is a sign-out button in its top bar.

Everything on the screen applies as soon as you choose it.

1. **Appearance** has three options: "Match the system", "Light" and "Dark".
2. **Language and formatting** has four locales: "English (United States)"
   (`en-US`), "English (United Kingdom)" (`en-GB`), "English (Europe)" (`en-150`)
   and "Português (Brasil)" (`pt-BR`). Numbers, dates and amounts reformat, and
   amounts you type are read in the chosen locale.
3. **Display currency** has "Each in its own currency" and one entry per currency
   the API supports, shown as "CODE — Name". The screen explains: "Figures are
   converted by the instance, which records the rate it used. Anything it cannot
   convert is shown in its own currency." The conversion is the API's, never the
   interface's. For what a display currency does and how rates are chosen, see
   [Currencies and PTAX]({{< relref "/docs/concepts#currencies-and-ptax" >}}).
4. **Preview** shows a sample amount in US dollars and a projected amount in
   Brazilian reais, so you can see how figures and the "projected" marking read
   in your choices.

The choices are stored in the device's ordinary preference storage, which never
holds a token or a credential.

Notable alternatives:

- **No locale chosen yet.** The platform's locale is used if it is one of the four,
  otherwise `en-US`.
- **The currency list cannot be loaded.** A line says so with a **Retry** button,
  and either "Your choice of … still applies." or "Figures continue to show in
  their own currencies."
- **Preferences cannot be stored.** A notice at the top says "These choices apply
  now but could not be saved on this device, so they will not be remembered next
  time."
- **A figure the API cannot convert** is shown in its own currency instead of
  being converted by the interface. The charts label such an amount "Not
  converted:".

## UC-10: Manage Two-Factor Authentication

**Not reachable.** The screen for this use case is written and has tests, and the
UI README marks the use case done. But `lib/app/routes.dart` has no route for it
and no other screen links to it, so a person using the application cannot open it
in this version. The sign-in side, where a challenge would be answered, is the
unfinished UC-04; see [Signing in]({{< relref "/docs/user-guide/signing-in#uc-04-complete-a-two-factor-challenge" >}}).

What the screen does, as written (title "Two-factor authentication"):

1. It states whether two-factor is on and by which method: "Two-factor
   authentication is off." or "Two-factor authentication is on, using …", with
   how many recovery codes are left.
2. When it is off, you tick **Authenticator app**, **Email**, or both, and press
   **Set up two-factor authentication**.
3. For an authenticator, it shows a scannable code (**Scan this**) and the same
   secret as text (**Or enter this secret by hand**, with a copy button), so
   someone who cannot scan can still proceed. You enter the first valid code,
   from your authenticator or from the email, and press **Confirm**.
4. **Keep these recovery codes** then shows the codes once, with a checkbox "I
   have saved these codes somewhere safe" and **Done**. Leaving before you tick it
   asks "Leave without keeping your codes?"; the codes are not shown again.
5. When it is on, the screen asks for **Your password** and **A current
   second-factor code**. **Replace my recovery codes** issues a new set, shown once.
   **Turn two-factor authentication off** needs both.

Alternatives: a wrong or expired confirmation code leaves the setup pending so you
can try again, and **Cancel this setup** leaves two-factor off. A wrong password or
code when turning it off gets the API's message, the same for both. An account that
signs in with Google cannot hold a second factor, and the API's reason is shown.

## UC-45: View Instance Health

**Screen:** Instance health, `/admin`. Only the instance administrator can open it.
An account owner who asks for the address is sent to `/` without being told it
exists. There is no financial figure, record or aggregate anywhere in this area,
and an administrator cannot reach the financial screens.

1. After sign-in an administrator lands here.
2. The application asks the instance for its detailed health.
3. A card headed "Instance" shows the overall status: **Healthy**, **Degraded**,
   **Unhealthy** or **Not configured**. If the instance is troubled, the card adds
   "Caused by" and the dependency names.
4. Below it, one row per dependency shows its name and status. A job queue row adds
   its "queue depth" and "oldest pending" seconds.
5. The refresh icon in the top bar asks again.

Notable alternatives:

- A dependency that is not set up in this deployment shows **Not configured**,
  which is distinct from unhealthy.
- A status the interface does not recognize is shown as the API named it.
- If the instance reports no dependencies, the screen says "The instance reported
  no dependencies."
- If the request fails, the message "The instance health could not be read" (or
  the API's reason) appears with a **Retry** button, and no partial view.

The API behind this screen is its detailed health route; see the
[API Operations & Infrastructure Document](https://github.com/artur-rios/fortuna-api/blob/develop/docs/requirements/Operations%20%26%20Infrastructure%20Document.md)
for its statuses.

## Sources

This page is written from the interface's
[Use Case Specification](https://github.com/artur-rios/fortuna-ui/blob/develop/docs/requirements/Use%20Case%20Specification%20Document.md)
(UC-10, UC-13, UC-45), checked against the code in
[`lib/features/preferences`](https://github.com/artur-rios/fortuna-ui/tree/develop/lib/features/preferences),
[`lib/features/auth/ui/two_factor_screen.dart`](https://github.com/artur-rios/fortuna-ui/blob/develop/lib/features/auth/ui/two_factor_screen.dart),
[`lib/features/administration`](https://github.com/artur-rios/fortuna-ui/tree/develop/lib/features/administration)
and the [route table](https://github.com/artur-rios/fortuna-ui/blob/develop/lib/app/routes.dart).
