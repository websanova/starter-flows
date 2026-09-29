# Subscription Resume - Stripe

Status: built
Updated: 2026-09-29

## Description

A User resumes a cancelled Stripe Subscription from a dedicated confirm page, while the paid term is still running and a Stripe Payment Method resolves for the renewal. The existing Stripe Subscription carries on, so the plan and interval are not picked again and nothing is charged on confirm.

## Requirements

- Authenticated Users only.
- Resume a cancelled Stripe Subscription that is still inside the paid term.
- Refused once the end date has passed.
- Refused when no Stripe Payment Method resolves for the renewal.
- The control leads to a dedicated confirm page rather than an inline button or a dialog.
- The page states the plan, the interval and the date billing picks back up. Confirm is the only action on it.
- The App hides the control on the same rule, and the API decides it again on every request.
- Resume keeps the existing Stripe Subscription. The plan and interval are not picked again.
- Nothing is charged on confirm.
- The API writes the API Subscription off Stripe's response, and the matching Stripe webhook rewrites the same fields as a backstop.
- The response is the answer. Nothing settles afterwards, so there is nothing to poll.

## Flow

1. App shows the resume control on the billing page, only on a cancelled Stripe Subscription that is still inside the paid term with a Stripe Payment Method on file. Hiding it is display, the API reads the same rule again on the request.
   1. Send a User whose end date has passed through the [subscription create flow](#flows/subscription/stripe/Create1Load). There is nothing left at Stripe to resume.
2. App opens a dedicated confirm page. The page states the plan, the interval and the date billing picks back up, and confirm is the only action on it.
3. App calls the API when the User confirms. No body, the Stripe Subscription is resolved from the API User.
4. API resumes the Stripe Subscription.
   1. Re-read the API Subscription and refuse once the end date has passed. The gate only gets stricter with time, so a term that ran out while the page sat open is refused here and the User subscribes again through create.
   2. Refuse when no Stripe Payment Method resolves. Read `default_payment_method` on the Stripe Subscription, falling back to `invoice_settings.default_payment_method` on the Stripe Customer, which is the order the renewal itself reads. Not the API Payment Method, which is display and lags the webhook. See the note below.
   3. Return the current state when the Stripe Subscription is not cancelled, and change nothing at Stripe. A double submit, a second tab and a direct call all land here.
   4. Update the Stripe Subscription with `cancel_at_period_end` set to false. Stripe returns the updated Stripe Subscription in the same call, status still `active`, `cancel_at_period_end` false, `cancel_at` null.
   5. Write the API Subscription off the returned object, clearing the cancelled marker and the end date.
   6. Error back for display when Stripe refuses the update. The API Subscription is left as it is and the User retries.
5. App refreshes the Auth User and takes the success action, back to billing or a confirmation page.
   1. Read the new state off the refreshed Auth User rather than the response body, since subscription state is spread across flags the Auth User carries and the response holds only the one record.
6. API runs the same write on `customer.subscription.updated`. The event and the response carry the same fields, so whichever lands second rewrites the same values, and the same handler covers a resume done in the Stripe Dashboard.

## Diagram

```mermaid
flowchart LR
    A[User hits resume on the billing page] --> B{Cancelled, inside the term,<br/>Stripe Payment Method on file?}
    B -->|no| C[Control hidden, request refused]
    B -->|yes| D["Confirm page.<br/>Plan, interval, next charge"]
    D --> E[User confirms]
    E --> F[App calls the API]
    F --> G{Re-read the API Subscription}
    G -->|end date passed| G1[Refuse. The User subscribes<br/>again through create]
    G -->|no Stripe Payment Method| G2[Refuse. The renewal would fail]
    G -->|not cancelled| G3[Return the current state,<br/>nothing changes at Stripe]
    G -->|cancelled, inside the term| H["subscriptions.update<br/>cancel_at_period_end: false"]
    H --> I{Result}
    I -->|error| J[Error back for display,<br/>API Subscription untouched]
    I -->|ok| K["Write the API Subscription off the response.<br/>Clear the cancelled marker and end date"]
    K --> L[Refresh the Auth User,<br/>success action]
    K -.-> M["customer.subscription.updated<br/>lands after, same fields"]
```

## Notes

### Nothing is charged on confirm

The Stripe Subscription bills again at the renewal it was always going to bill at. Clearing `cancel_at_period_end` generates no invoice.

### Refusing a resume with no Stripe Payment Method on file

Stripe does not reject one. Clearing `cancel_at_period_end` generates no invoice and attempts no payment, so there is nothing for Stripe to validate against, the Stripe Subscription is `active` throughout, and a Stripe Customer with no Stripe Payment Method is a legal state. The failure lands at the renewal instead, where the invoice cannot be paid and the Stripe Subscription drops into dunning. Letting the resume through trades one refusal now for a guaranteed failed payment weeks later.

### Cancelled with no Stripe Payment Method on file

Rare in practice. The Stripe Payment Method is stored during subscribe and a cancellation does not touch it, so the only way to arrive here is a User who deleted the card themselves after cancelling. The resume control is hidden and the API refuses until a card is back on the Stripe Customer.

Putting one there is the [payment method flow](#flows/payment-method/stripe/Update1Collect)'s job, which takes a Stripe Payment Method whether or not one is already on file. Resume has no opinion on how the User gets a card back, only that it will not run until one resolves.

### Plan controls while cancelled

Anywhere plans are listed, a cancelled Stripe Subscription still inside the term has exactly one thing on offer, resuming the plan and interval it already carries. Every other control on that surface leads somewhere that refuses, so none of them belong there. Absent rather than disabled, since a disabled control still needs a label and there is no honest one to put on it.

The same applies to an interval toggle. Resume takes no plan and no interval, so a toggle that changes what the resume control appears to offer is describing something the flow cannot do.
