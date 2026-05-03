Alright team, let's dive into the core endpoints powering the `docai_manual_xye9t5fi` repository's Stripe integrations. This project is a goldmine for understanding how to integrate Stripe's Payment Element and prebuilt Checkout Page across various backend languages. While the repository supports multiple server-side implementations (Python, Node, Java, Ruby, Go, .NET), I'll focus on the common patterns and specifically detail examples from the **Python/Flask** implementation, as `Flask` is explicitly identified as a framework in use.

These endpoints are designed to be consumed by client-side applications (like the React/Vue clients also present in this repo) to facilitate secure and compliant payment flows.

---

## Endpoint Documentation

### 1. GET `/config`

This endpoint provides the necessary public configuration for the client-side application, primarily the Stripe publishable key, allowing the client to initialize Stripe.js.

*   **Summary**: Retrieves the server's public configuration, including the Stripe publishable key.
*   **Method**: `GET`
*   **URL**: `/config`
*   **Request Example**:
    ```http
    GET /config HTTP/1.1
    Host: localhost:4242
    ```
*   **Response Example**:
    ```json
    HTTP/1.1 200 OK
    Content-Type: application/json

    {
      "publishableKey": "pk_test_YOUR_STRIPE_PUBLISHABLE_KEY"
    }
    ```
*   **Authentication**: None. This endpoint serves public information.
*   **Rate Limits**: Standard web server rate limits apply. Stripe API limits are not directly applicable here.
*   **SDK Guidance**:
    *   **Client-side (e.g., JavaScript/React/Vue)**: Call this endpoint to fetch `publishableKey` before initializing `Stripe` object from `@stripe/stripe-js`.
        ```javascript
        import { loadStripe } from '@stripe/stripe-js';

        async function initStripe() {
          const response = await fetch('/config');
          const { publishableKey } = await response.json();
          const stripe = await loadStripe(publishableKey);
          // Use 'stripe' object for further operations
          return stripe;
        }
        ```

---

### 2. POST `/create-payment-intent`

This endpoint is used for integrating with Stripe's Payment Element. It creates a `PaymentIntent` on the server, which is then used client-side to render and confirm payment details without exposing sensitive API keys.

*   **Summary**: Creates a Stripe PaymentIntent and returns its `client_secret` to the client. This is crucial for rendering the Payment Element and confirming payments.
*   **Method**: `POST`
*   **URL**: `/create-payment-intent`
*   **Request Example**:
    ```http
    POST /create-payment-intent HTTP/1.1
    Host: localhost:4242
    Content-Type: application/json

    {
      "items": [{ "id": "prod_12345" }],
      "currency": "usd",
      "paymentMethodType": "card"
      // Additional parameters like 'amount', 'metadata', 'setup_future_usage' might be supported
    }
    ```
    *Note: The actual request body might vary slightly depending on the specific server implementation (e.g., if it calculates amount based on 'items' or expects 'amount' directly).*
*   **Response Example**:
    ```json
    HTTP/1.1 200 OK
    Content-Type: application/json

    {
      "clientSecret": "pi_YOUR_PAYMENT_INTENT_CLIENT_SECRET_TOKEN"
    }
    ```
*   **Authentication**: This endpoint is usually protected by server-side logic to ensure valid requests, often leveraging session management or basic API key checks in production, though examples might be simplified. The Stripe secret key is used on the server, not exposed.
*   **Rate Limits**: Standard web server rate limits apply. Stripe API calls for `PaymentIntent` creation are subject to Stripe's rate limits (e.g., 200 writes/second for PaymentIntents).
*   **SDK Guidance**:
    *   **Server-side (Python/Flask)**: Uses `stripe.PaymentIntent.create()` with `amount`, `currency`, and `automatic_payment_methods`.
        ```python
        import stripe
        from flask import Flask, jsonify, request

        # ... app setup ...

        @app.route('/create-payment-intent', methods=['POST'])
        def create_payment_intent():
            try:
                data = request.get_json()
                # Calculate amount based on data['items'] or receive it directly
                amount = 2000 # example amount for $20.00
                payment_intent = stripe.PaymentIntent.create(
                    amount=amount,
                    currency=data.get('currency', 'usd'),
                    automatic_payment_methods={'enabled': True},
                    # Add additional parameters like 'setup_future_usage' if needed
                )
                return jsonify({'clientSecret': payment_intent.client_secret})
            except Exception as e:
                return jsonify(error=str(e)), 400
        ```
    *   **Client-side (e.g., JavaScript/React/Vue)**: After fetching `clientSecret` from this endpoint, pass it to `elements.create('payment')` or `stripe.confirmPayment()`.
        ```javascript
        import { Elements } from '@stripe/react-stripe-js';

        // ... in a React component ...
        const response = await fetch('/create-payment-intent', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ items: [{ id: 'prod_123' }] }),
        });
        const { clientSecret } = await response.json();

        // Use clientSecret to initialize Payment Element or confirm payment
        const appearance = { theme: 'stripe' };
        const options = { clientSecret, appearance };
        // <Elements options={options} stripe={stripePromise}> ... </Elements>
        ```

---

### 3. POST `/create-checkout-session`

This endpoint is dedicated to integrating with Stripe's prebuilt Checkout Page. It creates a `Checkout Session` on the server, which then redirects the customer to a Stripe-hosted payment page.

*   **Summary**: Creates a Stripe Checkout Session and returns its `id` to the client. The client then redirects the user to the Stripe-hosted Checkout Page using this session ID.
*   **Method**: `POST`
*   **URL**: `/create-checkout-session`
*   **Request Example**:
    ```http
    POST /create-checkout-session HTTP/1.1
    Host: localhost:4242
    Content-Type: application/json

    {
      "items": [{ "price": "price_12345", "quantity": 1 }],
      "success_url": "http://localhost:4242/success",
      "cancel_url": "http://localhost:4242/cancel"
      // Additional parameters like 'mode' (payment, subscription, setup), 'customer_email'
    }
    ```
    *Note: The actual request body might vary. Common parameters include `line_items`, `mode`, `success_url`, `cancel_url`.*
*   **Response Example**:
    ```json
    HTTP/1.1 200 OK
    Content-Type: application/json

    {
      "id": "cs_YOUR_CHECKOUT_SESSION_ID"
    }
    ```
*   **Authentication**: Similar to `/create-payment-intent`, usually protected by server-side logic. The Stripe secret key is used on the server.
*   **Rate Limits**: Standard web server rate limits apply. Stripe API calls for `Checkout Session` creation are subject to Stripe's rate limits.
*   **SDK Guidance**:
    *   **Server-side (Python/Flask)**: Uses `stripe.checkout.Session.create()` with `line_items`, `mode`, `success_url`, and `cancel_url`.
        ```python
        import stripe
        from flask import Flask, jsonify, request, redirect

        # ... app setup ...

        @app.route('/create-checkout-session', methods=['POST'])
        def create_checkout_session():
            try:
                data = request.get_json()
                # Example line items - adjust based on your product data
                line_items = [
                    {
                        'price': data['items'][0]['price'], # e.g., 'price_123'
                        'quantity': data['items'][0]['quantity'],
                    }
                ]
                checkout_session = stripe.checkout.Session.create(
                    line_items=line_items,
                    mode='payment', # or 'subscription', 'setup'
                    success_url=data.get('success_url', 'http://localhost:4242/success?session_id={CHECKOUT_SESSION_ID}'),
                    cancel_url=data.get('cancel_url', 'http://localhost:4242/cancel'),
                    # Add customer_email or customer_creation if applicable
                )
                return jsonify({'id': checkout_session.id})
            except Exception as e:
                return jsonify(error=str(e)), 400
        ```
    *   **Client-side (e.g., JavaScript/React/Vue)**: After fetching `id` from this endpoint, redirect the user using `stripe.redirectToCheckout()`.
        ```javascript
        // ... assuming stripePromise is loaded ...
        const stripe = await stripePromise;

        const response = await fetch('/create-checkout-session', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
                items: [{ price: 'price_123', quantity: 1 }],
                success_url: window.location.origin + '/success',
                cancel_url: window.location.origin + '/cancel',
            }),
        });
        const { id: sessionId } = await response.json();

        const { error } = await stripe.redirectToCheckout({ sessionId });
        if (error) {
            console.error('Error redirecting to Checkout:', error);
        }
        ```

---

### 4. POST `/webhook`

This endpoint is crucial for asynchronous event handling from Stripe. Stripe sends events (like `payment_intent.succeeded`, `checkout.session.completed`, `customer.subscription.created`) to this URL, allowing the server to update its database, fulfill orders, or send notifications.

*   **Summary**: Receives and processes events pushed by Stripe.
*   **Method**: `POST`
*   **URL**: `/webhook`
*   **Request Example**:
    ```http
    POST /webhook HTTP/1.1
    Host: localhost:4242
    Content-Type: application/json
    Stripe-Signature: t=1678886400,v1=YOUR_STRIPE_SIGNATURE

    {
      "id": "evt_12345",
      "object": "event",
      "api_version": "2020-08-27",
      "created": 1678886400,
      "data": {
        "object": {
          "id": "pi_XYZ",
          "object": "payment_intent",
          "status": "succeeded",
          "amount": 2000,
          "currency": "usd",
          // ... other PaymentIntent data
        }
      },
      "livemode": false,
      "pending_webhooks": 1,
      "request": {
        "id": null,
        "idempotency_key": null
      },
      "type": "payment_intent.succeeded"
    }
    ```
*   **Response Example**:
    ```json
    HTTP/1.1 200 OK
    Content-Type: application/json

    {
      "received": true
    }
    ```
    *Note: A 200 OK response tells Stripe the webhook was received. The actual processing might happen asynchronously.*
*   **Authentication**: Stripe uses a digital signature (`Stripe-Signature` header) to verify the authenticity and integrity of webhook events. The server must validate this signature using a webhook secret key.
*   **Rate Limits**: Stripe sends webhooks at a high volume. Your server needs to be robust. Stripe has retry mechanisms for failed deliveries.
*   **SDK Guidance**:
    *   **Server-side (Python/Flask)**: Uses `stripe.Webhook.construct_event()` to verify the signature and parse the event.
        ```python
        import stripe
        import os
        from flask import Flask, jsonify, request

        # ... app setup ...

        # Load webhook secret from environment variables
        WEBHOOK_SECRET = os.getenv('STRIPE_WEBHOOK_SECRET')

        @app.route('/webhook', methods=['POST'])
        def stripe_webhook():
            payload = request.get_data(as_text=True)
            sig_header = request.headers.get('Stripe-Signature')
            event = None

            try:
                event = stripe.Webhook.construct_event(
                    payload, sig_header, WEBHOOK_SECRET
                )
            except ValueError as e:
                # Invalid payload
                return jsonify(error='Invalid payload'), 400
            except stripe.error.SignatureVerificationError as e:
                # Invalid signature
                return jsonify(error='Invalid signature'), 400

            # Handle the event
            if event['type'] == 'payment_intent.succeeded':
                payment_intent = event['data']['object']
                print(f"PaymentIntent was successful for {payment_intent['amount'] / 100} {payment_intent['currency'].upper()}")
                # TODO: Fulfill the order, update your database, etc.
            elif event['type'] == 'payment_method.attached':
                payment_method = event['data']['object']
                print(f"PaymentMethod {payment_method['id']} was attached to Customer {payment_method['customer']}")
                # TODO: Associate PM with customer in your system
            elif event['type'] == 'checkout.session.completed':
                session = event['data']['object']
                print(f"Checkout Session {session['id']} completed. Customer: {session.get('customer_details', {}).get('email')}")
                # TODO: Fulfill order, provision access, send receipt
            else:
                print(f"Unhandled event type {event['type']}")

            return jsonify(received=True)
        ```