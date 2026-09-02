# Terms

Status: done
Updated: 2026-09-02

The master vocabulary. Every flow and doc in this repo draws its terms from here, word for word.

| Term | Description |
| --- | --- |
| API | The back end. Holds the API records and talks to the providers. |
| API Address | The API record holding the billing name and address. |
| API Payment Method | The API record holding the Stripe Payment Method's brand and last4. |
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
| Stripe Setup Intent | The Stripe object that stores a Stripe Payment Method against a Stripe Customer without charging it. |
| Stripe Subscription | The subscription at Stripe. |
| User | The human using the App. Never the App and never the API. |
