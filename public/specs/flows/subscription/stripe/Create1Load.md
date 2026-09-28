# Subscription Create Load - Stripe (Checkout Sessions, Payment Element)

Status: done
Updated: 2026-09-28

## Description

A User with no Stripe Subscription opens the subscribe page and the API hands out a Stripe Checkout Session for them to work against. The App initialises against it and opens on the address step, or straight on the confirm step when a Stripe Payment Method is already on file, and everything the page shows is read off the Stripe Checkout Session.

## Terms

| Term | Description |
| --- | --- |
| API | The back end. Holds the API records and talks to the providers. |
| API Subscription | The API record mirroring the Stripe Subscription. Stripe id, plan, interval, status. |
| API User | The User's record on the API side. |
| App | The front end the User is looking at, web or mobile. |
| App Storage | Short lived storage in the App that survives a redirect away and back. |
| Auth User | The signed in User's data held by the App. |
| Stripe Billing Address Element | The Stripe Element collecting the billing address and name. |
| Stripe Checkout Session | The Stripe object a checkout runs on. Carries the line items, the address, the promotion code, the total and the Stripe Payment Method. |
| Stripe Customer | The Stripe object holding the User's id, address and saved Stripe Payment Methods. |
| Stripe Element | A Stripe UI component mounted by the App. |
| Stripe Payment Element | The Stripe Element collecting the Stripe Payment Method. |
| Stripe Payment Method | The payment method at Stripe, saved against the Stripe Customer. |
| Stripe Subscription | The subscription at Stripe. |
| User | The human using the App. Never the App and never the API. |

## Requirements

- Create an initial Stripe Subscription, either for a new User or for a User whose previous Stripe Subscription has ended.
- Authenticated Users only.
- Three step wizard, the billing address and name first, required for tax purposes, then the payment method, then the confirm.
- A User with a Stripe Payment Method already on file opens on the confirm step. The address and the payment method are both already held, so there is nothing to collect and no Stripe Element on screen.
- Use the Stripe Payment Element in the App against a Stripe Checkout Session created by the API.
- Collect the address with the Stripe Billing Address Element, so field layout and country rules come from Stripe.
- Style both Stripe Elements with the App's own appearance so the page matches the rest of the App.
- Show the total, tax and discount as Stripe calculates them. Nothing is priced in the App.
- Start a trial when the User is eligible, with a Stripe Payment Method collected up front. Eligibility is decided by the API.
- Refuse anyone who already has a Stripe Subscription. Past due and unpaid refuse with their own error, and default to billing until the App handles them. See the [subscription guards flow](../Guards.md).
- The billing page shows the subscribe control only to a User with no Stripe Subscription, and the route is guarded on the same. The API decides it again on the create.
- Only one Stripe Checkout Session at a time per User. Opening the subscribe page cancels any the User already has open, so a page left sitting in another tab or on another device cannot be completed later and subscribe them twice.
- Nothing is created until the User confirms. Abandoning the page leaves no Stripe Subscription, no trial and no API records.

## Flow

1. User hits the subscribe page. The steps are component state rather than routes, so there is nothing to route between and no landing check on a step.
   1. The billing page shows the subscribe control only to a User with no Stripe Subscription, and the route is guarded on the same. The guard reads the API Subscription off the Auth User and counts live, trialing, cancelled inside the paid term, past due and unpaid all as subscribed, so it lines up with the `already_subscribed` and `payment_required` refusals below. Refusing in the App is display. The API reads the same rule again on the request, so what the App said was never the rule.
   2. A secret in App Storage means the User is coming back from a bank challenge. Re-initialise against that Stripe Checkout Session and hand straight to the [submit flow](Create3Submit.md), no create. See the [submit flow](Create3Submit.md) for what the App does with it from there.
   3. Every visit without a stored secret creates.
2. The App asks the API for a Stripe Checkout Session.
   1. Load the API User's Stripe Customer id. Create the Stripe Customer if there isn't one, and save the returned id. The load goes first because the sweep below is addressed to a Stripe Customer, and a Stripe Customer made a moment ago has nothing to sweep. Passing `customer` on the Stripe Checkout Session is also what satisfies its email requirement, so no contact details element is needed.
   2. Expire every open Stripe Checkout Session the Stripe Customer already has. One live Stripe Checkout Session at a time, so a tab left open on another device cannot be confirmed after this one is handed out. Only an `open` Stripe Checkout Session can be expired and a completed one throws, so the sweep swallows the throw rather than failing the create on it. See the note below.
   3. Refuse if the User already has a Stripe Subscription. A live one, which counts a trial and a cancelled one still inside its paid term, comes back as `already_subscribed` on the response's `error` field. Past due or unpaid is `payment_required`, kept separate so the two can be told apart rather than telling a past due User they are already subscribed. Both default to billing until the App handles the past due case. See the [subscription guards flow](../Guards.md). Reject either way, and there is no success to hand back. The check sits after the sweep so that anything an abandoned Stripe Checkout Session already completed is visible to it rather than racing it. See the note below.
   4. Work out trial eligibility here. The API decides eligibility and never answers a question about it. The App reads whether the Stripe Checkout Session it was handed carries a trial, and uses that for what it says on screen and nothing else.
   5. Read whether the Stripe Customer has a Stripe Payment Method on file. That answer decides which address parameters go on the create, and nothing else on this request changes with it. See the note below.
   6. Create the Stripe Checkout Session with `ui_mode: 'elements'`, `mode: 'subscription'`, the Stripe Customer, `line_items` carrying the plan's price id at quantity one, a `return_url`, `allow_promotion_codes: true`, `automatic_tax: { enabled: true }` when the API's automatic tax flag is on, `subscription_data.payment_settings.save_default_payment_method`, and `subscription_data.trial_end` when eligible. Use `trial_end` rather than `trial_period_days`, since a User carrying a partial trial keeps whatever is left of it and a whole number of days cannot say that.
   7. Add `billing_address_collection: 'required'` and `customer_update: { address: 'auto', name: 'auto' }` only when there is no Stripe Payment Method on file. Those two are what carry a collected address onto the Stripe Customer at confirm, and a User with a Stripe Payment Method on file already has an address there for tax to read.
   8. A Stripe Payment Method already on the Stripe Customer comes back on the Stripe Checkout Session as `savedPaymentMethods`, carrying its id, brand and last4. Only one whose `allow_redisplay` is `always` appears, which is what the [payment method flow](../../payment-method/stripe/Update.md) sets when it stores one.
   9. A Stripe Payment Method up front on a trial is the default. `payment_method_collection` only needs setting when you want a trial without one, which is the opposite of what this flow wants, so the field is left alone.
   10. Nothing is created beyond the Stripe Checkout Session itself. No address is written to the Stripe Customer, no intent is opened, no Stripe Subscription exists. A User who abandons here leaves a Stripe Checkout Session that ages out on its own, or gets swept on their next visit, so there is nothing to deduplicate and nothing to clean up.
   11. Stripe erroring on the create errors back for display. Without a client secret there is nothing to mount, so the page cannot continue.
   12. Return the Stripe Checkout Session's `client_secret`.
3. The App initialises the Stripe Checkout SDK against that secret.
   1. Load stripe.js if it isn't already on the page.
   2. Call `stripe.initCheckoutElementsSdk({ clientSecret })`, then `await checkout.loadActions()` for the actions the rest of the page runs on.
   3. Nothing is mounted here. The page is still showing its loading state at this point and the containers the Stripe Elements go into are not in the document yet.
   4. Failing to load stripe.js, or failing to resolve the actions, leaves the User with nowhere to enter anything. Show the failure rather than an empty page where the form should be.
4. Which step opens, and what goes up with it, follows `savedPaymentMethods` on the Stripe Checkout Session.
   1. A Stripe Payment Method on file opens the confirm step and creates no Stripe Element at all. The address is on the Stripe Customer, the Stripe Payment Method is on the Stripe Customer, and there is nothing left for the User to type.
   2. No Stripe Payment Method on file opens the address step and creates both Stripe Elements once the loading state drops and the containers exist.
   3. Create the address element with `checkout.createBillingAddressElement()` and no prefill, the name along with the address. `contacts` is a different thing, a picker for addresses already saved against the Stripe Customer.
   4. Create the payment element with `checkout.createPaymentElement({ fields: { billingDetails: { name: 'never' } } })`. The Stripe Billing Address Element collects a name and gives no option not to, so the Stripe Payment Element has to stand down. Leaving both on it fails the confirm for collecting the same field twice.
   5. Country is an ISO alpha-2 select and the field layout follows the country, so the shape of an address is Stripe's problem rather than a hand rolled form's.
   6. Both Stripe Elements take the same appearance object, so they match the rest of the App.
   7. A step that is not open hides its container, it never removes it. Taking the container out of the document tears the mount down and the Stripe Element does not come back on its own.
   8. The Stripe Billing Address Element's change event is the only thing that reports what is in it. The event carries the value, which is what gets pushed onto the Stripe Checkout Session, and a complete flag, which is what says the User can move on. Both have to be held, since nothing else on the page knows either.
5. Read the Stripe Checkout Session for what to display, on every step.
   1. Call `actions.getSession()` for `total` and `lineItems`. Render those rather than pricing the plan in the App. This is enforced rather than advised. Reading and displaying either `total.total.amount`, or `total.total.minorUnitsAmount` alongside `currency` and `minorUnitsAmountDivisor`, is required and Stripe throws if you skip it.
   2. Tax is calculated off whatever address the Stripe Checkout Session is carrying and the discount off the promotion code, both by Stripe, both before anything is charged.
   3. The address step shows a total before tax and needs a line saying so. `tax.status` reads `requires_billing_address` until the Stripe Checkout Session has an address to calculate against, and the Stripe Billing Address Element does not put one there.
   4. A Stripe Payment Method on file means the Stripe Checkout Session reads tax off the Stripe Customer's address from the moment it is created. `tax.status` is `ready` and `tax.automaticTax.addressSource` is `customer`, so the confirm step opens with a final total.
   5. Use `canConfirm` to gate the confirm button. Stripe is the one that knows whether enough has been collected on either path.
   6. A trial carrying a promotion code shows two figures, `$0` today and the discounted amount from trial end. See the [user action flow](Create2UserAction.md).

## Diagram

The API handing out the Stripe Checkout Session.

```mermaid
flowchart LR
    A[User hits the subscribe page] --> A0{Stripe Subscription<br/>on the Auth User?}
    A0 -->|yes| A2[Control hidden and the route guarded.<br/>Default to billing]
    A0 -->|no| A1{Secret in App Storage<br/>from a confirm?}
    A1 -->|yes| Z[Re-initialise that Stripe Checkout Session<br/>and hand to the submit flow]
    A1 -->|no| B[App asks the API for a<br/>Stripe Checkout Session]
    B --> C[Load or create the Stripe Customer,<br/>save the returned id]
    C --> C1[Expire every open Stripe Checkout Session<br/>the Stripe Customer already has.<br/>One live one at a time]
    C1 --> B1{Stripe Subscription<br/>already on the User?}
    B1 -->|"live, past due or unpaid"| B2[Refuse. already_subscribed,<br/>or payment_required.<br/>Nothing is created]
    B1 -->|no| D[Work out trial eligibility.<br/>The App never asks]
    D --> D1{Stripe Payment Method<br/>on the Stripe Customer?}
    D1 -->|yes| E1["checkout.sessions.create<br/>ui_mode: elements, mode: subscription<br/>customer, line_items, return_url<br/>allow_promotion_codes, automatic_tax<br/>save_default_payment_method, trial_end?"]
    D1 -->|no| E2["Same create, plus<br/>billing_address_collection: required<br/>customer_update: address, name auto"]
    E1 --> F{Result}
    E2 --> F
    F -->|error| F1[Error back for display.<br/>No secret means nothing mounts]
    F -->|ok| G[Return client_secret]
```

The App initialising and picking the step to open.

```mermaid
flowchart LR
    G["initCheckoutElementsSdk({ clientSecret })<br/>then loadActions()"] --> I{Loaded?}
    I -->|no| I1[Show the failure, not an empty page<br/>where the form should be]
    I -->|yes| I3{savedPaymentMethods<br/>on the Stripe Checkout Session?}
    I3 -->|"none"| J["Address step opens. Create both<br/>Stripe Elements. Total before tax and<br/>says so, tax.status<br/>requires_billing_address"]
    I3 -->|"one on file"| M1["Confirm step opens, no Stripe Element<br/>created. tax.status ready,<br/>addressSource customer, brand and<br/>last4 off savedPaymentMethods"]
    J --> U[User action flow]
    M1 --> U
```

## Notes

### The Stripe Payment Element over hosted and embedded checkout

The Stripe Payment Element on a page the App renders itself over [hosted](../../../refs/subscription/stripe/CreateHosted.md) and [embedded](../../../refs/subscription/stripe/CreateEmbedded.md) checkout. The address and the payment method are Stripe Elements the App mounts and styles with the same appearance object as everything else in it, where hosted and embedded render Stripe's UI, styled by the logo, colors, fonts and border radius set in the Dashboard and nothing further. All three create the same kind of Stripe Checkout Session, so the checkout mechanics match and the fork is UI control against build cost.

What the choice costs is everything Stripe's UI does inside its own page. The steps, since the Stripe Billing Address Element does not write itself onto the Stripe Checkout Session. The mount lifecycle, including a secret held in App Storage so a bank challenge returns to the same Stripe Checkout Session. The total read off the Stripe Checkout Session and rendered by hand. The promotion code field, which no Stripe Element provides, so the input, the apply, the remove and the error a rejected code lands on are all the App's.

What it buys beyond styling is the confirm step for a User with a Stripe Payment Method on file, which is a page of the App's own text and no Stripe UI at all. Hosted and embedded put a payment form in front of that User whether or not anything needs collecting.

### Stripe API version

Everything here assumes `2026-03-25.dahlia` or later. Two separate reasons stack up to that floor. Version `2025-03-31.basil` is where the Stripe Subscription is created after payment completes, and anything earlier creates it upfront, so a refused charge leaves an incomplete Stripe Subscription with a finalized invoice sitting behind it to reconcile. Dahlia is where `ui_mode` gained `elements`, which is the value used throughout this flow. On Basil the same mode is named `custom`.

### Where the address comes from

Two paths put an address on the Stripe Customer, and this flow is one of them. A User with no Stripe Payment Method on file types an address here, `customer_update: { address: 'auto', name: 'auto' }` has Stripe copy it and the name onto the Stripe Customer at confirm, and the API writes neither. A User who already has a Stripe Payment Method on file put both there through the [payment method flow](../../payment-method/stripe/Update.md), which writes the Stripe Customer's address directly.

Nothing about the address is stored on the API. The Stripe Customer holds it, every renewal invoice computes tax off it, and Stripe's own invoices are where the User reads it back.

### Note on 2.2 - why open Stripe Checkout Sessions are expired

The check runs once, when the secret is handed out, and Stripe keeps a Stripe Checkout Session usable for up to a day after that. So a User could pass the check with no subscription, leave the tab open, subscribe on another device, then come back to the stale tab and press subscribe. It went through, and they had two.

Expiring the Stripe Customer's open Stripe Checkout Sessions on every create is what closes the hole. The stale tab is confirming against one that no longer accepts a confirm, so it fails there instead of buying a second Stripe Subscription, and the User starts again on a fresh Stripe Checkout Session. That is the whole reason the sweep exists.

A narrow window survives. The subscription check reads the API Subscription, and that record is only written once the sync call or the webhook lands, so it trails Stripe by a moment. Confirm on one device and then confirm on a second before the first API Subscription arrives and both go through. It takes the same person confirming twice within seconds of each other, so we are living with it.

Closing the window properly means asking Stripe for the Stripe Customer's live Stripe Subscriptions on every create rather than reading the API Subscription, which is a call to Stripe on every subscribe to cover that. Worth looking into later, not important now.

A Stripe Subscription created at the Dashboard or by an admin is outside all of this. There is no Stripe Checkout Session to expire, and blocking a second one could be wrong anyway, since it may well be deliberate.

### Note on 2.3 - why the subscription check sits here

Stripe creates the Stripe Subscription inside the confirm, in the App. The API sees the User once, when it hands out the Stripe Checkout Session secret, so that is the only place a check can run at all.

Reject either way. The Stripe Subscription exists whether it is live, past due or unpaid, so a second one is wrong regardless. There is nothing to hand back on success either, since the only thing this request returns is a Stripe Checkout Session secret, and a Stripe Subscription is not that. A double submit gets the same refusal as anything else.

Past due and unpaid get their own `error` code rather than sharing one, so the two can be told apart. Both default to billing until the App handles the past due case, which is a guard question and not one this flow answers. See the [subscription guards flow](../Guards.md).

### Note on 2.5 - why the branch is only about the address

The two creates differ by `billing_address_collection` and `customer_update` and nothing else. Both carry the same plan, the same promotion code setting, the same automatic tax setting and the same trial. The App reads `savedPaymentMethods` off the Stripe Checkout Session to decide which step to open, so the API never has to say which branch it took and the App never has to ask.

Asking for the address on a Stripe Checkout Session whose Stripe Customer already has one would put a step in front of a User with nothing to correct, and `customer_update` would then overwrite a tax address from a form they did not come to fill in.

## Todo

- Dunning. Past due and unpaid are refused here and belong to a flow that does not exist.
- Asking Stripe for the Stripe Customer's live Stripe Subscriptions on every create rather than reading the API Subscription, to close the window where two confirms seconds apart both go through.
- Trial eligibility is restricted to a single plan, always the cheapest one. Which plan that is, how the API identifies it, and what the App shows a User who picks any other plan are not pinned down.
