# Distributed API Rate Limiter Gateway

A robust, distributed API Gateway that implements rate limiting using the **Token Bucket Algorithm**. This project intercepts incoming requests and manages traffic to prevent backend service overload, utilizing Redis to maintain state across multiple gateway instances.

## 🚀 Key Features

* **Token Bucket Implementation:** Efficiently handles both steady traffic and sudden bursts of requests.
* **Distributed State Management:** Uses Redis to store and synchronize rate limit counters (tokens and refill timestamps) across a clustered environment.
* **Thread-Safe Operations:** Leverages Redis atomic operations (like `DECR`) to prevent race conditions during concurrent high-volume traffic.
* **Smart IP Detection:** Accurately identifies clients by evaluating `X-Forwarded-For` headers (for traffic through proxies/load balancers) with a fallback to direct remote addresses.
* **Custom Gateway Filters:** Implements Spring Cloud Gateway filters to apply rate-limiting logic seamlessly before routing requests to backend microservices.

## 🛠️ Tech Stack

* **Language:** Java 21
* **Framework:** Spring Boot 3.5.12, Spring Cloud Gateway (v2024.0.0)
* **Data Store:** Redis (via Jedis client)
* **Build Tool:** Maven 3.8+
* **Utilities:** Lombok

## ⚙️ Configuration

The rate limiting behavior and backend routing can be configured in the `application.properties` file:

| Property | Description | Default Value |
| :--- | :--- | :--- |
| `rate-limiter.capacity` | Maximum tokens the bucket can hold (burst capacity) | `10` |
| `rate-limiter.refill-rate` | Number of tokens added back to the bucket every second | `5` |
| `rate-limiter.api-server-url` | The destination backend service URL where allowed requests are routed | `http://localhost:8081` |
| `spring.data.redis.host` | Redis server hostname | `localhost` |
| `spring.data.redis.port` | Redis server port | `6379` |

## 🌐 API Reference

### Rate-Limited Routes
All requests directed to `/api/**` are intercepted by the gateway.
* **Action:** The `/api` prefix is stripped, and the request is forwarded to the configured backend server.
* **Response Headers:** Every response includes rate-limit tracking headers:
    * `X-RateLimit-Limit`: The total configured capacity for the client.
    * `X-RateLimit-Remaining`: The number of tokens currently remaining for the client.
* **Error Handling:** If the limit is exceeded, the gateway short-circuits the request and returns a `429 Too Many Requests` status with a JSON error payload.

### Monitoring Endpoints
* `GET /gateway/health`: Returns the health status of the gateway service.
* `GET /gateway/rate-limit/status`: Returns the current token count, total capacity, and identified Client ID for the calling user.

## 🚦 Getting Started

### Prerequisites
* Java 21 installed
* Maven 3.8+ installed
* A running instance of Redis (defaulting to `localhost:6379`)

### Installation & Execution

1.  **Clone the repository:**
    ```bash
    git clone <your-repository-url>
    cd RateLimitingApplication
    ```

2.  **Start your Redis server:**
    ```bash
    redis-server
    ```

3.  **Build and run the application:**
    ```bash
    mvn clean install
    mvn spring-boot:run
    ```

## 🧪 Testing

The project includes a standalone Java test script (`RateLimiterTest.java`) utilizing the modern Java `HttpClient` to simulate rapid, concurrent requests.

To verify the logic:
1. Ensure the gateway is running.
2. Execute the `main` method in `RateLimiterTest.java`.
3. **Expected Output:** With default settings (capacity of 10), you should see the first 10 requests successfully allowed (200 OK or 502 if the backend is off) and the subsequent requests immediately blocked with a `429` status.
