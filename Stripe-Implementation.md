Awesome—here’s a clean, practical setup you can follow to integrate **Stripe** with the **Mwanga** application. I’ll show two paths:

* **A. Simple store** (single-vendor) using Checkout/Payment Intents
* **B. Marketplace** (multi-vendor payouts) using **Stripe Connect (Express)**

---

# A) Core Setup (works for both paths)

1. **Create a Stripe account & keys**

* Go to Stripe Dashboard → Developers → API keys
* Copy **Publishable key** and **Secret key** (use **Test mode** first)
* Add to Mwanga’s secrets:

```
STRIPE_SECRET_KEY=sk_test_...
STRIPE_PUBLISHABLE_KEY=pk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...   # fill after you create the webhook
```

2. **Install SDKs**

* **Java (Spring Boot)** backend:

```xml
<!-- pom.xml -->
<dependency>
  <groupId>com.stripe</groupId>
  <artifactId>stripe-java</artifactId>
  <version>24.9.0</version>
</dependency>
```

* **JS frontend** (if using React/Next):

```
npm i @stripe/stripe-js
```

3. **Initialize Stripe in backend**

```java
// e.g., a @Configuration class
import com.stripe.Stripe;
import jakarta.annotation.PostConstruct;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Configuration;

@Configuration
public class StripeConfig {
  @Value("${STRIPE_SECRET_KEY}")
  private String secret;

  @PostConstruct
  public void init() { Stripe.apiKey = secret; }
}
```

4. **Decide your payment flow**

* **Checkout Session** (fastest, PCI-safe hosted page)
* **Payment Intents + Elements** (custom UI; more control)

---

# A1) Simple Store Flow (Checkout Sessions)

### Backend: create a Checkout Session

```java
// POST /api/payments/create-checkout-session
@PostMapping("/api/payments/create-checkout-session")
public ResponseEntity<Map<String, Object>> createCheckoutSession(@RequestBody CreateSessionRequest req) throws Exception {
  List<Object> lineItems = new ArrayList<>();

  Map<String, Object> priceData = new HashMap<>();
  priceData.put("currency", "usd");
  priceData.put("unit_amount", req.amountCents()); // e.g., 1000 = $10.00
  Map<String, Object> productData = Map.of("name", req.productName(), "metadata", Map.of("mwanga_product_id", req.productId()));
  priceData.put("product_data", productData);

  Map<String, Object> lineItem = new HashMap<>();
  lineItem.put("quantity", 1);
  lineItem.put("price_data", priceData);
  lineItems.add(lineItem);

  Map<String, Object> params = new HashMap<>();
  params.put("mode", "payment");
  params.put("line_items", lineItems);
  params.put("success_url", req.successUrl()); // e.g., https://mwanga.app/checkout/success?session_id={CHECKOUT_SESSION_ID}
  params.put("cancel_url", req.cancelUrl());   // e.g., https://mwanga.app/checkout/cancel

  com.stripe.model.checkout.Session session = com.stripe.model.checkout.Session.create(params);

  return ResponseEntity.ok(Map.of("id", session.getId(), "url", session.getUrl()));
}

record CreateSessionRequest(String productId, String productName, Long amountCents, String successUrl, String cancelUrl) {}
```

### Frontend: redirect to Checkout

```ts
import {loadStripe} from '@stripe/stripe-js';

const stripe = await loadStripe(process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY!);

const res = await fetch('/api/payments/create-checkout-session', {
  method: 'POST',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({
    productId: 'abc123',
    productName: 'Kente Textile',
    amountCents: 2500,
    successUrl: 'https://mwanga.app/checkout/success',
    cancelUrl: 'https://mwanga.app/checkout/cancel'
  })
});
const {id, url} = await res.json();
window.location.href = url; // quickest
// or await stripe?.redirectToCheckout({sessionId: id});
```

### Webhook: mark orders paid

```java
// POST /webhooks/stripe
@PostMapping("/webhooks/stripe")
public ResponseEntity<String> handleStripe(@RequestHeader("Stripe-Signature") String sigHeader,
                                           @RequestBody String payload,
                                           @Value("${STRIPE_WEBHOOK_SECRET}") String endpointSecret)
        throws Exception {
  com.stripe.model.Event event;
  try {
    event = com.stripe.net.Webhook.constructEvent(payload, sigHeader, endpointSecret);
  } catch (Exception e) {
    return ResponseEntity.status(400).body("Bad signature");
  }

  switch (event.getType()) {
    case "checkout.session.completed" -> {
      var session = (com.stripe.model.checkout.Session) event.getDataObjectDeserializer().getObject().get();
      // session.getId(), session.getPaymentIntent()
      // TODO: update Mwanga DB: order -> paid, send receipt, fulfill
    }
  }
  return ResponseEntity.ok("ok");
}
```

**Run the webhook locally:**

```
stripe login
stripe listen --forward-to localhost:8080/webhooks/stripe
```

Copy the `whsec_...` into `STRIPE_WEBHOOK_SECRET`.

---

# B) Marketplace Flow (Stripe Connect, recommended for Mwanga)

If Mwanga pays **individual sellers**, use **Connect (Express)**:

**Key ideas**

* Each seller has a **Connected Account** (Express).
* Buyers pay Mwanga; Stripe routes funds to the seller minus Mwanga’s fee.
* Use **Checkout Session** with `payment_intent_data[application_fee_amount]` and `transfer_data[destination]`
  *or* **PaymentIntent** with a **destination charge**.

### Steps

1. **Enable Connect** in Dashboard → Settings → Connect → Choose **Express**.

2. **Create a connected account (per seller)**

```java
// Create Express account
Map<String, Object> accParams = new HashMap<>();
accParams.put("type", "express");
accParams.put("country", "US"); // seller country
accParams.put("email", "seller@example.com");
var account = com.stripe.model.Account.create(accParams);

// Create onboarding link
Map<String, Object> linkParams = new HashMap<>();
linkParams.put("account", account.getId());
linkParams.put("refresh_url", "https://mwanga.app/onboarding/refresh");
linkParams.put("return_url", "https://mwanga.app/onboarding/return");
linkParams.put("type", "account_onboarding");
var link = com.stripe.model.AccountLink.create(linkParams);

// Send link.getUrl() to seller to complete KYC
```

Store `account.getId()` (e.g., `acct_123`) on the seller record.

3. **Create a Checkout Session that splits funds**

```java
// buyer pays $100. Mwanga takes $10 fee; seller receives $90.
long amountCents = 10000;
long appFeeCents = 1000;
String sellerAccountId = "acct_...";

Map<String, Object> priceData = Map.of(
  "currency", "usd",
  "unit_amount", amountCents,
  "product_data", Map.of("name", "Handwoven Basket")
);

Map<String, Object> sessionParams = new HashMap<>();
sessionParams.put("mode", "payment");
sessionParams.put("line_items", List.of(Map.of("quantity", 1, "price_data", priceData)));
sessionParams.put("success_url", "https://mwanga.app/checkout/success?session_id={CHECKOUT_SESSION_ID}");
sessionParams.put("cancel_url", "https://mwanga.app/checkout/cancel");

// Attach fee + destination
sessionParams.put("payment_intent_data", Map.of(
  "application_fee_amount", appFeeCents,
  "transfer_data", Map.of("destination", sellerAccountId)
));

var session = com.stripe.model.checkout.Session.create(sessionParams);
```

4. **Handle webhooks**

* `checkout.session.completed` → mark order paid
* `account.updated` → track KYC/verification
* `charge.succeeded` / `payment_intent.succeeded` → optional post-processing
* **Payouts**: Stripe will pay out to sellers per their account settings.

---

# Operational Best Practices

* **Use Test Mode** end-to-end first:

  * Test cards: `4242 4242 4242 4242` (Visa), any future expiry, any CVC, any ZIP.
* **Security**

  * Never expose `sk_...` to the frontend.
  * Verify **webhook signatures** (as shown).
  * Use **idempotency keys** on backend requests that could be retried:

    ```java
    RequestOptions opts = RequestOptions.builder().setIdempotencyKey(UUID.randomUUID().toString()).build();
    com.stripe.model.checkout.Session.create(params, opts);
    ```
* **Metadata**

  * Add `metadata` (orderId, buyerId, sellerId) to Sessions/PaymentIntents for reconciliation.
* **Currencies & amounts**

  * Stripe expects **minor units** (cents).
* **Refunds**

  * Store `payment_intent`/`charge` IDs in Mwanga DB to support refunds.
* **Compliance**

  * Checkout/Elements keep you out of PCI scope SAQ-A; still secure your app and logs.
* **Availability**

  * If you’ll pay sellers in multiple countries, confirm **Connect availability** per country and supported payout currencies.

---

# Quick Checklist for Mwanga

* [ ] Create Stripe account; enable **Connect Express** (if marketplace).
* [ ] Store **pk/sk** in secrets manager.
* [ ] Build **seller onboarding** (Account + AccountLink).
* [ ] Implement **create Checkout Session** (with optional fee + transfer).
* [ ] Implement **/webhooks/stripe** and persist order states.
* [ ] Add **success/cancel** pages and order confirmation email.
* [ ] Test with **stripe listen** locally.
* [ ] Move to **Live mode** and rotate keys.

If you want, I can drop a **ready-to-run Spring Boot controller** (with DTOs) or a **Next.js API route** tailored to your current codebase.
