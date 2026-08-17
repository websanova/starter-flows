# Billing Delete - Stripe

Status: draft
Updated: 2026-08-17

## Purpose & Scope

Removing the card on file once the subscription can no longer be charged. No element, no confirm, no 3DS, nothing asynchronous, so the request is one API call and its response is the answer. Counterpart to the swap in [update.md](update.md).

Gated on no further invoice being due. Allowed once the subscription has ended, or while it is cancelled and running out a grace period to the end of the paid term. Refused while a subscription is live and will renew, pulling the card there only books a failed renewal.

Detach is not a delete on Stripe's side. The payment method survives, unhooked from the customer, and can never be re-attached. Removal is permanent. Subscribing again later goes through the create flow, which collects a card from scratch.

Not covered: the card swap ([update.md](update.md)), subscribe, cancel, resume, plan change, dunning, failed renewals.

## Flow

1. Delete is gated on the subscription no longer being chargeable. Allowed once the subscription has ended, or while it is cancelled and running out a grace period to the end of the paid term. In both cases no further invoice is coming. Refused while a subscription is live and will renew, pulling the card there just books a failed renewal.
2. The gate is decided on the API side. The client hides or disables the control off the same subscription state, but that is display only, the API re-checks it.
3. Client hits the delete endpoint.
   1. Re-check the gate against the local subscription row. Refuse if anything is still going to be charged.
   2. Detach the payment method from the Stripe customer.
   3. What detaching does to the defaults is the open bit. It should null `invoice_settings.default_payment_method` on the customer, but whether it also clears `default_payment_method` on a subscription that is still live through a grace period is not confirmed. If it does not, the subscription is left pointing at a detached card. Needs checking against Stripe before this gets built.
   4. Clear the card fields on the local row.
   5. Return.
4. No element, no confirm, no 3DS, so there is nothing asynchronous to wait on. The response is the answer, no polling.
5. `payment_method.detached` lands afterwards. Idempotent, the local row is already clear by then.
6. Subscribing again later goes through the create flow, which collects a card from scratch.

## Diagram

```mermaid
flowchart LR
    A[User hits delete] --> B{Subscription still<br/>chargeable?}

    B -->|active and renewing| C[Refuse]
    B -->|ended or grace period| D["DELETE /billing/card"]

    D --> E[Re-check gate API side]
    E --> F[paymentMethods.detach]
    F --> G["Customer default cleared<br/>sub default: open question"]
    G --> H[Clear card fields on local row]
    H --> I[Return, nothing to poll]
    I --> J["payment_method.detached<br/>lands later, no-op"]
```
