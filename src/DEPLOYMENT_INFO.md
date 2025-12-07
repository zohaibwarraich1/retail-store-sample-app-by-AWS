# Deployment Information

## Service Deployment Order
To ensure proper connectivity and dependency resolution, deploy the services in the following order:

1.  **Infrastructure / Databases** (If not using in-memory mode)
    *   DynamoDB (for Cart)
    *   MySQL (for Catalog)
    *   PostgreSQL (for Orders)
    *   RabbitMQ (for Orders)
    *   Redis (for Checkout)

2.  **Catalog Service**
    *   **Dependencies**: Database (MySQL)
    *   **Port**: 8080
    *   **Health Check**: `/health`

3.  **Cart Service**
    *   **Dependencies**: Database (DynamoDB)
    *   **Port**: 8080
    *   **Health Check**: `/actuator/health`

4.  **Orders Service**
    *   **Dependencies**: Database (PostgreSQL), Messaging (RabbitMQ)
    *   **Port**: 8080
    *   **Health Check**: `/actuator/health`

5.  **Checkout Service**
    *   **Dependencies**: Orders Service, Redis
    *   **Port**: 8080
    *   **Health Check**: `/health` (or `/api` for Swagger)

6.  **UI Service**
    *   **Dependencies**: Catalog, Cart, Orders, Checkout
    *   **Port**: 8080
    *   **Health Check**: `/actuator/health`

---

## Service Connections & Ports

All services listen on **Port 8080** by default. In a Kubernetes environment, each service should be exposed via a Service resource on port 80 (mapping to targetPort 8080).

| Service | Default Port | Environment Variable to Override Port | Connected Services (Outbound) |
| :--- | :--- | :--- | :--- |
| **Catalog** | 8080 | `PORT` | MySQL |
| **Cart** | 8080 | `SERVER_PORT` | DynamoDB |
| **Orders** | 8080 | `SERVER_PORT` | PostgreSQL, RabbitMQ |
| **Checkout** | 8080 | `PORT` | Orders Service (`RETAIL_CHECKOUT_ENDPOINTS_ORDERS`), Redis |
| **UI** | 8080 | `SERVER_PORT` | Catalog (`RETAIL_UI_ENDPOINTS_CATALOG`), Cart (`RETAIL_UI_ENDPOINTS_CARTS`), Checkout (`RETAIL_UI_ENDPOINTS_CHECKOUT`), Orders (`RETAIL_UI_ENDPOINTS_ORDERS`) |

## Routing & Request Flow

The **UI Service** acts as the entry point for the client (browser). It handles requests in two ways:

### 1. Server-Side Rendering (BFF Pattern)
The UI Service renders HTML pages using Thymeleaf. It makes backend calls internally to populate these pages.
*   **Route**: `/cart` -> **UI Service** (Java) -> calls **Cart Service** API -> Renders HTML.
*   **Route**: `/home` -> **UI Service** (Java) -> calls **Catalog Service** API -> Renders HTML.

### 2. API Proxying
The UI Service also exposes a proxy endpoint to allow the browser to make direct API calls to backends without CORS issues.
*   **Route**: `/proxy/catalog/*` -> forwards to **Catalog Service**
*   **Route**: `/proxy/carts/*` -> forwards to **Cart Service**
*   **Route**: `/proxy/checkout/*` -> forwards to **Checkout Service**
*   **Route**: `/proxy/orders/*` -> forwards to **Orders Service**

**Example**:
A request to `http://ui-service/proxy/catalog/products` is forwarded by the UI Service to `http://catalog-service:8080/products`.
