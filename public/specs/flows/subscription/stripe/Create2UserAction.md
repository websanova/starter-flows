# Subscription Create User Action - Stripe (Checkout Sessions, Payment Element)

Status: done
Updated: 2026-09-29

## Description

The User works through the subscribe page on the Stripe Checkout Session the [load flow](#flows/subscription/stripe/Create1Load) handed them, filling the address, entering a payment method and applying a promotion code. Everything typed goes onto the Stripe Checkout Session as the User moves, so the total, the tax and the discount are Stripe's to calculate and the App only renders them.

## Requirements

- Show the total, tax and discount as Stripe calculates them. Nothing is priced in the App.
- The address step shows a total before tax and says so, since tax cannot be calculated until the Stripe Checkout Session carries an address.
- Support promotion codes, applied and validated by Stripe against the Stripe Checkout Session.
- Let the User take an applied promotion code back off and see the total return to what it was.
- A promotion code on a trial discounts the first charge at trial end, since signup charges nothing either way.
- The confirm step shows the Stripe Payment Method on file as its brand and last4, with no picker.

## Flow

1. App renders the totals off the Stripe Checkout Session, on every step.
   1. Call `actions.getSession()` for `total` and `lineItems`. Rendering either `total.total.amount`, or `total.total.minorUnitsAmount` alongside `currency` and `minorUnitsAmountDivisor`, is required and Stripe throws if you skip it.
   2. Read the tax off the Stripe Checkout Session, calculated against whatever address it carries, and the discount off the promotion code. Both are Stripe's, both before anything is charged.
   3. Read the trial off the Stripe Checkout Session. That is the only place the App learns there is one.
   4. Gate the subscribe button on `canConfirm`. Stripe is the one that knows whether enough has been collected.
2. App collects the address. A User with a Stripe Payment Method on file enters at step 4, so this step and step 3 never run for them. The [load flow](#flows/subscription/stripe/Create1Load) decides which one they open on.
   1. Show a total before tax and say so. `tax.status` reads `requires_billing_address` until the Stripe Checkout Session carries an address, and the Stripe Billing Address Element does not put one there.
   2. Push the value with `updateBillingAddress()` and unmount the Stripe Billing Address Element when the User continues. One action, never split. Stripe refuses to confirm while the Stripe Billing Address Element is mounted and the address has also been set that way, since the two are competing sources.
   3. Keep the instance on unmount, along with everything typed into it, so coming back is a remount and nothing is lost.
   4. Error on the step when Stripe cannot place the address. The User corrects the address and continues again.
3. App collects the payment method.
   1. Show the Stripe Payment Element and a continue to the confirm step. The Stripe Billing Address Element is already down by this point, which is the whole of what satisfies the competing sources refusal.
   2. Remount the Stripe Billing Address Element when the User goes back, which blocks the confirm step while it stands.
4. App shows the confirm step.
   1. Show the total, the promotion code field and the subscribe button.
   2. Show the Stripe Payment Method on file as its brand and last4, read off `savedPaymentMethods`. There is no picker, since exactly one is ever on file.
   3. Show a final total when a Stripe Payment Method is on file. `tax.status` is `ready` and `tax.automaticTax.addressSource` is `customer`, since the Stripe Checkout Session reads tax off the Stripe Customer's address from the moment it is created.
   4. Write the promotion code field and its apply control by hand. No Stripe Element collects one.
   5. Apply a code with `actions.applyPromotionCode()`. Setting `allow_promotion_codes` is what puts the field in play, so there is no verify call of your own and no code travelling on a subscribe payload to be resolved later.
   6. Re-read with `getSession()` once the action resolves. The discount lands on `total.discount`, `total.subtotal` drops, tax recalculates against the reduced amount and `total.total` follows. The App subtracts nothing.
   7. Remove with `actions.removePromotionCode()` and the same re-read, so a User who applied a code can reach the undiscounted total without reloading the page.
   8. Error on the field a rejected code was typed into, not on the page. Expired, unknown and not applicable to the plan all land there the same way, and the User types another code or confirms without one.

## Diagram

The User walking the steps.

```mermaid
flowchart LR
    J["Address step.<br/>Total before tax and says so,<br/>tax.status requires_billing_address"] --> K[User presses continue]
    K --> K1["updateBillingAddress() pushes the value, then<br/>the Stripe Billing Address Element unmounts.<br/>One action, never split"]
    K1 --> K2{Stripe places<br/>the address?}
    K2 -->|no| K3[Error on the step. The User corrects<br/>the address and continues again]
    K3 --> J
    K2 -->|yes| L[Payment method step.<br/>Stripe Payment Element]
    L -.->|change address| J
    L --> M["Confirm step. getSession() total carries tax.<br/>canConfirm gates the subscribe button"]
    M1["Stripe Payment Method on file.<br/>Brand and last4 off savedPaymentMethods,<br/>no picker. tax.status ready,<br/>addressSource customer, total is final"] --> M
    M --> N[Submit flow]
```

The promotion code on the confirm step.

```mermaid
flowchart LR
    M["Confirm step"] --> P1{Promotion code?}
    P1 -->|apply| P2["applyPromotionCode(), then re-read with<br/>getSession(). Discount on total.discount,<br/>subtotal drops, tax recalculates,<br/>total follows"]
    P1 -->|remove| P3["removePromotionCode(), the same<br/>re-read puts the total back"]
    P1 -->|rejected| P4[Error on the field the code was typed<br/>into, not on the page. Another code,<br/>or confirm without one]
    P2 --> M
    P3 --> M
    P4 --> M
```

## Notes

### A promotion code against a trial

A trial charges nothing at signup, so an applied promotion code takes nothing off what the User is about to pay. The discount sits on the recurring total the Stripe Checkout Session is carrying, rides onto the Stripe Subscription at confirm, and comes off the first real invoice when the trial ends.

Two figures have to be on screen for that to read correctly. A total of `$0` today, and the discounted amount from trial end. Showing only the discounted recurring figure reads as a charge that is not happening, and showing only the `$0` throws away the reason the User typed a code.

Whether the discount survives to trial end is the coupon's own duration rather than anything this flow sets. A code lasting one billing period is spent on the first invoice after the trial, which is the invoice the User was expecting it against.
