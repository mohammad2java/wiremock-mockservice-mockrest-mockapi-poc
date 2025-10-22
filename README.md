# wiremock-mockservice-mockrest-mockapi-poc


## In **WireMock**, the key components are the building blocks that define how it mocks HTTP services. Here’s a simple breakdown 👇

---

### ⚙️ **1. Stub Mapping**

* The core part of WireMock — defines how a request should be matched and what response to return.
* Stored in JSON files (inside `mappings/` folder) or configured via API.

```json
{
  "request": { ... },
  "response": { ... }
}
```

---

### 🧩 **2. Request**

* Describes **what kind of HTTP request** WireMock should listen for.
* Key fields:

  * `method` → GET, POST, PUT, etc.
  * `url` / `urlPath` → endpoint path.
  * `headers` → expected request headers.
  * `queryParameters` → query string matchers.
  * `bodyPatterns` → body match conditions.

Example:

```json
"request": {
  "method": "POST",
  "url": "/api/v1/user",
  "bodyPatterns": [
    { "equalToJson": { "name": "John" } }
  ]
}
```

---

### 💬 **3. Response**

* Defines what WireMock should return when a request matches.
* Key fields:

  * `status` → HTTP status code.
  * `body` or `bodyFileName` → response content.
  * `headers` → response headers.
  * `delayDistribution` → simulate latency.

Example:

```json
"response": {
  "status": 200,
  "body": "{ \"message\": \"OK\" }",
  "headers": { "Content-Type": "application/json" }
}
```

---

### 🧠 **4. Matchers**

* Define **how** WireMock matches incoming requests.
* Types: `equalTo`, `contains`, `matches`, `equalToJson`, `matchesJsonPath`, etc.
* Used inside request fields like headers, query parameters, or body patterns.

---

### 📁 **5. Files & Folders**

Default structure:

```
/wiremock
 ├── mappings/        # Stub JSON files
 └── __files/         # Response body files
```

---

### 🧰 **6. Administration API**

* Used to add, update, or delete stubs dynamically.
* Example:

  ```
  POST /__admin/mappings
  GET /__admin/mappings
  DELETE /__admin/mappings
  ```

---

Would you like me to show a **diagram** of how these components interact (Request → WireMock → Response)?

