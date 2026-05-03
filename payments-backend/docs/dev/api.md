Hello fellow devs! I'm DocAI, your guide to the nitty-gritty of the `docai_manual_yogn4xpx` repository's backend. This project serves as a robust collection of payment integration examples, heavily leveraging Stripe. You'll find implementations for various payment flows across multiple languages and frameworks.

Let's dive into some of the core server-side endpoints you'll encounter, particularly in the Python/Flask examples, which are crucial for driving the client-side payment experiences.

---

## API Overview

The endpoints documented here facilitate secure payment processing via Stripe, handling everything from creating payment intents/checkout sessions to processing asynchronous webhook events. These are fundamental for any modern e-commerce or service platform requiring robust payment integration.

---

## Authentication

All server-side interactions with the Stripe API require **Stripe Secret API Keys**. These keys grant your server applications permission to create and manage Stripe resources.

*   **Method**: API Key in the server-side code.
*   **Security**: Never expose your Stripe Secret API Key in client-side code. It should be loaded securely on the server, typically from environment variables, as suggested by the `.env` patterns often used with `python-dotenv`.

    ```python
    # Example (Python/Flask)
    import os
    import stripe
    from dotenv import load_dotenv

    load_dotenv() # Load environment variables from .env

    # Your Stripe secret key
    stripe.api_key = os.environ.get('STRIPE_SECRET_KEY')

    # ... use stripe.api_key for operations
    ```

---

## Rate Limits

These endpoints typically proxy calls to Stripe's API. Stripe enforces rate limits to prevent abuse and ensure stability. While your specific server might have its own limits, expect Stripe's limits to apply to the underlying API calls.

*   **Stripe API Rate Limits**: Generally, Stripe allows up to **100 read requests per second** and **100 write requests per second**. Bursts are often tolerated, but sustained high rates might lead to `429 Too Many Requests` responses from Stripe.
*   **Best Practice**: Implement retry logic with exponential backoff for Stripe API calls in your server code to handle temporary rate limit exceedances gracefully.

---

## SDK Guidance

This repository inherently demonstrates the use of Stripe's official SDKs across various languages. When integrating these endpoints into your own applications, it is **highly recommended** to use the official Stripe SDKs for your chosen server-side language.

*   **Benefits**:
    *   Simplified API calls.
    *   Automatic request signing and handling of API versions.
    *   Built-in error handling and type safety.
    *   Webhook signature verification utilities.

    **Example (Python)**:
    ```bash
    pip install stripe
    ```

    **Example (JavaScript/Node.js)**:
    ```bash
    npm install stripe
    ```

    **Example (Java)**:
    ```xml
    <!-- Maven -->
    <dependency>
      <groupId>com.stripe</groupId>
      <artifactId>stripe-java</artifactId>
      <version>20.x.x</version>
    </dependency>
    ```
    Similar SDKs exist for Ruby, Go, PHP, .NET, etc.

---

## Endpoints

Here are some real-world endpoints likely implemented within the `docai_manual_yogn4xpx` repository, especially in the Flask-based server examples.

---

### 1. Create Payment Intent

This endpoint is typically used in "Payment Element" or "Custom Payment Flow" integrations to create a `PaymentIntent` on the server-side, which is then used by the client to confirm a payment.

*   **Summary**: Creates a Stripe `PaymentIntent` object. A `PaymentIntent` tracks the lifecycle of a customer's payment attempt. The `client_secret` returned by this endpoint is essential for completing the payment on the client side using Stripe.js.
*   **Method**: `POST`
*   **URL**: `/create-payment-intent`

#### Request

*   **Headers**:
    *   `Content-Type: application/json`
*   **Body**: JSON object containing payment details.

    ```json
    {
      "amount": 2000,           // Required: Amount in cents/smallest currency unit (e.g., $20.00)
      "currency": "usd",        // Required: Three-letter ISO currency code
      "customer_id": "cus_N5dJ..." // Optional: ID of an existing Stripe Customer
    }
    ```

#### Response

*   **Status**: `200 OK` on success, `400 Bad Request` or `500 Internal Server Error` on failure.
*   **Body**: JSON object containing the `client_secret` of the created Payment Intent, along with its `id` and `status`.

    ```json
    {
      "client_secret": "pi_3O7gD22eRvN1nJzJ0B8sFpWj_secret_hVp0R6M...",
      "id": "pi_3O7gD22eRvN1nJzJ0B8sFpWj",
      "status": "requires_payment_method"
    }
    ```

#### Example (Python/Flask Server)

```python
# .devcontainer/payment-element-server-python/server.py (Conceptual Snippet)
from flask import Flask, jsonify, request
import stripe
import os
from dotenv import load_dotenv

load_dotenv()
stripe.api_key = os.environ.get('STRIPE_SECRET_KEY')

app = Flask(__name__)

@app.route('/create-payment-intent', methods=['POST'])
def create_payment_intent():
    try:
        data = request.get_json()
        amount = data['amount']
        currency = data.get('currency', 'usd') # Default to USD
        customer_id = data.get('customer_id')

        payment_intent_params = {
            'amount': amount,
            'currency': currency,
            'automatic_payment_methods': {
                'enabled': True,
            },
        }

        if customer_id:
            payment_intent_params['customer'] = customer_id

        payment_intent = stripe.PaymentIntent.create(**payment_intent_params)

        return jsonify({
            'client_secret': payment_intent.client_secret,
            'id': payment_intent.id,
            'status': payment_intent.status
        })
    except stripe.error.StripeError as e:
        return jsonify(error={'message': str(e)}), 400
    except Exception as e:
        return jsonify(error={'message': str(e)}), 500

if __name__ == '__main__':
    app.run(port=4242)
```

#### Example (cURL Request)

```bash
curl -X POST -H "Content-Type: application/json" \
     -d '{"amount": 2000, "currency": "usd"}' \
     http://localhost:4242/create-payment-intent
```

---

### 2. Create Checkout Session

This endpoint is typically used for "Prebuilt Checkout Page" integrations, where Stripe hosts the entire payment form. Your server creates a `Checkout Session`, and the client redirects the user to Stripe's hosted page.

*   **Summary**: Creates a Stripe `Checkout Session` object. This session represents a customer's intention to pay for something. After creation, the client typically redirects the customer to the `session.url` to complete the payment on Stripe's hosted checkout page.
*   **Method**: `POST`
*   **URL**: `/create-checkout-session`

#### Request

*   **Headers**:
    *   `Content-Type: application/json`
*   **Body**: JSON object specifying line items, success/cancel URLs, and mode.

    ```json
    {
      "items": [ // Required: List of items the customer is purchasing
        {
          "price_data": {
            "currency": "usd",
            "product_data": {
              "name": "T-shirt",
              "images": ["https://example.com/t-shirt.png"]
            },
            "unit_amount": 2000
          },
          "quantity": 1
        }
      ],
      "success_url": "http://localhost:4242/success?session_id={CHECKOUT_SESSION_ID}", // Required: URL to redirect after successful payment
      "cancel_url": "http://localhost:4242/cancel",                                 // Required: URL to redirect if customer cancels
      "mode": "payment",                                                              // Required: 'payment', 'subscription', or 'setup'
      "customer_email": "customer@example.com"                                        // Optional: Pre-fill customer email
    }
    ```

#### Response

*   **Status**: `200 OK` on success, `400 Bad Request` or `500 Internal Server Error` on failure.
*   **Body**: JSON object containing the `session.id` and `session.url` for redirection.

    ```json
    {
      "id": "cs_test_a1O9eFfWc...",
      "url": "https://checkout.stripe.com/c/pay/cs_test_a1O9eFfWc..."
    }
    ```

#### Example (Python/Flask Server)

```python
# .devcontainer/prebuilt-checkout-page-server-python/server.py (Conceptual Snippet)
from flask import Flask, jsonify, request, redirect
import stripe
import os
from dotenv import load_dotenv

load_dotenv()
stripe.api_key = os.environ.get('STRIPE_SECRET_KEY')

app = Flask(__name__)

@app.route('/create-checkout-session', methods=['POST'])
def create_checkout_session():
    try:
        data = request.get_json()
        line_items = data['items']
        success_url = data.get('success_url', 'http://localhost:4242/success?session_id={CHECKOUT_SESSION_ID}')
        cancel_url = data.get('cancel_url', 'http://localhost:4242/cancel')
        mode = data.get('mode', 'payment')
        customer_email = data.get('customer_email')

        checkout_session_params = {
            'line_items': line_items,
            'mode': mode,
            'success_url': success_url,
            'cancel_url': cancel_url,
        }

        if customer_email:
            checkout_session_params['customer_email'] = customer_email

        checkout_session = stripe.checkout.Session.create(**checkout_session_params)

        return jsonify({'id': checkout_session.id, 'url': checkout_session.url})
    except stripe.error.StripeError as e:
        return jsonify(error={'message': str(e)}), 400
    except Exception as e:
        return jsonify(error={'message': str(e)}), 500

# ... other routes like /success, /cancel
if __name__ == '__main__':
    app.run(port=4242)
```

#### Example (cURL Request)

```bash
curl -X POST -H "Content-Type: application/json" \
     -d '{
           "items": [
             {
               "price_data": {
                 "currency": "usd",
                 "product_data": { "name": "Test Item" },
                 "unit_amount": 1500
               },
               "quantity": 1
             }
           ],
           "success_url": "http://localhost:4242/success?session_id={CHECKOUT_SESSION_ID}",
           "cancel_url": "http://localhost:4242/cancel",
           "mode": "payment"
         }' \
     http://localhost:4242/create-checkout-session
```

---

### 3. Webhook Handler

This endpoint is crucial for handling asynchronous events from Stripe (e.g., `payment_intent.succeeded`, `checkout.session.completed`). Stripe will send `POST` requests to this endpoint when significant events occur in your account.

*   **Summary**: Receives and processes webhook events sent by Stripe. This allows your application to react to payment successes, failures, refunds, and other important lifecycle events without constant polling. **Crucially, it includes signature verification to ensure the request genuinely originated from Stripe.**
*   **Method**: `POST`
*   **URL**: `/webhook`

#### Request

*   **Headers**:
    *   `Stripe-Signature`: Contains the timestamp and signatures used to verify the webhook's authenticity.
*   **Body**: Raw JSON event payload from Stripe.

    ```json
    {
      "id": "evt_12345...",
      "object": "event",
      "api_version": "2020-08-27",
      "created": 1678886400,
      "data": {
        "object": {
          "id": "pi_12345...",
          "object": "payment_intent",
          "amount": 2000,
          "currency": "usd",
          "status": "succeeded",
          "charges": {
            "data": [
              {
                "id": "ch_12345...",
                "status": "succeeded"
              }
            ]
          }
        }
      },
      "livemode": false,
      "pending_webhooks": 1,
      "request": {
        "id": "req_ABCDE...",
        "idempotency_key": "..."
      },
      "type": "payment_intent.succeeded"
    }
    ```

#### Response

*   **Status**: `200 OK` (Always return 200 to acknowledge receipt of the event, even if you don't process it. Stripe will retry if it doesn't receive a 2xx response.)
*   **Body**: Typically empty or a simple acknowledgement.

    ```json
    {}
    ```

#### Example (Python/Flask Server)

```python
# .devcontainer/payment-element-server-python/server.py (Conceptual Snippet)
from flask import Flask, jsonify, request
import stripe
import os
from dotenv import load_dotenv

load_dotenv()
stripe.api_key = os.environ.get('STRIPE_SECRET_KEY')

# Your Stripe webhook secret, stored securely
STRIPE_WEBHOOK_SECRET = os.environ.get('STRIPE_WEBHOOK_SECRET')

app = Flask(__name__)

@app.route('/webhook', methods=['POST'])
def webhook_received():
    event = None
    payload = request.data
    sig_header = request.headers.get('stripe-signature')

    try:
        event = stripe.Webhook.construct_event(
            payload, sig_header, STRIPE_WEBHOOK_SECRET
        )
    except ValueError as e:
        # Invalid payload
        return jsonify(error={'message': str(e)}), 400
    except stripe.error.SignatureVerificationError as e:
        # Invalid signature
        return jsonify(error={'message': str(e)}), 400

    # Handle the event
    if event['type'] == 'payment_intent.succeeded':
        payment_intent = event['data']['object'] # contains a stripe.PaymentIntent
        print('PaymentIntent was successful for:', payment_intent['amount'], payment_intent['currency'])
        # TODO: Fulfill the customer's order here
    elif event['type'] == 'payment_method.attached':
        payment_method = event['data']['object'] # contains a stripe.PaymentMethod
        print('PaymentMethod was attached to a Customer:', payment_method['id'])
    elif event['type'] == 'checkout.session.completed':
        session = event['data']['object'] # contains a stripe.checkout.Session
        print('Checkout Session completed for:', session['id'])
        # TODO: Fulfill the customer's order here, often by retrieving full session details
        # session = stripe.checkout.Session.retrieve(session.id, expand=['line_items'])
    # ... handle other event types

    return jsonify(success=True)

if __name__ == '__main__':
    app.run(port=4242)
```

#### Example (Stripe CLI for Testing)

*   **Forward events to your local server**:
    ```bash
    stripe listen --forward-to localhost:4242/webhook
    ```
*   **Trigger a test event**:
    ```bash
    stripe trigger payment_intent.succeeded
    ```
    This will send a `payment_intent.succeeded` event to your `/webhook` endpoint.

---

This covers the essential server-side endpoints for integrating Stripe payments. Dive into the specific language folders (e.g., `./.devcontainer/payment-element-server-python`) for full, runnable examples of these implementations!