# Migration Guide

*Updated: 2026-05-03*

This document outlines the necessary steps to migrate your existing integrations to the new API v2, which introduces enhanced security, improved consistency, and expanded payment functionalities. Please read this guide thoroughly before commencing your migration.

---

# API v2 Migration Guide: Authentication and Extended Payment Features

## 1. What Changed and Why

This release introduces API v2, a significant upgrade designed to improve security, standardize API interactions, and offer a richer set of payment-related features. This is a **breaking change** and requires updates to all clients consuming our API.

### Key Breaking Changes:

1.  **API Versioning (`/api/v2/` Prefix):**
    *   **What:** All API endpoints are now prefixed with `/api/v2/`. For example, `/api/create-payment-intent` becomes `/api/v2/create-payment-intent`.
    *   **Why:** To allow for concurrent versions of the API, enabling backward compatibility during future upgrades and a clear distinction for consumers between different API contracts.

2.  **Mandatory API Key Authentication:**
    *   **What:** All API endpoints now require mandatory authentication via an `X-API-Key` header. This key is validated against an `API_SECRET_KEY` configured on the backend.
    *   **Why:** To significantly enhance API security, preventing unauthorized access and ensuring that only legitimate applications can interact with our services.

3.  **`create-payment-intent` Endpoint Modification:**
    *   **What:** The `create-payment-intent` endpoint has changed from a `GET` request to a `POST` request. Instead of query parameters, it now requires `amount` and `currency` to be sent in the request body as JSON.
    *   **Why:** `POST` is the semantically correct HTTP method for creating resources that involve sending sensitive or complex data. Moving parameters to the request body improves security (not logged in server access logs by default) and flexibility for future expansions.

4.  **Standardized API Response Structure:**
    *   **What:** All API responses now follow a consistent JSON structure:
        ```json
        {
          "success": boolean,
          "data": object | null, // Contains the successful response payload
          "error": {
            "code": string,
            "message": string,
            "details": object | null // Optional additional error details
          } | null // Contains error details if success is false
        }
        ```
    *   **Why:** To provide a predictable and consistent way for clients to handle API responses, distinguish between successful operations and errors, and simplify error handling logic.

### New Features Introduced:

Beyond the breaking changes, API v2 expands our payment capabilities significantly:

*   **`POST /api/v2/customers`**: Create a new customer record.
*   **`GET /api/v2/payment-intent/:id`**: Retrieve the status and details of a specific payment intent.
*   **`POST /api/v2/refunds`**: Process a refund for a previously captured payment.
*   **`GET /api/v2/orders/:id`**: Retrieve details for a specific order.
*   **`POST /api/v2/payment-intent/:id/confirm`**: Explicitly confirm a payment intent, often used in scenarios requiring 3D Secure or other post-creation steps.

**Why These Changes?**
This comprehensive API upgrade is driven by a need for:
*   **Enhanced Security:** Mandatory API key authentication protects sensitive payment operations.
*   **Improved Developer Experience:** Consistent API design, versioning, and standardized responses make integration more predictable and easier to maintain.
*   **Expanded Functionality:** New endpoints provide a more complete and robust payment processing suite, allowing for advanced workflows like refunds, explicit confirmation, and customer management directly through the API.

## 2. Step-by-Step Migration Instructions

Follow these steps to migrate your client applications to API v2.

### Prerequisites:

*   **Obtain Your API Key:** Contact your administrator or project lead to acquire your new `API_SECRET_KEY`. This key is essential for all API v2 interactions.
*   **Local Development Environment:** Ensure your development environment is set up and ready to make code changes and test against the new API v2 endpoint.

### Phase 1: Environment Setup

1.  **Update Environment Variables:**
    *   Locate your project's environment configuration (e.g., `.env` file, configuration service).
    *   Add or update the `API_SECRET_KEY` variable with the new API key you obtained.
    *   **Example (`.env`):**
        ```
        # Previous (if any)
        # API_BASE_URL=https://api.yourdomain.com/api

        # New API Key for v2
        API_SECRET_KEY=sk_your_unique_api_key_here
        ```
    *   Ensure your application can securely access this environment variable.

### Phase 2: Update API Endpoints and Authentication

1.  **Prefix All API Calls with `/api/v2/`:**
    *   Identify all locations in your codebase where you make calls to our backend API.
    *   Modify the base path for these calls from `/api/` (or similar) to `/api/v2/`.
    *   **Example:**
        *   `GET /api/status` becomes `GET /api/v2/status`
        *   `POST /api/some-resource` becomes `POST /api/v2/some-resource`

2.  **Add `X-API-Key` Header to All Requests:**
    *   For *every* API call you make to our backend (including the `create-payment-intent` endpoint), you must now include an `X-API-Key` header with your `API_SECRET_KEY` as its value.
    *   **Example (JavaScript `fetch`):**
        ```javascript
        const API_KEY = process.env.API_SECRET_KEY; // Or retrieve securely from your config

        fetch('/api/v2/your-endpoint', {
          method: 'GET', // or POST, PUT, DELETE
          headers: {
            'X-API-Key': API_KEY, // ADD THIS HEADER
            'Content-Type': 'application/json' // if sending a body
          }
          // ... other options
        });
        ```

### Phase 3: Refactor `create-payment-intent` Endpoint

1.  **Change HTTP Method from `GET` to `POST`:**
    *   Locate the code that calls the `create-payment-intent` endpoint.
    *   Change the HTTP method from `GET` to `POST`.

2.  **Move Parameters to Request Body:**
    *   Instead of sending `amount` and `currency` as URL query parameters, serialize them into a JSON object and send them in the request body.
    *   Ensure the `Content-Type` header is set to `application/json`.

### Phase 4: Adapt to New Response Structure

1.  **Update Response Handling Logic:**
    *   Modify your code that processes API responses to expect the new standardized `{ success, data, error }` format.
    *   Check the `success` boolean to determine if the operation was successful.
    *   Access the actual data from the `data` field for successful responses.
    *   Access error details (code, message, optional details) from the `error` field for failed responses.

### Phase 5: Integrate New Functionality (Optional but Recommended)

Once the core migration is complete, consider integrating the new API v2 features to leverage the extended capabilities:

1.  **Customer Management:** Use `POST /api/v2/customers` to create and manage customer profiles.
2.  **Payment Intent Retrieval:** Utilize `GET /api/v2/payment-intent/:id` to fetch the status of payment intents for enhanced UI and backend logic.
3.  **Refund Processing:** Implement `POST /api/v2/refunds` for full or partial refunds directly through the API.
4.  **Order Details:** Use `GET /api/v2/orders/:id` to retrieve comprehensive order information.
5.  **Payment Intent Confirmation:** Integrate `POST /api/v2/payment-intent/:id/confirm` for explicit confirmation workflows, especially relevant for specific payment methods.

### Phase 6: Testing

1.  **Thorough Testing:** After making all changes, rigorously test all API interactions in your application.
    *   Verify successful payment intent creation.
    *   Test error handling for various scenarios (e.g., invalid amount, missing API key).
    *   Test any newly integrated features.
    *   Check for correct data parsing from the `data` field and error message parsing from the `error` field.

## 3. Code Examples (Before/After)

### 3.1. `create-payment-intent` Endpoint

**Before (API v1 - GET Request with Query Params):**

```javascript
// Example: Client-side JavaScript
const amount = 1000; // in cents
const currency = 'usd';

fetch(`/api/create-payment-intent?amount=${amount}&currency=${currency}`)
  .then(response => response.json())
  .then(data => {
    // Assuming data directly contains the client secret or similar
    console.log('Payment Intent Client Secret (v1):', data.clientSecret);
  })
  .catch(error => {
    console.error('Error creating payment intent (v1):', error);
  });
```

**After (API v2 - POST Request with Body and Authentication):**

```javascript
// Example: Client-side JavaScript
const API_KEY = 'sk_your_unique_api_key_here'; // IMPORTANT: In a real app, retrieve securely, e.g., from an environment variable or a secure server-side proxy
const amount = 1000; // in cents
const currency = 'usd';

fetch('/api/v2/create-payment-intent', {
  method: 'POST',
  headers: {
    'X-API-Key': API_KEY, // Mandatory API Key for v2
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    amount: amount,
    currency: currency
  })
})
  .then(response => response.json())
  .then(json => {
    if (json.success) {
      // Access data from the 'data' field
      console.log('Payment Intent Client Secret (v2):', json.data.clientSecret);
      // Example: Initialize Stripe.js with the client secret
      // stripe.initPaymentElement({ clientSecret: json.data.clientSecret });
    } else {
      // Handle error from the 'error' field
      console.error('Error creating payment intent (v2):', json.error.message, json.error.details);
    }
  })
  .catch(error => {
    console.error('Network or parsing error (v2):', error);
  });
```

### 3.2. General API Call with Authentication (Example: Retrieve Payment Intent)

```javascript
// Example: Client-side JavaScript
const API_KEY = 'sk_your_unique_api_key_here'; // Retrieve securely
const paymentIntentId = 'pi_xyz123abc';

fetch(`/api/v2/payment-intent/${paymentIntentId}`, {
  method: 'GET',
  headers: {
    'X-API-Key': API_KEY, // Mandatory API Key for v2
    'Content-Type': 'application/json' // Good practice, even for GET
  }
})
  .then(response => response.json())
  .then(json => {
    if (json.success) {
      console.log('Payment Intent Details:', json.data);
    } else {
      console.error('Error retrieving payment intent:', json.error.message, json.error.details);
    }
  })
  .catch(error => {
    console.error('Network or parsing error:', error);
  });
```

## 4. Common Issues and Solutions

| Issue                                     | Symptom                                                              | Solution                                                                                                                                                                                                                                                                                                                                                         |
| :---------------------------------------- | :------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **401 Unauthorized**                      | API calls fail with a `401 Unauthorized` or `Invalid API Key` error. | **Missing `X-API-Key` Header:** Ensure `X-API-Key` is included in *all* API v2 requests. <br/> **Incorrect API Key:** Verify the `API_SECRET_KEY` value in your environment variables matches the key provided. <br/> **Empty/Null API Key:** Ensure your application is correctly loading the `API_SECRET_KEY` from its environment.                                    |
| **404 Not Found**                         | API calls fail with a `404 Not Found` error.                         | **Missing `/v2/` Prefix:** Ensure all API paths are updated to include `/api/v2/`. <br/> **Incorrect Endpoint Path:** Double-check the full path for the specific endpoint (e.g., `create-payment-intent` vs. `payment-intent`).                                                                                                                                    |
| **400 Bad Request (create-payment-intent)** | `create-payment-intent` fails with a `400 Bad Request` or similar.   | **Incorrect HTTP Method:** Ensure the method is `POST`, not `GET`. <br/> **Missing Request Body:** `amount` and `currency` must be sent in the JSON request body. <br/> **Incorrect Content-Type:** Set `Content-Type: application/json` header for `POST` requests with a JSON body. <br/> **Invalid Request Body:** Ensure `amount` is an integer and `currency` is a valid ISO 4217 code (e.g., `usd`). |
| **Unexpected Response Format**            | Application fails to parse API response or `data` is undefined.      | **Not Handling New Structure:** Update your response handling logic to check `json.success` and access data via `json.data` or errors via `json.error`. Avoid direct access to properties that might have been at the root level in v1.                                                                                                                             |
| **CORS Issues**                           | Browser blocks API requests with CORS errors.                        | **Server Configuration:** While not directly caused by v2, if new origins are making requests, ensure your backend's CORS policy (`payment-element/server/*`) is updated to allow requests from your client's domain.                                                                                                                                                    |
| **New features not working**              | `GET /api/v2/payment-intent/:id` returns 404 or 401                  | Ensure the specific new feature endpoints are correctly implemented, include `/api/v2/` prefix, and have the `X-API-Key` header. Double-check required parameters for each new endpoint.                                                                                                                                                                       |

## 5. Rollback Instructions

If you encounter critical issues during or after the migration that cannot be immediately resolved, you can roll back to the previous API v1 implementation.

1.  **Revert Code Changes:**
    *   Use your version control system (e.g., Git) to revert all code changes related to API v2 migration.
    *   This includes reverting changes to API endpoint paths, HTTP methods, request body structures, and response handling logic.

2.  **Revert Environment Variables:**
    *   Remove or comment out the `API_SECRET_KEY` from your environment configuration.
    *   If you had specific v1 configurations (e.g., a base API URL), restore them.

3.  **Redeploy Previous Version:**
    *   Deploy the previously stable version of your application that was compatible with API v1.

4.  **Monitor:**
    *   After rollback, closely monitor your application for any lingering issues or regressions.

**Important Note:** Rollback should be a last resort. We strongly recommend testing thoroughly in a staging environment before deploying to production to minimize the need for a rollback.

## 6. Support

If you encounter any issues not covered in this guide or require further assistance, please contact our support team through [Your Company's Support Channel/Email/Ticketing System]. Please provide detailed error messages, request/response payloads, and relevant code snippets to help us diagnose the problem quickly.