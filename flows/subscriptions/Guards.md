# Subscription Guards

Status: WIP
Updated: 2026-08-30


## Notes

* Guards for api flags like subs required on/off (api and client), also related to if payment fails and is in "past due" state or other, etc. How this will affect those guards and plan limits, do they fallback to freemium if in that mode etc.
* Also there should be like has_payment_method (already) and perhaps like a "status" column on the payment_method object to detect this kind of stuff for the client.

---

* In Laravel how does a pm get filled in on the api side if it's Apple/Google pay or Klarna, or some other thing, like does the users card type/last_four always get filled in by Cashier somehow? Like how else do we know if there is a payment method at all times? where does has_payment_method value even come from, etc?
