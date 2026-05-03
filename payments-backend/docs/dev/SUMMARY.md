# Payments Backend API - `docai_smart_dkpaon23`

Welcome to the Payments Backend API repository! This project serves as a comprehensive example and implementation of a robust, secure, and feature-rich payment processing backend. It demonstrates various server-side technologies interacting with a payment gateway (implied by Stripe integration) and provides client-side examples for integration.

This repository recently underwent a significant upgrade, introducing **API v2.0**, which enhances security, standardizes responses, and expands payment-related functionalities.

---

## 🚀 Overview

The `payments-backend` repository provides a collection of backend server implementations (in Java, Node.js, Next.js, Python, and Ruby) that expose a unified API for managing payment intents, customers, refunds, and orders. It is designed to be highly extensible and serves as a reference for building secure payment systems.

**Key Highlights of API v2.0:**

*   **Mandatory API Key Authentication**: All v2 API endpoints now require an `X-API-Key` header for enhanced security.
*   **API Versioning**: All new and updated endpoints are prefixed with `/api/v2/`.
*   **Standardized Responses**: Consistent success and error response formats across all endpoints.
*   **Extended Payment Features**: New dedicated endpoints for customer creation, retrieving payment intent status, processing refunds, and fetching order details.
*   **Breaking Changes**: Existing clients must migrate to API v2.0, updating endpoint paths, authentication headers, and request/response structures.

This upgrade significantly improves the API's maintainability, security, and developer experience.

## 🏛️ Architecture

The project adopts a polyglot architecture, showcasing how different backend technologies can implement the same API specification.

*   **Multiple Backend Implementations**:
    *   Each language (Java, Node.js, Next.js, Python, Ruby) has its own server implementation, demonstrating how to handle API routing, authentication, and business logic.
    *   These servers are responsible for interacting with the external payment gateway (e.g., Stripe) to perform payment operations.
*   **Authentication Middleware**: A common authentication layer is applied across all v2 endpoints, enforcing API key validation.
*   **Centralized Routing Concept**: Although implemented separately in each language, the API routes follow a consistent pattern (`/api/v2/...`).
*   **Client Examples**: Frontend applications (HTML, React) are provided to demonstrate how to integrate with the authenticated v2 API endpoints.

```
+----------------+       +---------------------+
|  Client (HTML) |       |  Client (React)     |
+----------------+       +---------------------+
        |                       |
        |  HTTPS Requests       |  HTTPS Requests
        v                       v
+------------------------------------------------+
|           API Gateway / Load Balancer          | (Conceptual/Implicit)
|           (Routes to specific backend)         |
+------------------------------------------------+
        |  API v2.0 (X-API-Key)
        v
+-------------------------------------------------------------------------------------------------------+
|                                  Backend Services (Polyglot)                                          |
|  +-------------------+  +-------------------+  +-------------------+  +-------------------+  +-------------------+  |
|  |   Java Server     |  |   Node.js Server  |  |   Next.js Server  |  |   Python Server   |  |   Ruby Server     |  |
|  | (API v2 Logic)    |<->| (API v2 Logic)    |<->| (API v2 Logic)    |<->| (API v2 Logic)    |<->| (API v2 Logic)    |  |
|  +-------------------+  +-------------------+  +-------------------+  +-------------------+  +-------------------+  |
|        | API Key Auth / Payment Intent / Refunds / Customers / Orders Logic (Shared Specification)                     |
+-------------------------------------------------------------------------------------------------------+
        |
        v
+----------------------+
| External Payment GW  | (e.g., Stripe)
+----------------------+
```

## 🚀 Getting Started

To run this project locally, you'll need to set up the environment variables and start at least one backend server along with a client example.

### Prerequisites

*   **Git**: For cloning the repository.
*   **Node.js & npm/yarn**: For Node.js/Next.js backends and React/HTML clients.
*   **Java Development Kit (JDK)**: For the Java backend.
*   **Python 3 & pip**: For the Python backend.
*   **Ruby & Bundler**: For the Ruby backend.
*   **Stripe Account**: Necessary to obtain API keys for payment processing (or use test keys).

### Setup Steps

1.  **Clone the Repository:**
    ```bash
    git clone https://github.com/docai_smart_dkpaon23/payments-backend.git
    cd payments-backend
    ```

2.  **Configure Environment Variables:**
    Create a `.env` file in the root directory of the repository (or in specific backend folders if preferred) with the following variables. These are crucial for API authentication and payment gateway integration.

    ```dotenv
    API_VERSION=2.0.0
    API_SECRET_KEY=sk_your_generated_api_key_for_backend_auth # This is your custom key for authenticating API v2 requests
    STRIPE_SECRET_KEY=sk_test_your_stripe_secret_key          # Get this from your Stripe dashboard
    STRIPE_PUBLIC_KEY=pk_test_your_stripe_public_key          # Get this from your Stripe dashboard
    ```
    *   **`API_SECRET_KEY`**: This is a custom secret key your clients will use to authenticate requests to your `payments-backend` API. **Generate a strong, unique key.**
    *   **`STRIPE_SECRET_KEY`**: Your secret key for the Stripe API. Found in your Stripe Dashboard.
    *   **`STRIPE_PUBLIC_KEY`**: Your publishable key for the Stripe API. Used by the client-side.

3.  **Run a Backend Server:**
    Navigate to your preferred backend implementation and follow its specific instructions.

    *   **Example (Node.js/Express):**
        ```bash
        cd backends/nodejs-express
        npm install
        npm start # or `node server.js`
        ```
    *   **Example (Java/Spring Boot):**
        ```bash
        cd backends/java-spring-boot
        # Ensure Maven/Gradle is installed
        ./mvnw spring-boot:run # or `gradle bootRun`
        ```
    *   Similar steps apply for `backends/nextjs`, `backends/python`, `backends/ruby`. Refer to their respective `README` files for precise instructions.

4.  **Run a Client Example:**
    Once a backend server is running, you can start a client to interact with it.

    *   **Example (React Client):**
        ```bash
        cd clients/react-client
        npm install
        npm start
        ```
        This will typically open the client application in your browser (`http://localhost:3000`).

    *   **Example (HTML Client):**
        ```bash
        cd clients/html-client
        # You might need a simple static file server for browsers to correctly load all assets
        # E.g., using Node.js: `npx http-server` or Python: `python -m http.server`
        ```

## 💡 Usage

Interacting with the API v2.0 requires including the `X-API-Key` header with your `API_SECRET_KEY`.

### Example API Request (using `curl`)

Let's assume your backend is running on `http://localhost:4242`.

1.  **Create a Payment Intent:**
    ```bash
    curl -X POST \
      http://localhost:4242/api/v2/create-payment-intent \
      -H 'Content-Type: application/json' \
      -H 'X-API-Key: sk_your_generated_api_key_for_backend_auth' \
      -d '{
            "amount": 2000,
            "currency": "usd"
          }'
    ```
    Expected Response:
    ```json
    {
      "success": true,
      "data": {
        "clientSecret": "pi_xxxxxxxxxxxx_secret_xxxxxxxxxxxx",
        "publishableKey": "pk_test_xxxxxxxxxxxx",
        "amount": 2000,
        "currency": "usd",
        "id": "pi_xxxxxxxxxxxx"
      }
    }
    ```

2.  **Retrieve Payment Intent Status:**
    (Replace `pi_xxxxxxxxxxxx` with an actual Payment Intent ID from a previous creation)
    ```bash
    curl -X GET \
      http://localhost:4242/api/v2/payment-intent/pi_xxxxxxxxxxxx \
      -H 'X-API-Key: sk_your_generated_api_key_for_backend_auth'
    ```
    Expected Response:
    ```json
    {
      "success": true,
      "data": {
        "id": "pi_xxxxxxxxxxxx",
        "amount": 2000,
        "currency": "usd",
        "status": "requires_payment_method",
        "customer": null,
        "description": null
      }
    }
    ```

For a comprehensive guide on all API v2.0 endpoints, request/response structures, and migration details for existing clients, please refer to the dedicated **`API_V2_MIGRATION.md`** document.

## 📁 Project Structure

The repository is organized to separate backend implementations, client examples, and documentation.

```
payments-backend/
├── .env.example                     # Example environment variables file
├── API_V2_MIGRATION.md              # Detailed guide for API v2.0 migration
├── README.md                        # This file
├── backends/                        # Contains all backend server implementations
│   ├── java-spring-boot/            # Java backend using Spring Boot
│   │   ├── src/
│   │   ├── pom.xml
│   │   └── ...
│   ├── nodejs-express/              # Node.js backend using Express
│   │   ├── server.js
│   │   ├── package.json
│   │   └── ...
│   ├── nextjs/                      # Next.js API Routes backend
│   │   ├── pages/api/
│   │   ├── package.json
│   │   └── ...
│   ├── python/                      # Python backend (e.g., Flask/Django)
│   │   ├── app.py
│   │   ├── requirements.txt
│   │   └── ...
│   └── ruby/                        # Ruby backend (e.g., Rails/Sinatra)
│       ├── app.rb
│       ├── Gemfile
│       └── ...
└── clients/                         # Contains client-side integration examples
    ├── html-client/                 # Simple HTML/JavaScript client
    │   ├── index.html
    │   ├── script.js
    │   └── style.css
    └── react-client/                # React application client
        ├── src/
        ├── public/
        ├── package.json
        └── ...
```

## 🌐 API Overview (v2.0)

API v2.0 introduces a robust and secure set of endpoints for payment processing and related operations.

*   **Base URL**: All v2 endpoints are prefixed with `/api/v2/`.
*   **Authentication**:
    *   Required Header: `X-API-Key`
    *   Value: Your `API_SECRET_KEY` configured in the environment.
*   **Standardized Responses**:
    ```json
    // Success
    {
      "success": true,
      "data": { /* resource object */ }
    }

    // Error
    {
      "success": false,
      "error": {
        "code": "error_code_string",
        "message": "Human-readable error message."
      }
    }
    ```

### Key Endpoints

| Method | Endpoint                        | Description                                                         | Request Body                 | Response Data                 |
| :----- | :------------------------------ | :------------------------------------------------------------------ | :--------------------------- | :---------------------------- |
| `GET`  | `/api/v2/config`                | Retrieves client-side configuration (e.g., Stripe public key).      | None                         | `{ publishableKey: string }`  |
| `POST` | `/api/v2/create-payment-intent` | Creates a new Payment Intent with specified amount and currency.    | `{ amount: int, currency: string }` | `{ clientSecret: string, publishableKey: string, ... }` |
| `POST` | `/api/v2/customers`             | Creates a new customer in the payment gateway.                      | `{ email: string, name: string, ... }` | `{ id: string, email: string, ... }` |
| `GET`  | `/api/v2/payment-intent/:id`    | Retrieves the status and details of a specific Payment Intent.      | None                         | `{ id: string, amount: int, status: string, ... }` |
| `POST` | `/api/v2/refunds`               | Processes a refund for a payment (requires Payment Intent ID).      | `{ payment_intent_id: string, amount: int }` | `{ id: string, status: string, ... }` |
| `GET`  | `/api/v2/orders/:id`            | Retrieves details for a specific order.                             | None                         | `{ id: string, items: [], total: int, ... }` |

**For a detailed changelog and migration guide from previous API versions, please see `API_V2_MIGRATION.md`.**

## 👨‍💻 Development Workflow

*   **Adding New Backend Implementations**:
    *   Create a new directory under `backends/`.
    *   Implement the API v2.0 specification, including authentication middleware.
    *   Ensure all defined endpoints (`create-payment-intent`, `customers`, `refunds`, `orders`, `config`) are present and function correctly.
    *   Add a `README.md` to your specific backend directory with setup and run instructions.
*   **Contributing to Existing Backends**:
    *   Follow the coding standards and practices of the specific language/framework.
    *   Implement new features or bug fixes as per the API v2.0 specification.
    *   Ensure all changes are covered by tests (if applicable) and align with the `API_V2_MIGRATION.md` documentation.
*   **Updating Documentation**:
    *   Any changes to the API, environment variables, or setup process should be reflected in this `README.md` and critically in `API_V2_MIGRATION.md`.

## 📦 Key Dependencies

This project leverages a variety of technologies across its different implementations:

*   **Backend Languages & Frameworks**:
    *   **Node.js**: Express.js, Next.js (for API routes)
    *   **Java**: Spring Boot
    *   **Python**: Flask / Django (implied)
    *   **Ruby**: Ruby on Rails / Sinatra (implied)
*   **Payment Gateway Integration**:
    *   Stripe SDKs (for various languages)
*   **Frontend**:
    *   React.js
    *   Plain HTML/JavaScript/CSS
*   **Build Tools**:
    *   npm / yarn
    *   Maven / Gradle (for Java)
    *   pip (for Python)
    *   Bundler (for Ruby)

---

We welcome contributions and feedback! Please refer to the `API_V2_MIGRATION.md` for detailed information regarding the API v2.0 upgrade.

---

## Navigation Index
# Summary

* [Home](SUMMARY.md)

## Architecture
* [V1.0-ARCHITECTURE](architecture/v1.0-architecture.md)

## Workflow
* [V1.0-WORKFLOW](workflow/v1.0-workflow.md)

## API
* [API Documentation](api.md)

## Documentation Info
* Persona: **dev**
* Generated: 2026-05-03 17:23 UTC
## Changes

* [Introduce API v2 with Authentication and Extended Payment Features](changes/a56c0b78adda367399f1e4c8e0a94e417c6232dc-feature.md)

