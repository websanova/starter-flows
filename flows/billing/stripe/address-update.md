# Billing Address Update - Stripe

Status: draft
Updated: 2026-08-17

## Flow

1. User opens the billing address form, prefilled off the local record. No element, no intent, no client secret. This is a customer record write, not a payment.
2. Submit sends the address to the API.
   1. Validate the shape. Country, line1, city, postal code, and state where the country requires one.
   2. Load the user's Stripe customer id. If there isn't one the user has never subscribed, so the write is local only and there is nothing to push.
   3. Set `address` on the Stripe customer. This is the field `automatic_tax` reads when it computes an invoice.
   4. Read `customer.tax.automatic_tax` off the response. `supported` means the next invoice computes tax. `unrecognized_location` or `failed` means the next renewal invoice will not finalize, and that has to surface here rather than a month later.
   5. Write the address to the local row.
3. Nothing is charged and no invoice is created. The current cycle is already finalized and its tax is locked at the rate that applied when it was issued.
4. The change applies from the next renewal invoice. Stripe recomputes tax off the customer address every time it creates one, so nothing on the subscription has to be re-pointed.
5. The payment method's `billing_details.address` is a separate field and is not touched by this. That one is AVS data the bank checks against the card, tax reads the customer address. Editing the address here does not change it, and editing the card does not change the tax address.
6. The response is the answer. Nothing asynchronous, no webhook, no polling.
