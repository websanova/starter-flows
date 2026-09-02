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

  The payment method update flow does none of this. It swaps the Stripe Payment Method on file and stops, and replacing one does not prompt Stripe to retry anything, so a User sent there comes back still past due. Recovery is its own flow and probably its own page.

---

* Payment recovery, the shape of it. A past due User lands on a page whose only job is to get them current again, and everything it needs comes off the Stripe Subscription's open invoice rather than off anything stored.

  The API call behind it reads that invoice and answers one of two things. Nothing owed, because a Stripe retry cleared it while the User was walking over, so send them back to billing. Or here is the invoice's confirmation secret.

  The page mounts a Stripe Payment Element against that secret. That one mount covers both of the ways this goes wrong. A Stripe Payment Method that is fine but whose bank wants the charge authenticated confirms against the card already on file and the challenge runs there. A Stripe Payment Method that is simply dead takes a new one in the same element. Either way it is one page, one confirm, and the invoice is paid at the end of it.

  Worth being clear about why it is not just a redirect to the payment method update page. Half the time the Stripe Payment Method was never the problem, so asking for a new card is asking for something that was never wrong. And the round trip means two bank challenges back to back, one for storing the card and one for the charge, for a single problem. Confirming the invoice directly collapses both.

  Open question. Once the invoice is paid with a new Stripe Payment Method, that card should become the one on file rather than paying once and vanishing. Stripe can do this itself through the Stripe Subscription's `save_default_payment_method`, which wants confirming before anything is built on it.

  Also unanswered, whether the page is reachable on its own or only ever arrived at from a guard, and what it shows a User who is not past due at all.
