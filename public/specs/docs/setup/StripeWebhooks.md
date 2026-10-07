# Stripe Webhooks

Status: done
Updated: 2026-10-07

One endpoint takes every Stripe event. Most of them back a flow that already wrote from the browser, so the webhook is the second writer and every handler is idempotent.

## Events

| Event | Writes | Flow |
| --- | --- | --- |
| `checkout.session.completed` | API Subscription, API Payment Method, both Stripe defaults | [Create Submit](#flows/subscription/stripe/Create3Submit) |
| `customer.subscription.updated` | API Subscription, cancelled marker, end date, plan, interval | [Cancel](#flows/subscription/stripe/Cancel), [Resume](#flows/subscription/stripe/Resume), [Update](#flows/subscription/stripe/Update) |
| `setup_intent.succeeded` | API Payment Method, both Stripe defaults | [Update Submit](#flows/payment-method/stripe/Update2Submit) |
| `payment_method.detached` | Clears the API Payment Method's brand and last4 | [Delete](#flows/payment-method/stripe/Delete) |

One handler covers `customer.subscription.updated` for all three subscription flows, and the same handler covers a change made in the Stripe Dashboard. It reads the Stripe Subscription and writes what it finds rather than branching on which flow ran.

## Events not handled

| Event | Why it matters |
| --- | --- |
| `invoice.payment_failed` | The start of dunning. The Stripe Subscription moves to `past_due` and the User has to be sent somewhere that can settle it. |
| `invoice.paid` | The renewal that keeps a Stripe Subscription live. The API learns about one only through `customer.subscription.updated`. |
| `customer.subscription.deleted` | The end of a cancelled term, and a Stripe Dashboard deletion. Nothing clears the API Subscription, which counts as subscribed until its stored end date passes. |

## Handling

- One endpoint, one signing secret. The signature is verified before the payload is read, and an event that fails verification runs nothing.
- Respond first, write after. Stripe treats a slow handler the same as a failed one and retries for days.
- Every handler is idempotent. A Stripe retry, or a sync call that already wrote, returns without error and without a second write.
- Neither writer is guaranteed first. The sync call exists because the User is waiting, the webhook because the browser can be closed or sent to a bank and never come back.
- A Stripe Setup Intent carries no address, so the Stripe Customer's address is the one field only the sync call writes.

## Local development

The Stripe CLI container forwards events inward, so a local endpoint receives the same payloads and the same signature header as production with its own signing secret. See the [Docker doc](#docs/setup/Docker) for the container.
