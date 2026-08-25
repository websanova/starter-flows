# Subscription Create - Stripe (Payment Element, intent on init)

Status: draft
Updated: 2026-08-25

## Purpose & Scope

Creating a subscription with the Payment Element, where the intent gets created before the element mounts. The element is driven straight off a real client secret, so there is no amount to keep in sync. Trials fall out for free - the server decides whether it is a payment or a setup and just tells the client which.

The tradeoff is that a subscription gets opened on Stripe for anyone who so much as lands on the page, and anything that changes the amount afterwards, a promo code or a billing address that changes the tax, means tearing it down and building a new one.

The billing address is captured before the element mounts either way, and a promo code too when codes are on. The intent cannot be created until every input to the amount is known, and once it is created the first invoice is finalized and its amount does not change, so neither can be applied after the fact. Re-pointing the mounted element at a new secret is not an option either, `clientSecret` is fixed when `elements()` is created. That also means letting the user go back and change the address or promo code costs a fresh intent and a fresh mount, which wipes the details they typed.

## Actors & Entities

Actors

- User - enters payment details in the Payment Element on your page.
- Client App - requests the intent, mounts the element, confirms, polls after success.
- API - creates the customer and the subscription, writes the local row, receives the webhook.
- Stripe - issues the intent, runs 3DS, settles the charge, fires the webhook.

Entities

- User record - holds the Stripe customer id and the billing address that gets pushed up.
- Stripe Customer - must carry a validated billing address, tax on or off.
- Subscription - created at `incomplete` (or `trialing`) before any payment.
- Invoice - first invoice, finalized at creation. `$0` on a trial.
- PaymentIntent - on the invoice when there is no trial.
- SetupIntent - on `pending_setup_intent` when there is a trial.
- Local subscription row - written at `incomplete` as soon as the Stripe id exists, before payment.

## Flow

1. On page load, collect the billing address first, always, and the promo code too if codes are in play. The intent cannot be created until every input to the amount is known.
2. Hit the API for the intent, sending `{ plan, interval, promo_code? }`.
   1. Load the user's Stripe customer id from your DB.
   2. If there isn't one, create the customer on Stripe. Save the returned customer id to your users table.
   3. Push the billing address to the Stripe customer with `tax[validate_location]` set to `immediately`, whether or not tax is enabled. Has to be done before the intent is created, see Decisions and the [address update flow](../../billing/stripe/address-update.md) for details.
   4. Work out trial eligibility on the API side, since it already knows whether this user has burned a trial before.
   5. If a promo code came through it gets resolved into a Stripe promo code object. The field is optional, no code means this step is skipped entirely.
   6. Create the Subscription on Stripe with customer (stripe id), the plan's price id, `discounts: [{ promotion_code: 'promo_xxx' }]` if there was a code, `trial_period_days` if eligible, `payment_behavior: 'default_incomplete'`, `automatic_tax: { enabled: true }`, `expand: ['latest_invoice.confirmation_secret', 'pending_setup_intent']`. `confirmation_secret` is not expanded by default, it has to be named explicitly or it comes back absent.
   7. That single call creates the Subscription on Stripe at status `incomplete` (or `trialing`). It also creates an intent. Whether the user gets a trial decides which kind of intent, and the two kinds are not interchangeable. They come back on different fields, the client secret is read from a different key on each, and the front end has to call a different confirm method for each.
      - No trial - there is a real amount to charge, so Stripe creates a first Invoice for the full amount and a PaymentIntent against it. The client secret is read off `latest_invoice.confirmation_secret.client_secret`, not off the PaymentIntent itself. It starts with `pi_`. The front end confirms with `confirmPayment()`.
      - Trial - the first invoice is `$0`, so there is nothing to charge and no PaymentIntent. Stripe creates a SetupIntent instead, which stores the payment method for when the trial ends. It comes back on the subscription's `pending_setup_intent` field and the client secret is read off `pending_setup_intent.client_secret`. It starts with `seti_`. The front end confirms with `confirmSetup()`.
   8. The amount is computed by Stripe from price + tax. You never send one, and nothing on the client has to match it.
   9. If it errors out, that error needs to get sent back to the client for display.
   10. On success, write your local subscription row now that the Stripe id exists - stripe sub id, plan, interval, status.
   11. Return the client secret to the client app, along with a `type` of `payment` or `setup` saying which of the two it came from. The `type` is a field we set ourselves, Stripe does not return it, and it is a convenience only. The front end can derive the same thing from the secret's prefix, `pi_` or `seti_`, and step 7 does exactly that when the user comes back from 3DS onto a cold page.
3. Element mounts against that secret with `elements({ clientSecret })`. No mode, no amount, no currency, and no trial handling, Stripe reads all of that off the intent. The `type` is not used here at all, only at confirm.
   1. Load stripe.js if it isn't already on the page.
   2. `elements({ clientSecret })` builds the Elements object locally. No network calls here.
   3. `paymentElement.mount(target)` creates the iframe.
4. User hits subscribe.
5. Branch on the `type` from above - `confirmSetup()` for a trial, `confirmPayment()` otherwise. Both take `{ elements, clientSecret, confirmParams: { return_url }, redirect: 'if_required' }`. The `return_url` is mandatory.
6. Response is success / error / 3DS. 3DS either runs in a dialog and resolves inline, or sends the browser away to the bank and back to your return url. Either way you end up at the same place - a settled intent.
7. If it redirected, the user comes back to a freshly loaded page with no state. Stripe appends the secret to the return url, and which key it appends also tells you the type, so read the pair together and call `retrievePaymentIntent` or `retrieveSetupIntent` to see how it landed rather than starting the flow over. Same mount path as the initial one since it is a client secret either way, so there is only one mode to support.
8. On success, your API still knows nothing. Nothing in the chain above told it the payment landed, only the webhook does.
9. Stripe fires the webhook. Your backend looks up the local row by Stripe sub id and flips it to `active`, or `trialing` if there was a trial. This is asynchronous and has no fixed timing, it can land before confirm even resolves in the browser, or seconds after.
10. Reload the auth user and check for the subscription. Poll this, a hit means the webhook arrived and the API picked it up. Give up after a ceiling rather than spinning forever.
11. Take the success action - redirect to billing, a success page, wherever.

## Diagram

```mermaid
flowchart LR
    A[Page load] --> B1["Collect address always,<br/>promo code if on"]
    B1 --> C

    C["POST /subscription/intent<br/>{plan, interval, promo_code?}"] --> D["Load or create Stripe customer<br/>push address, validate_location"]
    D --> E["Resolve promo code<br/>(if sent)"]
    E --> F["subscriptions.create<br/>default_incomplete"]
    F --> G{Trial?}

    G -->|no| H1["PaymentIntent on invoice<br/>type: payment"]
    G -->|yes| H2["SetupIntent on pending_setup_intent<br/>type: setup"]

    H1 --> I["Write local row<br/>status: incomplete"]
    H2 --> I
    I --> J[Return client_secret + type]

    J --> K["elements({ clientSecret })<br/>mount element"]
    K --> L[User hits subscribe]
    L --> M{type}

    M -->|payment| N1[confirmPayment]
    M -->|setup| N2[confirmSetup]

    N1 --> O{Result}
    N2 --> O

    O -->|declined| L
    O -->|3DS redirect| O1["Back at return_url<br/>retrieve intent"]
    O -->|success| P
    O1 --> P

    P[Stripe fires webhook] --> Q["Local row -> active / trialing"]
    Q --> R[Client polls auth user]
    R --> S[Success action]
```

## States

Local subscription row.

| State | Meaning |
| ----- | ------- |
| incomplete | Written at intent creation. Subscription exists on Stripe, nothing charged, payment method may never be entered. |
| trialing | Webhook landed, trial running, payment method stored via SetupIntent. Counts as subscribed. |
| active | Webhook landed, first invoice paid. |

Allowed transitions

| From | To | Trigger |
| ---- | -- | ------- |
| none | incomplete | Subscription created on Stripe at `default_incomplete` |
| incomplete | active | Webhook, payment settled |
| incomplete | trialing | Webhook, trial was eligible |
| incomplete | incomplete | Confirm declined. Same secret stays confirmable, user retries |
| incomplete | expired | User abandons. Stripe expires the subscription after 23 hours, local row needs cleaning up |

Unlike hosted and embedded, a local row exists before any payment. That row is the cleanup problem.

## Rules

- Authenticated user required.
- Trial eligibility is decided on the API side only. The client never asserts it - it is told the `type` to confirm with.
- The Stripe customer must carry a validated billing address before the subscription is created, tax on or off. Pushed with `tax[validate_location]` set to `immediately`, see the [address update flow](../../billing/stripe/address-update.md).
- The address is always captured before the element mounts, and the promo code too when codes are on. Nothing can be applied after.
- One in-flight subscription per user, plan and interval. Revisiting the page must hand back the existing incomplete subscription rather than opening another.
- The subscribed flag must treat `trialing` as subscribed, otherwise a trial signup polls forever.
- Only the webhook flips the row to active or trialing. A resolved confirm is not proof.
- Polling has a ceiling. Past it, show a pending state rather than spinning.

## Edge & Error Cases

| Case | Cause | Expected behavior |
| ---- | ----- | ----------------- |
| Promo code entered after mount | First invoice is already finalized. A discount applied now only affects future invoices | Cancel the subscription on Stripe, create a new one with the promo attached, mount a fresh element against the new secret. The typed details are lost |
| Billing address changes the tax after init | Same as above, the amount is locked once the intent exists | Same teardown and remount. The typed details are lost |
| Abandoned page | Subscription opened for anyone who lands | Incomplete subscription on Stripe and an incomplete row in your DB. Stripe expires its side after 23 hours, your rows need a cleanup job |
| Revisit while incomplete | User comes back to the page | Hand back the in flight subscription for the same plan and interval rather than opening a second one |
| Payment method declined | Bank refused | Element stays mounted against the same secret, user corrects the details and submits again. The intent is still confirmable, no new secret needed |
| 3DS sends the browser away | Bank requires a challenge page | User returns to a cold page with no state. Stripe appends the secret to the return url, and the key name tells you the type. Retrieve the intent rather than restarting the flow |
| Trial hits 3DS | Bank wants the payment method verified even though nothing is charged | Handle 3DS on the SetupIntent path too, not just payment |
| Address resolves to no tax jurisdiction | Stripe cannot place it | The customer update errors before the intent is created. Nothing to tear down, the user corrects and retries |
| Tax not computable | Missing registration or no product tax code | Error back to the client for display. Subscription create fails |
| Invalid promo code | Code does not resolve to a Stripe promo object | Error back to the client for display |
| 100% promo zeroes the invoice | Nothing to charge, so there is no intent and no secret to return | Not worked out yet. Also affects a trial, where a `once` code is consumed by the `$0` trial invoice. See TODO |
| Webhook lands late | Asynchronous, can arrive before confirm resolves | Poll the auth user, show pending until the flag flips |
| Webhook never arrives | API dropped or failed the attempts | User sits in a pending state with an incomplete row. Needs a manual sync command to reconcile against Stripe |
| Trial signup polls forever | Subscribed flag ignores `trialing` | Flag must count `trialing` as subscribed |

## Decisions

Chose the Payment Element with the intent created on init. Four strategies were evaluated - hosted Checkout, embedded Checkout, Payment Element with the intent on init, and Payment Element with a deferred intent. The three not taken are kept in `reference/`.

On-init over deferred - the element mounts against a real client secret, so there is no amount, currency or mode to keep in sync and no `IntegrationError` class of failure at confirm. The server decides trial eligibility and tells the client which confirm to call, rather than the client guessing it up front. The 3DS cold return also uses the same mount path as the initial one, so there is only one mode to support instead of two.

On-init over hosted and embedded - the payment UI is the Payment Element on your own page, styleable with the Appearance API. Checkout gives you Dashboard branding and nothing more.

Cost of the choice: a subscription is opened on Stripe for every visitor to the page, which needs a cleanup job on your side. Tax and promo codes have to be collected before the element mounts, and changing either afterwards means a full teardown that wipes the details the user typed.

The billing address is collected and validated on every subscribe, tax on or off. Collecting it only when tax is on saves a step in the subscription flow but ends up with all the users having missing or unvalidated addresses for tax purposes. This can lead to headaches and tax liability, and takes some work to then backfill the enforcement. This must be done before the intent is created. An address pushed afterwards won't work, it's either missing from the intent calculation when tax is on or it fails after the subscribe already went through. That leaves `automatic_tax` as a backend flag, it decides what goes on the subscription create and nothing else.

## TODO

Now

- Tax and promo codes are each a configurable on/off field. The API owns both and is the source of truth - the client is told what is on, it never decides. The address step runs either way, so what is left to decide is:
  - Promo off - the address is collected, pushed, and the element mounts behind it.
  - Promo on - a code field is shown alongside the address and resolved before the subscription is created.
- Decide how the client learns the two flags (config payload at page load vs baked into the plan response).
- Decide how subscribe failures get diagnosed. When Stripe rejects the create (tax misconfigured, bad address, invalid promo), the API returns a generic "provider unavailable" and the real reason is only attached when app.debug is on. In production it is discarded, so a user reports a failed subscribe and there is nothing to go on.
- Lay out how the address and promo code get collected before mount.

Later

- Cleanup job for abandoned incomplete rows.
- Manual sync command to reconcile against Stripe when a webhook is dropped.
- Polling ceiling value and what the pending state looks like.
- Work out the 100% promo case - what the API returns when there is no secret, and whether `once` codes are allowed alongside a trial.

Out of scope

- Cancel, resume, plan change.
- Renewal failures and dunning.
