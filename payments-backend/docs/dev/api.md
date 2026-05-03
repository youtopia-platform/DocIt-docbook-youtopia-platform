As DocAI, your expert API writer, I'm analyzing the `docai_smart_y9708hjc` repository based on the recent "Comprehensive Refactor and Expansion of Payment Examples & Dev Environment" commit.

It's important to clarify: this repository does not expose a unified public API for its own functionality. Instead, it provides a comprehensive suite of **example applications** demonstrating how to integrate Stripe's Payment Element, Custom Payment Flows, and Prebuilt Checkout Pages across various server-side technologies (Node.js, Python, Ruby, Java, Go, .NET, and a new Next.js example) and modern frontend frameworks (React with Vite, Vue with Vite, plain HTML).

The endpoints documented below represent the **common API route patterns** that these example applications implement on their respective backends. Your client-side applications (e.g., the React or Vue examples in this repo) would interact with these custom backend endpoints, which then, in turn, leverage the Stripe API on the server.

---

### **Overview of Example API Endpoint Patterns**

The repository's examples generally expose two primary server-side endpoints to facilitate Stripe Payment Element and Custom Payment Flow integrations: one for initializing a payment intent, and another for securely handling asynchronous webhook events from Stripe.

---

### **1. Create Payment Intent**

This endpoint is responsible for creating a `PaymentIntent` object on the Stripe API and returning its `client_secret` to the frontend. The `client_secret` is essential for the Stripe.js library to securely collect payment details and confirm payments on the client side without exposing your secret API keys.

*   **Summary:** Initiates a new payment transaction by creating a Stripe `PaymentIntent` object on your server. It returns the `client_secret` needed by your frontend to complete the payment securely.
*   **Method:** `POST`
*   **Example Paths (varies by language/framework):**
    *   `/create-payment-intent` (Common in many server examples)
    *   `/api/create-payment-intent` (Typical for Next.js API routes)

*   **Request Body Example:**
    ```json
    {
      "amount": 2000,
      "currency": "usd",
      "payment_method_types": ["card", "us_bank_account"] // Optional: specify payment methods
      // Additional parameters might be passed, e.g., "customer_id", "metadata"
    }
    ```
    *   **`amount`** (integer, required): The amount to charge in the smallest currency unit (e.g., 2000 for $20.00 USD).
    *   **`currency`** (string, required): Three-letter ISO currency code (e.g., "usd", "eur").
    *   **`payment_method_types`** (array of strings, optional): An array of payment method types that this PaymentIntent is allowed to use. Defaults to `["card"]`.

*   **Response Body Example:**
    ```json
    {
      "clientSecret": "pi_XXXXXXXXXXXXXXXXXXXXXXXX_secret_YYYYYYYYYYYYYYYYYYYYYYYY",
      "publishableKey": "pk_test_XXXXXXXXXXXXXXXXXXXXXXXX",
      "status": "requires_payment_method"
      // Other relevant PaymentIntent fields may be included for debugging or information
    }
    ```
    *   **`clientSecret`** (string): The client secret of the created PaymentIntent. This is critical for the frontend to interact with the PaymentIntent.
    *   **`publishableKey`** (string): Your Stripe publishable API key, used by the frontend for initializing Stripe.js.
    *   **`status`** (string): The current status of the PaymentIntent (e.g., `requires_payment_method`, `succeeded`).

*   **Authentication:**
    *   **Server-Side:** The example server-side code authenticates with Stripe using your **secret API key**. This key should be kept strictly confidential and should never be exposed to the client.
    *   **Example Endpoints:** For simplicity, these example endpoints typically do not implement user authentication. In a production application, you would secure this endpoint to prevent unauthorized PaymentIntent creation (e.g., requiring user authentication for purchase requests).

*   **Rate Limits:**
    *   These example endpoints do not have explicit rate limits configured within the repository code.
    *   **Stripe API:** Be mindful of Stripe's own API rate limits when your application scales. Design your backend to handle these limits appropriately.

*   **SDK Guidance:**
    *   **Frontend (Stripe.js):** Your frontend uses the `publishableKey` to load `Stripe.js` and the `clientSecret` to initialize the Payment Element and confirm the payment.
        ```javascript
        import { loadStripe } from '@stripe/stripe-js';

        async function initializeAndDisplayPayment() {
          const response = await fetch('/create-payment-intent', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ amount: 1099, currency: 'usd' })
          });
          const { clientSecret, publishableKey } = await response.json();

          const stripe = await loadStripe(publishableKey);
          const elements = stripe.elements({ clientSecret });
          const paymentElement = elements.create('payment');
          paymentElement.mount('#payment-element');

          // To confirm payment later (e.g., on form submission):
          // const { error } = await stripe.confirmPayment({
          //   elements,
          //   confirmParams: {
          //     return_url: 'https://your-domain.com/payment-complete',
          //   }
          // });
        }
        ```
    *   **Backend (Example Server Code):** The server implementations (e.g., Node.js, Python, Java) utilize the respective Stripe server-side SDKs (`stripe-node`, `stripe-python`, etc.) to call `stripe.paymentIntents.create()`.

---

### **2. Webhook Listener**

This endpoint is absolutely critical for handling asynchronous events from Stripe. After a payment is confirmed, fails, or other significant events occur (like refunds or subscription changes), Stripe sends an event notification to this endpoint. This allows your backend to securely react to these events, update order statuses, fulfill goods, send receipts, and handle other post-payment logic without relying on potentially unreliable client-side responses.

*   **Summary:** Receives and processes event notifications sent by Stripe, allowing your server to react reliably to changes in payment status and other critical lifecycle events.
*   **Method:** `POST`
*   **Example Paths (varies by language/framework):**
    *   `/webhook` (Common in many server examples)
    *   `/api/webhook` (Typical for Next.js API routes)

*   **Request Body Example:** A Stripe `Event` object, containing detailed information about the event that occurred.
    ```json
    {
      "id": "evt_XXXXXXXXXXXXXXXXXXXXXXXX",
      "object": "event",
      "api_version": "2020-08-27",
      "created": 1678886400,
      "data": {
        "object": {
          "id": "pi_YYYYYYYYYYYYYYYYYYYYYYYY",
          "object": "payment_intent",
          "amount": 2000,
          "currency": "usd",
          "status": "succeeded",
          "charges": {
            "data": [
              {
                "id": "ch_ZZZZZZZZZZZZZZZZZZZZZZZZ",
                "object": "charge",
                "status": "succeeded"
              }
            ]
          }
        }
      },
      "livemode": false,
      "pending_webhooks": 1,
      "request": { "id": "req_AAAAAAAAAAAAAAAAAAAAAAAA" },
      "type": "payment_intent.succeeded" // Key event type
    }
    ```
    *   The `type` field (e.g., `payment_intent.succeeded`, `checkout.session.completed`, `charge.succeeded`) indicates the specific event that triggered the webhook.
    *   The `data.object` field contains the relevant Stripe object (e.g., `PaymentIntent`, `Charge`, `CheckoutSession`) associated with the event.

*   **Response Body Example:** An empty `200 OK` response.
    ```
    (No content, HTTP Status 200)
    ```
    *   Responding with a `2xx` status code within a reasonable time (typically 30 seconds) tells Stripe you successfully received the event. Anything else may cause Stripe to retry the delivery.

*   **Authentication:**
    *   **Webhook Signature Verification:** This is absolutely critical for security. Stripe sends a `Stripe-Signature` header with each webhook event. The example server code demonstrates how to verify this signature using your **webhook secret** (obtained from your Stripe dashboard) to ensure the event genuinely originated from Stripe and hasn't been tampered with by a malicious third party.
    *   **Never process webhook events without verifying the signature.**

*   **Rate Limits:**
    *   These example endpoints do not enforce explicit rate limits.
    *   **Stripe Webhooks:** Stripe has its own internal mechanisms for retrying webhook deliveries if your endpoint doesn't respond. Design your webhook handler to be idempotent (processing the same event multiple times has the same outcome) and efficient, or offload complex processing to a background job/queue to ensure a quick response.

*   **SDK Guidance:**
    *   **Backend (Example Server Code):** The server-side examples utilize the Stripe SDK's webhook construction methods to parse the raw request body and verify the signature against your endpoint secret.
        ```javascript
        // Example for Node.js (using Express)
        const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);
        const webhookSecret = process.env.STRIPE_WEBHOOK_SECRET; // Your webhook secret

        app.post('/webhook', express.raw({ type: 'application/json' }), (request, response) => {
          const signature = request.headers['stripe-signature'];
          let event;

          try {
            event = stripe.webhooks.constructEvent(request.body, signature, webhookSecret);
          } catch (err) {
            console.error(`Webhook signature verification failed: ${err.message}`);
            return response.sendStatus(400); // Invalid signature
          }

          // Handle the event based on its type
          switch (event.type) {
            case 'payment_intent.succeeded':
              const paymentIntent = event.data.object;
              console.log(`PaymentIntent ${paymentIntent.id} succeeded for ${paymentIntent.amount}!`);
              // ✅ Implement your business logic here (e.g., fulfill order, update database, send confirmation email)
              break;
            case 'payment_intent.payment_failed':
              const failedPaymentIntent = event.data.object;
              console.log(`PaymentIntent ${failedPaymentIntent.id} failed: ${failedPaymentIntent.last_payment_error.message}`);
              // ⚠️ Handle failed payment (e.g., notify user, log error, retry payment)
              break;
            // ... handle other relevant event types (e.g., 'charge.succeeded', 'checkout.session.completed')
            default:
              console.log(`Unhandled event type ${event.type}.`);
          }

          // Respond with a 200 to acknowledge successful receipt of the event
          response.sendStatus(200);
        });
        ```

---

The comprehensive refactor outlined in the commit provides developers with these robust example patterns in multiple languages and frameworks, significantly improving the utility and developer experience for building modern Stripe integrations. The introduction of the Next.js server example and modernization with Vite tooling are particularly valuable for contemporary full-stack development.