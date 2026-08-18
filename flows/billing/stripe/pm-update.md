# Billing Payment Method Update - Stripe (Payment Element)

Status: draft
Updated: 2026-08-17

## Purpose & Scope

Replacing the payment method on file with the Payment Element. The user already entered a valid payment method during subscribe, so a customer and a payment method exist. This is a swap, not a first capture. A SetupIntent is created on page load and the element mounts straight off its secret. Nothing is charged, the current cycle is already paid, the new payment method is what the next renewal invoice bills.

The page is dedicated to this one job, so arriving on it is already the user declaring intent. The element is mounted and ready on arrival rather than sitting behind an additional button, which would ask for the same declaration twice.

The cost is a SetupIntent opened for every visitor. Unlike create there is no subscription and no local row behind it, an unconfirmed SetupIntent goes stale on Stripe on its own, so there is nothing to clean up.

## Actors & Entities

Actors

- User - enters the new payment method in the Payment Element.
- Client App - requests the setup intent on load, mounts the element, confirms, polls after success.
- API - creates the SetupIntent, receives the webhook, repoints the defaults, detaches the old payment method, writes the local row.
- Stripe - issues the SetupIntent, runs 3DS, attaches the payment method, fires the webhook.

Entities

- User record - holds the Stripe customer id. Must already exist, the payment method being replaced was entered during subscribe.
- Stripe Customer - carries `invoice_settings.default_payment_method`, the customer level default.
- Subscription - carries its own `default_payment_method`, which overrides the customer level one.
- SetupIntent - created on page load with `usage: off_session`. No amount, no price, nothing about the subscription.
- PaymentMethod - old one detached, new one attached and defaulted. Two separate operations.
- Local billing row - brand, last4, expiry. Display only.

## Flow

1. User lands on the dedicated billing update page. It shows the payment method currently on file (brand, last4, expiry) off the local row.
2. On page load, without waiting for any user action, the client hits the API for a setup intent. Nothing to send, the customer is the authenticated user. The element cannot mount without a secret, so this fires before the form is usable rather than behind a save or a change payment method button.
   1. Load the user's Stripe customer id from your DB. It has to already exist, the payment method being replaced was entered during subscribe. No customer id means there is nothing to update, error back.
   2. Create a SetupIntent on Stripe with `customer` and `usage: 'off_session'`. The `off_session` part matters, the payment method gets charged by the renewal with nobody at the keyboard, and that is what sets the mandate up for it.
   3. No amount, no price, nothing about the subscription. A SetupIntent only stores a payment method.
   4. If it errors out, that error needs to get sent back to the client for display.
   5. Return the client secret. Unlike create there is no `type` to branch on, it is always a setup.
   6. A SetupIntent is opened for anyone who lands on the page. Unlike create there is no subscription and no local row behind it, an unconfirmed SetupIntent just goes stale on Stripe, so there is nothing to clean up.
3. Element mounts against that secret with `elements({ clientSecret })`, then `paymentElement.mount(target)`. Same as create, no mode, no amount, no currency, Stripe reads it off the intent.
4. User enters the new payment method and hits save.
5. `confirmSetup({ elements, clientSecret, confirmParams: { return_url }, redirect: 'if_required' })`. The `return_url` is mandatory.
6. Response is success / error / 3DS. Nothing is being charged but the bank can still want the payment method verified, so 3DS is on the table exactly like create. It either runs in a dialog and resolves inline, or sends the browser away to the bank and back to your return url.
7. If it redirected, the user comes back to a freshly loaded page with no state. Stripe appends `setup_intent_client_secret` to the return url. Read it and call `retrieveSetupIntent` to see how it landed rather than starting the flow over. Only one key to look for here, there is no payment variant.
8. On success the payment method is attached to the Stripe customer and that is all. It is not the default and nothing will charge it. Attaching and defaulting are two separate things, and this is where the flow is easy to get wrong.
9. Stripe fires `setup_intent.succeeded`, carrying the SetupIntent with its `payment_method`. The API does the real work off that webhook:
   1. Set `invoice_settings.default_payment_method` on the Stripe customer to the new payment method. That is what a future invoice reads.
   2. Set `default_payment_method` on the subscription to the same payment method, or clear it. A subscription level default overrides the customer level one, so if the old payment method is still pinned on the subscription the renewal charges the old payment method no matter what the customer record says. Nothing fails now, it surfaces a month later on the renewal.
   3. Detach the old payment method, otherwise every update leaves another payment method sitting on the customer.
   4. Write the new brand, last4 and expiry to the local row for display.
   5. Has to be idempotent, the same setup intent can arrive twice.
10. Reload the auth user (or the billing endpoint) and check the payment method on file. Poll this, a hit on the new last4 means the webhook arrived and the API picked it up. Give up after a ceiling rather than spinning forever.
11. Take the success action, redirect back to billing, wherever.
12. Nothing is charged by any of this. The current cycle is already paid, the new payment method gets used by the next renewal invoice.

## Diagram

```mermaid
flowchart LR
    A[Billing update page] --> B["Show payment method on file<br/>brand, last4, exp"]
    A -->|on load| C["POST /billing/intent"]
    C --> D[Load Stripe customer id]
    D --> E["setupIntents.create<br/>usage: off_session"]
    E --> F[Return client_secret]

    F --> G["elements({ clientSecret })<br/>mount element"]
    G --> H[User hits save]
    H --> I[confirmSetup]
    I --> J{Result}

    J -->|declined| H
    J -->|3DS redirect| J1["Back at return_url<br/>retrieveSetupIntent"]
    J -->|success| K
    J1 --> K

    K["Payment method attached to customer<br/>not default yet"] --> L["Stripe fires<br/>setup_intent.succeeded"]
    L --> M["Customer invoice_settings<br/>default_payment_method = new pm"]
    M --> N["Subscription default_payment_method<br/>= new pm or cleared"]
    N --> O[Detach old pm]
    O --> P["Local row -> new brand, last4, exp"]
    P --> Q[Client polls until last4 changes]
    Q --> R[Success action]
    R --> S["Next renewal invoice<br/>charges the new payment method"]
```

## States

SetupIntent. The local row has no status here, only payment method fields.

| State | Meaning |
| ----- | ------- |
| requires_payment_method | Created on page load, element mounted against it, no payment method entered |
| requires_action | 3DS, either a dialog or a bank redirect |
| succeeded | Payment method attached to the customer. Not default, nothing repointed, nothing will charge it |
| repointed | Webhook processed. Customer and subscription defaults moved, old payment method detached, local row written |

Allowed transitions

| From | To | Trigger |
| ---- | -- | ------- |
| none | requires_payment_method | Page load, SetupIntent created |
| requires_payment_method | requires_action | confirmSetup, bank wants the payment method verified |
| requires_payment_method | succeeded | confirmSetup, no challenge |
| requires_action | succeeded | Challenge passed inline or at the return url |
| requires_payment_method | requires_payment_method | Declined. Same secret stays confirmable, user retries |
| succeeded | repointed | `setup_intent.succeeded` processed |
| requires_payment_method | stale | User abandons. Stripe ages it out, nothing local to clean |

## Rules

- Authenticated user required.
- A Stripe customer must already exist. No customer id is an error, not a create.
- `usage: off_session` is mandatory. The renewal charges with nobody at the keyboard, that is what the mandate is for.
- Attaching is not defaulting. Success on the SetupIntent alone changes nothing about what gets billed.
- Both defaults have to be handled. The subscription level default overrides the customer level one, so repointing only the customer leaves the old payment method billing at the next renewal.
- Detach the old payment method, or every update leaves another payment method on the customer.
- Skip the detach when the new payment method id equals the old one.
- Only the webhook repoints and writes. A resolved confirm is not proof.
- Webhook handling is idempotent, the same setup intent can arrive twice.
- Polling has a ceiling. Past it, show a pending state rather than spinning.
- Nothing is charged. The current cycle is already paid.

## Edge & Error Cases

| Case | Cause | Expected behavior |
| ---- | ----- | ----------------- |
| No Stripe customer id | User never subscribed, or the id was never saved | Error back to the client. There is no payment method to replace and this flow does not create one |
| Payment method declined | Bank refused | Element stays mounted against the same secret, user corrects and submits again. No new secret needed |
| 3DS sends the browser away | Bank requires a challenge page | User returns to a cold page. Read `setup_intent_client_secret` off the url and `retrieveSetupIntent` rather than restarting |
| Abandoned page | SetupIntent opened for every visitor | Unconfirmed intent goes stale on Stripe on its own. No local row behind it, nothing to clean up |
| Subscription default left pinned | Only the customer level default repointed | Renewal charges the old payment method. Nothing fails at update time, it surfaces a month later |
| Old payment method not detached | Detach step skipped | Payment methods pile up on the customer, one per update |
| New payment method is the one already on file | User re-enters the same details | Stripe issues a new payment method id. Repoint and detach the old one as normal |
| Webhook lands late | Asynchronous, can arrive before confirm resolves | Poll the payment method on file, show pending until last4 changes |
| Webhook never arrives | API dropped or failed the attempts | Payment method is attached at Stripe but nothing is defaulted and the row still shows the old payment method. The renewal charges the old payment method. Needs a manual sync |
| Same setup intent delivered twice | Stripe retries | Handler is idempotent, second pass is a no-op |
| Repoint succeeds, local row write fails | Partial failure mid webhook | Stripe is correct, display is stale, client polls to the ceiling. Needs a manual sync |

## Decisions

Dedicated page over an inline form or a dialog. 3DS can send the browser to the bank and back to a cold url with the secret on it. The dedicated page comes back up as itself, reads the secret and retrieves the intent. A dialog or an inline section has to work out it is mid flow and rebuild the state it was in before the redirect. Same result, more work.

The dedicated page fetches the intent on load. There is no need for additional setup or buttons here since hitting the page is the intent already. Gating it behind an additional button would only lower the count of stale SetupIntents, not remove them.

The repoint runs off the webhook, not off the confirm response. The browser can be closed or sent to the bank at that moment, so the confirm may never resolve on the page that started it. The webhook is the only path that always arrives.

Detach the old payment method rather than keeping a list. One payment method on file is the model everywhere else in the flow, and the local row holds a single brand, last4 and expiry.

## TODO

Now

- Decide whether the subscription level default gets set to the new payment method or cleared. Clearing leans on the customer default, setting it is explicit. Pick one and use it everywhere.
- Decide the polling ceiling and what the pending state shows.
- Decide the behavior when a payment method exists with no subscription. The repoint has no subscription to touch.

Later

- Manual sync command to reconcile the payment method on file against Stripe when a webhook is dropped.
- Multiple payment methods on file. The flow assumes exactly one throughout.

Out of scope

- First payment method capture. That is the create flow.
- Payment method removal. See the delete flow.
- Dunning and failed renewals.
