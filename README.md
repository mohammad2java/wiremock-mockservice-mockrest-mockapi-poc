# wiremock-mockservice-mockrest-mockapi-poc


## => In **WireMock**, the key components are the building blocks that define how it mocks HTTP services. Here’s a simple breakdown 👇

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
## => WireMock request configuration** that matches both a URL and a specific **request body** 👇

```json
{
  "request": {
    "method": "POST",
    "url": "/api/v1/user",
    "bodyPatterns": [
      {
        "equalToJson": {
          "name": "John",
          "email": "john@example.com"
        }
      }
    ]
  },
  "response": {
    "status": 200,
    "body": "{ \"message\": \"User created successfully\" }",
    "headers": {
      "Content-Type": "application/json"
    }
  }
}
```

### 🔍 Explanation

* **`method`** → HTTP method (e.g., POST, GET, PUT).
* **`url`** → The endpoint path you want to mock.
* **`bodyPatterns`** → Used to match the incoming request body.

  * `equalToJson` → Ensures the JSON body matches exactly.
  * Other options:

    * `contains` → match if body contains text
    * `matchesJsonPath` → match specific JSON paths
* **`response`** → What WireMock returns when the request matches.

## (a *partial match*): example

```json
{
  "request": {
    "method": "POST",
    "url": "/api/v1/user",
    "bodyPatterns": [
      {
        "matchesJsonPath": "$[?(@.email == 'john@example.com')]"
      }
    ]
  },
  "response": {
    "status": 200,
    "body": "{ \"message\": \"Email matched successfully\" }",
    "headers": {
      "Content-Type": "application/json"
    }
  }
}
```



## =>`bodyPatterns` array can contain **one or more patterns** — each pattern describes how the request body should be matched.

Here are the **supported pattern types** you can use inside `bodyPatterns`:

| Pattern type      | Description                                 | Example                                   |
| ----------------- | ------------------------------------------- | ----------------------------------------- |
| `equalTo`         | Matches the exact plain text body           | `"equalTo": "simple text"`                |
| `contains`        | Matches if the body contains specific text  | `"contains": "keyword"`                   |
| `matches`         | Matches if the body matches a regex pattern | `"matches": ".*success.*"`                |
| `doesNotMatch`    | Matches if body does *not* match regex      | `"doesNotMatch": ".*error.*"`             |
| `equalToJson`     | Matches exact JSON (ignores spacing)        | `"equalToJson": { "id": 1 }`              |
| `matchesJsonPath` | Matches based on JSONPath expression        | `"matchesJsonPath": "$.user.name"`        |
| `equalToXml`      | Matches exact XML                           | `"equalToXml": "<user><id>1</id></user>"` |
| `matchesXPath`    | Matches XML using XPath                     | `"matchesXPath": "/user/id[text()='1']"`  |

✅ You can use **multiple patterns** in the same `bodyPatterns` array — all must match for the stub to trigger.

### Example with multiple patterns:

```json
"bodyPatterns": [
  { "contains": "orderId" },
  { "matchesJsonPath": "$.user.name" }
]
```

This means:

* The request body must contain the text `orderId`, **and**
* The JSON body must have a field `user.name`.


## => You can keep response json in seperate file instead of mappings file..below is complete informations.

Perfect 👍 — let’s go a bit deeper into what happens inside the

```
/wiremock
 ├── mappings/
 └── __files/
```

folders and how WireMock uses them.

---

## 🗂️ **1. `/mappings/` Folder**

This folder contains all your **stub mapping files** — the heart of WireMock.
Each file describes **what kind of request** WireMock should expect and **what response** it should return.

### 🧩 Example: `create-user.json`

```json
{
  "request": {
    "method": "POST",
    "url": "/api/v1/user",
    "bodyPatterns": [
      { "matchesJsonPath": "$[?(@.email == 'john@example.com')]" }
    ]
  },
  "response": {
    "status": 200,
    "bodyFileName": "user-created.json",
    "headers": {
      "Content-Type": "application/json"
    }
  }
}
```

### 🔍 How it works:

* This file lives inside `/mappings/`.
* When WireMock starts, it automatically **loads all JSON files** from `/mappings/` and registers them as stubs.
* Each stub tells WireMock:

  * Which **URL** and **HTTP method** to match.
  * Optionally, which **request body**, **headers**, or **query params** to match.
  * What **response** to send (directly in `body`, or using a file via `bodyFileName`).

---

## 📁 **2. `/__files/` Folder**

This folder stores the **actual response body files** — typically JSON, XML, or plain text.

### 🧩 Example: `user-created.json`

Located at:

```
/wiremock/__files/user-created.json
```

Contents:

```json
{
  "message": "User created successfully",
  "userId": 123
}
```

### 🔍 How it works:

* When your mapping uses `"bodyFileName": "user-created.json"`,
  WireMock looks for that file inside `/__files/` and returns its content as the HTTP response body.
* This keeps your stubs **clean and readable**, especially when responses are large.

---

## 🧠 **Why use this structure**

| Benefit                          | Explanation                                                   |
| -------------------------------- | ------------------------------------------------------------- |
| **Separation of logic and data** | Keeps mappings (rules) separate from large response payloads. |
| **Reusability**                  | You can reuse the same response file for multiple stubs.      |
| **Organization**                 | Easy to manage multiple mock endpoints — one mapping per API. |
| **Maintainability**              | Easier to update responses without touching logic.            |

---

### 🏗️ Typical Folder Layout Example

```
wiremock/
├── mappings/
│   ├── get-user.json
│   ├── create-user.json
│   └── delete-user.json
└── __files/
    ├── user-details.json
    ├── user-created.json
    └── user-deleted.json
```



## => Multiple ways to integrate WireMock**, depending on your use case (local testing, CI/CD, or embedded in code).

---

## 🧱 **1. Standalone JAR (Manual Run)**

✅ **Most common for local testing**

* Download the WireMock standalone JAR from [wiremock.org](https://wiremock.org).
* Run it with:

  ```bash
  java -jar wiremock-standalone-x.x.x.jar --port 8080
  ```
* Default folder structure:

  ```
  /wiremock
   ├── mappings/
   └── __files/
  ```
* Place your stub JSONs and response files there.
* Access UI at `http://localhost:8080/__admin/`.

👉 Good for **manual testing**, **mocking external APIs**, or **integration testing**.

---

## 🐳 **2. Using Docker Container**

✅ **Most popular for CI/CD pipelines or team environments**

* Pull the official image:

  ```bash
  docker pull wiremock/wiremock
  ```
* Run with volume mount:

  ```bash
  docker run -it --rm \
    -p 8080:8080 \
    -v $(pwd)/wiremock:/home/wiremock \
    wiremock/wiremock:latest
  ```

💡 Folder `/home/wiremock` inside the container maps to your local stub files:

```
wiremock/
 ├── mappings/
 └── __files/
```

👉 Ideal for **team-shared mocks** or **containerized test setups**.

---

## 🧩 **3. Embedded WireMock (Java Library)**

✅ **Used for integration/unit testing inside your Java code**

Add dependency (Maven):

```xml
<dependency>
  <groupId>com.github.tomakehurst</groupId>
  <artifactId>wiremock-jre8</artifactId>
  <version>2.35.0</version>
  <scope>test</scope>
</dependency>
```

Use in tests:

```java
import static com.github.tomakehurst.wiremock.client.WireMock.*;
import com.github.tomakehurst.wiremock.WireMockServer;

public class MyTest {
  WireMockServer wireMockServer = new WireMockServer(8080);

  @Before
  public void setup() {
    wireMockServer.start();
    stubFor(get(urlEqualTo("/api/test"))
      .willReturn(aResponse().withBody("Hello WireMock")));
  }

  @After
  public void teardown() {
    wireMockServer.stop();
  }
}
```

👉 Perfect for **JUnit integration** and **mocking external services in test cases**.

---

## ☁️ **4. WireMock Cloud (Hosted Service)**

✅ **Managed solution from WireMock.io**

* No need to run locally — your stubs live in the cloud.
* Provides a **web UI**, collaboration tools, and versioning.
* Useful for **shared mocks** across teams or **external API simulations**.

Website: [https://www.wiremock.io](https://www.wiremock.io)

---

## 🔗 **5. WireMock via Testcontainers**

✅ **Dynamic containerized testing in Java**

Use with [Testcontainers](https://www.testcontainers.org/):

```java
import org.testcontainers.containers.WireMockContainer;

public class WireMockContainerTest {
  @Container
  private static final WireMockContainer wiremock =
      new WireMockContainer("wiremock/wiremock:latest")
          .withMapping("test", WireMockContainerTest.class, "test-mapping.json");
}
```

👉 Ideal for **isolated integration tests** that spin up WireMock automatically during test execution.

---

## ⚙️ **Summary Table**

| Method               | Environment            | Typical Use                 | Pros                    |
| -------------------- | ---------------------- | --------------------------- | ----------------------- |
| **Standalone JAR**   | Local                  | Manual or simple automation | Easy setup, lightweight |
| **Docker Container** | Local / CI             | Shared mocks, automation    | Consistent, portable    |
| **Embedded Java**    | Unit/Integration Tests | Full test control           | No external setup       |
| **WireMock Cloud**   | Cloud                  | Collaboration, remote APIs  | Managed, shareable      |
| **Testcontainers**   | CI / Integration Tests | Ephemeral mocks             | Fully automated         |




