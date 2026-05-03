Alright, devs! Buckle up. DocAI here, bringing you the lowdown on the shiny new API v2 for `docai_smart_dkpaon23`. This isn't just a patch; it's a full-on upgrade. We're talking mandatory API key authentication, proper versioning, and a bunch of new payment wizardry. All your existing endpoints have moved under `/api/v2/` and now demand an `X-API-Key` header.

Let's dive into the specifics.

---

## API Version 2.0 Overview

This release (`v2.0`) introduces critical enhancements focusing on security, consistency, and expanded payment capabilities.

*   **Base URL:** `https://api.yourdomain.com/api/v2` (Always use HTTPS)
*   **Breaking Change:** All API requests must now be prefixed with `/api/v2/`.
*   **Authentication:** A valid API Key is **mandatory** for *all* endpoints.
*   **Standardized Responses:** All responses, both success and error, now follow a consistent JSON structure.

### Authentication

All endpoints in API v2 require authentication using an API Key.

*   **Mechanism:** Your API Secret Key must be included in every request as an `X-API-Key` HTTP header.
*   **Where to find it:** Your `API_SECRET_KEY` is provisioned through your account dashboard or environment variables.
*   **Security:** Treat your API Key as securely as you would your password. Do not expose it in client-side code, public repositories, or unsecured channels. Always use server-side integrations to call these endpoints.

    **Example Header:**
    ```
    X-API-Key: sk_live_YOUR_API_SECRET_KEY_HERE
    ```

### Standardized Response Format

To provide a consistent experience, all API v2 responses adhere to the following JSON structures:

**Success Response (HTTP 200 OK):**
```json
{
  "success": true,
  "data": {
    // Resource-specific payload
  },
  "message": "Optional, human-readable success message."
}
```

**Error Response (HTTP 4xx or 5xx):**
```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE_STRING", // A programmatic error code
    "message": "A human-readable description of the error.",
    "details": {
      // Optional: Specific validation errors or additional context
      "field_name": "Issue with this field."
    }
  }
}
```

### Rate Limits

Currently, there are no strict rate limits enforced on API v2 endpoints. However, we advise against making excessive, rapid-fire requests. Implement exponential backoff for retries to handle transient network issues and to ensure fair usage. We reserve the right to introduce rate limits if abuse is detected.

### SDK Guidance

While dedicated SDKs for API v2 are under active development, you can seamlessly interact with these endpoints using any standard HTTP client library in your preferred programming language.

*   **Key Consideration:** Ensure your HTTP client correctly sets the `X-API-Key` header for every request.
*   **Parsing Responses:** Always anticipate and parse the new standardized JSON response format (`success`, `data`/`error` objects).
*   **Client Examples:** Refer to the updated `Frontend Client Examples` (HTML, React) and `Backend Server Implementations` (Java, Node.js, Next.js, Python, Ruby) within the `payments-backend` repository for practical integration patterns.

---

## Endpoints

Here's the breakdown of the available API v2 endpoints:

### 1. `GET /api/v2/config`

*   **Summary:** Retrieves the essential configuration details for the payment system, such as the publishable key and default currency. This is primarily for client-side setup.
*   **Authentication:** Required.
*   **Request:**
    *   **Method:** `GET`
    *   **Path:** `/api/v2/config`
    *   **Headers:**
        ```
        X-API-Key: sk_live_YOUR_API_SECRET_KEY
        ```
    *   **Body:** None
*   **Example Response (200 OK):**
    ```json
    {
      "success": true,
      "data": {
        "publishableKey": "pk_test_YOUR_STRIPE_PUBLISHABLE_KEY",
        "currency": "usd"
      }
    }
    ```
*   **Example Response (401 Unauthorized):**
    ```json
    {
      "success": false,
      "error": {
        "code": "AUTH_ERROR",
        "message": "Authentication required. Please provide a valid X-API-Key header."
      }
    }
    ```

---

### 2. `POST /api/v2/create-payment-intent`

*   **Summary:** Creates a new Payment Intent on the server-side, which is the first step in processing a payment with Stripe. This endpoint now requires the `amount` and `currency` in the request body, and it has transitioned from a `GET` to a `POST` request.
*   **Authentication:** Required.
*   **Request:**
    *   **Method:** `POST`
    *   **Path:** `/api/v2/create-payment-intent`
    *   **Headers:**
        ```
        Content-Type: application/json
        X-API-Key: sk_live_YOUR_API_SECRET_KEY
        ```
    *   **Body:**
        ```json
        {
          "amount": 1099,       // Required: Amount in cents (e.g., 10.99 USD)
          "currency": "usd"     // Required: Three-letter ISO currency code (e.g., "usd", "eur")
        }
        ```
*   **Example Response (200 OK):**
    ```json
    {
      "success": true,
      "data": {
        "clientSecret": "pi_YOUR_PAYMENT_INTENT_CLIENT_SECRET_ID_secret_YOUR_SECRET"
      },
      "message": "Payment Intent created successfully."
    }
    ```
*   **Example Response (400 Bad Request - Missing Parameters):**
    ```json
    {
      "success": false,
      "error": {
        "code": "VALIDATION_ERROR",
        "message": "Missing required parameters: 'amount' and 'currency' are mandatory.",
        "details": {
          "fields": {
            "amount": "Amount must be a positive integer.",
            "currency": "Currency must be a valid 3-letter ISO code."
          }
        }
      }
    }
    ```

---

### 3. `POST /api/v2/customers`

*   **Summary:** Creates a new customer record in the payment system. This allows you to associate payment methods and payment intents with a specific user, enabling features like recurring payments or stored payment methods.
*   **Authentication:** Required.
*   **Request:**
    *   **Method:** `POST`
    *   **Path:** `/api/v2/customers`
    *   **Headers:**
        ```
        Content-Type: application/json
        X-API-Key: sk_live_YOUR_API_SECRET_KEY
        ```
    *   **Body:**
        ```json
        {
          "email": "customer@example.com",     // Optional: Customer's email address
          "name": "Jane Doe",                  // Optional: Customer's full name
          "description": "Customer for monthly subscription" // Optional: A descriptive note for the customer
        }
        ```
*   **Example Response (200 OK):**
    ```json
    {
      "success": true,
      "data": {
        "id": "cus_YOUR_CUSTOMER_ID",
        "email": "customer@example.com",
        "name": "Jane Doe",
        "created": 1678886400, // Unix timestamp
        "livemode": false
      },
      "message": "Customer created successfully."
    }
    ```
*   **Example Response (400 Bad Request - Invalid Email):**
    ```json
    {
      "success": false,
      "error": {
        "code": "VALIDATION_ERROR",
        "message": "Invalid email format provided.",
        "details": {
          "fields": {
            "email": "Must be a valid email address."
          }
        }
      }
    }
    ```

---

### 4. `GET /api/v2/payment-intent/:id`

*   **Summary:** Retrieves the current status and detailed information for a specific Payment Intent using its unique identifier. This is useful for checking the outcome of a payment attempt.
*   **Authentication:** Required.
*   **Request:**
    *   **Method:** `GET`
    *   **Path:** `/api/v2/payment-intent/{id}` (e.g., `/api/v2/payment-intent/pi_1234567890abcdefghijklmnop`)
    *   **Headers:**
        ```
        X-API-Key: sk_live_YOUR_API_SECRET_KEY
        ```
    *   **Path Parameters:**
        *   `id` (string, required): The unique identifier of the Payment Intent (e.g., `pi_12345`).
    *   **Body:** None
*   **Example Response (200 OK):**
    ```json
    {
      "success": true,
      "data": {
        "id": "pi_YOUR_PAYMENT_INTENT_ID",
        "amount": 1099,
        "currency": "usd",
        "status": "succeeded", // e.g., "succeeded", "requires_payment_method", "requires_confirmation"
        "description": "Payment for order #12345",
        "charges": {
          "object": "list",
          "data": [
            {
              "id": "ch_YOUR_CHARGE_ID",
              "amount": 1099,
              "currency": "usd",
              "status": "succeeded",
              "receipt_url": "https://pay.stripe.com/receipts/acct_YOUR_ACCT/ch_YOUR_CHARGE_ID/rcpt_YOUR_RECEIPT_ID"
            }
          ]
        }
      },
      "message": "Payment Intent retrieved successfully."
    }
    ```
*   **Example Response (404 Not Found):**
    ```json
    {
      "success": false,
      "error": {
        "code": "NOT_FOUND",
        "message": "Payment Intent with ID 'pi_invalid_id' not found."
      }
    }
    ```

---

### 5. `POST /api/v2/refunds`

*   **Summary:** Initiates a refund for a previously successful payment. You can specify a full or partial refund amount.
*   **Authentication:** Required.
*   **Request:**
    *   **Method:** `POST`
    *   **Path:** `/api/v2/refunds`
    *   **Headers:**
        ```
        Content-Type: application/json
        X-API-Key: sk_live_YOUR_API_SECRET_KEY
        ```
    *   **Body:**
        ```json
        {
          "paymentIntentId": "pi_YOUR_PAYMENT_INTENT_ID", // Required: The ID of the Payment Intent to refund
          "amount": 500,                                 // Optional: Amount in cents to refund. If omitted, full amount is refunded.
          "reason": "duplicate_payment"                  // Optional: Reason for the refund (e.g., "duplicate_payment", "fraudulent", "requested_by_customer")
        }
        ```
*   **Example Response (200 OK):**
    ```json
    {
      "success": true,
      "data": {
        "id": "re_YOUR_REFUND_ID",
        "amount": 500,
        "currency": "usd",
        "payment_intent": "pi_YOUR_PAYMENT_INTENT_ID",
        "status": "succeeded", // or "pending", "failed"
        "reason": "duplicate_payment"
      },
      "message": "Refund processed successfully."
    }
    ```
*   **Example Response (400 Bad Request - Refund Failed):**
    ```json
    {
      "success": false,
      "error": {
        "code": "REFUND_ERROR",
        "message": "Refund failed: Insufficient funds or payment already fully refunded.",
        "details": {
          "reason": "The refund amount exceeds the unrefunded balance of the Payment Intent."
        }
      }
    }
    ```

---

### 6. `GET /api/v2/orders/:id`

*   **Summary:** Retrieves the comprehensive details of a specific order using its unique ID. This includes items, total amount, shipping information, and payment intent ID.
*   **Authentication:** Required.
*   **Request:**
    *   **Method:** `GET`
    *   **Path:** `/api/v2/orders/{id}` (e.g., `/api/v2/orders/ord_YOUR_ORDER_ID_123`)
    *   **Headers:**
        ```
        X-API-Key: sk_live_YOUR_API_SECRET_KEY
        ```
    *   **Path Parameters:**
        *   `id` (string, required): The unique identifier of the Order (e.g., `ord_12345`).
    *   **Body:** None
*   **Example Response (200 OK):**
    ```json
    {
      "success": true,
      "data": {
        "id": "ord_YOUR_ORDER_ID_123",
        "customer_id": "cus_YOUR_CUSTOMER_ID",
        "status": "completed", // e.g., "pending", "processing", "shipped", "completed", "cancelled"
        "total_amount": 1099,
        "currency": "usd",
        "items": [
          {
            "item_id": "prod_SKU123",
            "name": "Premium Widget",
            "quantity": 1,
            "unit_price": 999
          },
          {
            "item_id": "serv_ADDON456",
            "name": "Extended Warranty",
            "quantity": 1,
            "unit_price": 100
          }
        ],
        "shipping_address": {
          "line1": "123 Main St",
          "city": "Anytown",
          "state": "CA",
          "postal_code": "90210",
          "country": "US"
        },
        "payment_intent_id": "pi_YOUR_PAYMENT_INTENT_ID",
        "created_at": "2023-10-27T10:00:00Z",
        "updated_at": "2023-10-27T10:15:00Z"
      },
      "message": "Order details retrieved successfully."
    }
    ```
*   **Example Response (404 Not Found):**
    ```json
    {
      "success": false,
      "error": {
        "code": "NOT_FOUND",
        "message": "Order with ID 'ord_invalid' not found."
      }
    }
    ```

---

**IMPORTANT:** Due to the breaking changes introduced in API v2, please consult the `API_V2_MIGRATION.md` guide within the repository for detailed instructions on migrating your existing integrations. Happy coding!