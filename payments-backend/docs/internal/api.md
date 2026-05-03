Welcome to the DocAI manual for `docai_manual_0earfn3m`. This repository provides a collection of example applications demonstrating various Stripe payment flows across multiple programming languages and frameworks. These examples are designed for internal development and integration testing, showcasing how to implement features like the Payment Element, prebuilt Checkout Page, and custom payment flows.

Our primary goal is to provide **REAL** endpoints as implemented within these example servers, focusing on their functionality, request/response structures, and integration best practices.

---

## General Concepts & Setup

The example servers in this repository interact with the Stripe API to process payments. All sensitive API operations are performed on the server-side using your Stripe Secret Key. Client-side interactions typically use your Stripe Publishable Key.

**Environment Variables:**
Each server example leverages a `.env` file (managed via `dotenv` dependency in some cases) to configure Stripe API keys and other settings.
Key variables include:
*   `STRIPE_SECRET_KEY`: Your Stripe secret API key (e.g., `sk_test_...`).
*   `STRIPE_PUBLISHABLE_KEY`: Your Stripe publishable API key (e.g., `pk_test_...`).
*   `STRIPE_WEBHOOK_SECRET`: The secret for verifying Stripe webhook signatures.
*   `PORT`: The port the server listens on (e.g., `4242`).

---

## Endpoints

Below are the common endpoints found across the various server-side examples (e.g., `payment-element-server-python`, `prebuilt-checkout-page-server-node`, etc.), which serve as the backbone for integrating Stripe into your applications.

### 1. `POST /create-payment-intent`

*   **Summary:** Creates a new Stripe PaymentIntent on the server, which is the foundational object for handling the payment lifecycle. This endpoint is typically called by your client-side application to initiate a payment flow.
*   **Purpose:** To generate a `client_secret` that the client-side Stripe SDK uses to confirm the payment.

**Request:**

*   **Method:** `POST`
*   **Content-Type:** `application/json`

```json
{
  "amount": 2000,           // Required: Amount in cents (e.g., 2000 for $20.00)
  "currency": "usd",        // Required: Three-letter ISO currency code
  "paymentMethodType": "card" // Optional: Specific payment method type to use (e.g., "card", "ideal", "bancontact"). Defaults to 'card'.
}
```

**Response (Success - HTTP 200 OK):**

*   **Content-Type:** `application/json`

```json
{
  "clientSecret": "pi_xxxxxxxxxxxxxxxx_secret_xxxxxxxxxxxxxxxx", // The PaymentIntent client secret for client-side confirmation
  "publishableKey": "pk_test_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx" // Your Stripe publishable key, useful for client-side initialization
}
```

**Response (Error - HTTP 400 Bad Request or 500 Internal Server Error):**

*   **Content-Type:** `application/json`

```json
{
  "error": {
    "message": "Invalid amount provided." // Descriptive error message
  }
}
```

**SDK Guidance:**
The client-side Stripe SDK (e.g., `@stripe/stripe-js` for JavaScript) uses the `clientSecret` from this endpoint to initialize and confirm payments.

*   **JavaScript (example using `fetch` and Stripe.js):**
    ```javascript
    fetch('/create-payment-intent', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        amount: 2000,
        currency: 'usd',
        paymentMethodType: 'card'
      }),
    })
    .then((res) => res.json())
    .then((data) => {
      const clientSecret = data.clientSecret;
      const stripe = Stripe(data.publishableKey); // Initialize Stripe on client side
      // Use clientSecret with Stripe.js confirmCardPayment or paymentElement.mount()
    });
    ```

### 2. `GET /config`

*   **Summary:** Retrieves public configuration details, primarily the Stripe Publishable Key, required for client-side initialization of Stripe.js.
*   **Purpose:** To allow client-side applications to fetch necessary public keys without hardcoding them.

**Request:**

*   **Method:** `GET`

```
GET /config
```

**Response (Success - HTTP 200 OK):**

*   **Content-Type:** `application/json`

```json
{
  "publishableKey": "pk_test_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx" // Your Stripe publishable key
}
```

**SDK Guidance:**
This endpoint is typically called once when your client-side application loads to get the `publishableKey` to initialize `Stripe.js`.

*   **JavaScript (example using `fetch`):**
    ```javascript
    fetch('/config')
      .then((res) => res.json())
      .then((data) => {
        const stripe = Stripe(data.publishableKey); // Initialize Stripe.js
        // ... then mount Payment Element or perform other client-side operations
      });
    ```

### 3. `POST /webhook`

*   **Summary:** Receives and processes webhook events sent by Stripe. These events notify your application of changes in object states (e.g., a payment succeeded, a charge failed, a subscription updated).
*   **Purpose:** To asynchronously update your application's state or trigger actions based on Stripe events, without relying on client-side polling.

**Request:**

*   **Method:** `POST`
*   **Content-Type:** `application/json`
*   **Headers:**
    *   `Stripe-Signature`: Contains the timestamp and one or more signatures for verification.

```json
{
  "id": "evt_xxxxxxxxxxxxxxxxxxxx",
  "object": "event",
  "api_version": "2020-08-27",
  "created": 1678886400,
  "data": {
    "object": {
      "id": "pi_xxxxxxxxxxxxxxxx",
      "object": "payment_intent",
      "status": "succeeded",
      "amount": 2000,
      "currency": "usd",
      // ... other PaymentIntent details
    }
  },
  "type": "payment_intent.succeeded" // Example event type
}
```

**Response (Success - HTTP 200 OK):**

*   **Content-Type:** `application/json`

```json
{}
```
*Note: Stripe expects a 200 OK response quickly to acknowledge receipt of the event. Any complex processing should be done asynchronously.*

**Response (Error - HTTP 400 Bad Request, 401 Unauthorized, 500 Internal Server Error):**

*   **Content-Type:** `application/json`

```json
{
  "error": "Webhook signature verification failed." // Or other error messages
}
```

**Authentication:**
Stripe webhooks are secured using a signature in the `Stripe-Signature` header. Your server **must** verify this signature using your `STRIPE_WEBHOOK_SECRET` to ensure the event genuinely came from Stripe and hasn't been tampered with.

**SDK Guidance:**
All official Stripe SDKs provide helpers for webhook signature verification.

*   **Python (Flask example):**
    ```python
    import stripe
    from flask import Flask, request, jsonify
    import os

    app = Flask(__name__)
    stripe.api_key = os.getenv('STRIPE_SECRET_KEY')
    webhook_secret = os.getenv('STRIPE_WEBHOOK_SECRET')

    @app.route('/webhook', methods=['POST'])
    def webhook_received():
        request_data = request.data
        signature = request.headers.get('stripe-signature')

        try:
            event = stripe.Webhook.construct_event(
                payload=request_data, sig_header=signature, secret=webhook_secret
            )
        except ValueError as e:
            # Invalid payload
            print(f'Error: Invalid payload: {e}')
            return jsonify({'error': 'Invalid payload'}), 400
        except stripe.error.SignatureVerificationError as e:
            # Invalid signature
            print(f'Error: Invalid signature: {e}')
            return jsonify({'error': 'Invalid signature'}), 401

        # Handle the event
        if event['type'] == 'payment_intent.succeeded':
            payment_intent = event['data']['object']
            print(f'PaymentIntent was successful for {payment_intent["amount"]} {payment_intent["currency"]}')
            # TODO: Fulfill the order, update your database, etc.
        elif event['type'] == 'payment_intent.payment_failed':
            payment_intent = event['data']['object']
            print(f'PaymentIntent failed for {payment_intent["amount"]} {payment_intent["currency"]}')
            # TODO: Notify user, log, etc.
        else:
            print(f'Unhandled event type: {event["type"]}')

        return jsonify({'status': 'success'})
    ```

---

## Authentication

Authentication for these endpoints is handled primarily through two mechanisms:

1.  **Stripe API Keys (Server-side):**
    *   **Secret Key (`STRIPE_SECRET_KEY`):** Used on the server-side to make direct calls to the Stripe API (e.g., creating PaymentIntents, retrieving objects). This key should **never** be exposed client-side. It is loaded from environment variables (e.g., `.env`).
    *   **Publishable Key (`STRIPE_PUBLISHABLE_KEY`):** Used on the client-side to initialize Stripe.js and interact with Stripe UI components (like the Payment Element). This key is safe to expose. It can be retrieved via the `/config` endpoint.

2.  **Webhook Signature Verification (for `POST /webhook`):**
    *   Stripe sends a unique signature with each webhook event in the `Stripe-Signature` header. Your server-side webhook handler must verify this signature using your `STRIPE_WEBHOOK_SECRET` (from `.env`) to confirm the request's authenticity and integrity. This is crucial for security.

---

## Rate Limits

While the example endpoints themselves do not impose specific rate limits (as they are internal examples), all calls made by your server to the **Stripe API** are subject to [Stripe's API rate limits](https://stripe.com/docs/api/rate-limits). It's important to design your applications with these limits in mind, particularly for high-volume operations or during peak times. Stripe typically allows thousands of requests per second, but excessive bursts or sustained high rates might lead to rate-limiting responses (HTTP 429).

---

## SDK Guidance

This repository demonstrates implementations across various languages. We strongly recommend using the official Stripe SDKs for your chosen language to interact with the Stripe API and handle webhook events.

*   **Python:** `stripe-python` (`stripe` import)
*   **Node.js/JavaScript:** `stripe-node` (`stripe` import)
*   **Java:** `stripe-java`
*   **Ruby:** `stripe-ruby`
*   **Go:** `stripe-go`
*   **.NET:** `stripe-dotnet`

For client-side integrations (e.g., using the Payment Element), use the Stripe.js client-side library (`@stripe/stripe-js` for JavaScript/TypeScript applications).

The example directories (e.g., `./.devcontainer/payment-element-server-python`, `./.devcontainer/custom-payment-flow-server-node`) contain working implementations using these SDKs, which serve as excellent references for integrating these endpoints.