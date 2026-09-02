# Subscription Guards

Status: wip
Updated: 2026-08-30


## Notes

* Guards for api flags like subs required on/off (api and client), also related to if payment fails and is in "past due" state or other, etc. How this will affect those guards and plan limits, do they fallback to freemium if in that mode etc.
* Also there should be like has_payment_method (already) and perhaps like a "status" column on the payment_method object to detect this kind of stuff for the client.

---

* In Laravel how does a pm get filled in on the api side if it's Apple/Google pay or Klarna, or some other thing, like does the users card type/last_four always get filled in by Cashier somehow? Like how else do we know if there is a payment method at all times? where does has_payment_method value even come from, etc?

---

* Dunning and failed renewals. Moved here from the [payment method update flow](../billing/stripe/PaymentMethodUpdate.md), since what it really decides is what a User can still do while their payment is failing, which is a guard question rather than a card question.

  When a renewal fails, Stripe retries on its own schedule, a handful of attempts over a couple of weeks depending on the policy set in the dashboard. The Stripe Subscription sits at `past_due` for that whole window and moves to `unpaid` once the attempts run out. Cashier treats both as not active by default, so `is_subscribed` is false the moment the first renewal fails and the User is locked out of everything behind the subscribed guard, for weeks, while Stripe is still trying and while they may well have no idea anything is wrong.

  That is the guard question. Whether a User whose payment is failing keeps access through the retry window, drops to the free tier where there is one, or is locked out immediately as they are today. It is the same decision as the past due handling in the notes above, just arriving from the other side.

  The rest of it is notices. Nothing tells the User the renewal failed, nothing tells them how long they have, and nothing tells them it finally gave up. Stripe can send its own emails for this, or the API can handle it off `invoice.payment_failed` and the status moves that follow. Cashier has a payment notification hook for the authentication case that is not turned on.

  The payment method update flow charges the open invoice while the User is on the page, so the one path where they actively come and fix it is covered. Everything above is what happens to the User who does not.
