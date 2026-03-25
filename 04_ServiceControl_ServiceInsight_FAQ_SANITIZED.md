[[_TOC_]]
# ServiceControl & ServiceInsight FAQ

This document answers common questions about how Particular ServiceControl and ServiceInsight work.

---

## 1. How do our NServiceBus messages end up inside a collection of messages that can be accessed via HTTP or via ServiceInsight?

When you configure your endpoints for **audit** and **error forwarding**, NServiceBus sends:
1. A copy of each successfully processed message to the **audit queue**.
2. A copy of each failed message to the **error queue**.

A **ServiceControl Audit instance** subscribes to those queues as another NServiceBus endpoint. It:
- Reads each audit/error message,
- Indexes metadata (headers, message type, timestamp, etc.),
- Persists the full message body into its embedded RavenDB,
- Exposes all of this via REST endpoints (e.g., `GET /api/messages`) that can be called by any `HttpClient` or by the ServiceInsight tool.

---

## 2. When I'm inside the ServiceInsight tool, all I have to do is connect to an API endpoint, then once connected it shows me all the service "endpoints" for that environment. How does the desktop app know how to find each of these endpoints?

ServiceInsight connects to the **ServiceControl HTTP API** and calls the **Endpoints discovery** endpoint, typically:

```
GET /api/endpoints
```

ServiceControl maintains an internal registry of every endpoint it sees in incoming audit or error messages. Each time it ingests a message, it notes which endpoint processed or sent that message. ServiceInsight simply fetches that list and displays it in the "Endpoint Explorer" pane.

---

## 3. Is every service endpoint exposed or can they be hidden?

- **Visible**: Endpoints that send or process **audited** messages will appear in ServiceInsight.
- **Hidden**:  
  - Send-only endpoints (they never process incoming messages) won't show up unless you explicitly configure auditing of outgoing sends.  
  - You can also exclude endpoints via ServiceControl configuration (e.g., using the **remotes** settings to include/exclude certain endpoints).

---

## 4. How does the ServiceInsight tool display messages?

ServiceInsight uses several ServiceControl REST APIs to retrieve message data:

1. **Message Flow API** (e.g., `/api/messageflows`)  
2. **Sequence API** (e.g., `/api/messages/{id}/sequence`)

It then renders:
- **Flow diagrams** showing send/publish arrows between endpoints,  
- **Sequence diagrams** showing the chronological order of message handling steps.  

By default it pages through the most recent messages (usually up to 200 for performance), but you can page or filter as needed.

---

## 5. How are these messages stored and where are they stored?

ServiceControl uses an **embedded RavenDB** database:
- Each message (body + metadata) is stored as a JSON document.
- Documents are indexed by message ID, correlation ID, headers, endpoint, etc.
- Failed imports (e.g., invalid JSON) are placed in a separate "failed imports" store for troubleshooting.

---

## 6. How do you control which messages are and are not visible or stored?

Control is achieved through endpoint configuration and ServiceControl settings:
1. **Auditing settings** in your NServiceBus endpoint (`AuditConfig` or `ServiceControl/AuditQueue`): decide which messages are forwarded to audit.  
2. **Error queue** configuration: controls which failed messages are sent to the error queue.  
3. **Time-To-Be-Received (TTBR)** headers on messages: cause short-lived messages to expire before reaching queues.  
4. **ServiceControl retention policies** (in the ServiceControl host config): define how long messages remain in RavenDB before being purged.

---

## 7. Is it completely safe and reliable to implement a ServiceControl API inside my end-to-end test solution for the sole purpose of grabbing message body and message type metadata for writing assertions against?

Using the ServiceControl HTTP API in your end-to-end test harness to pull down audited messages and assert on their contents is a perfectly valid approach—but it isn't 100% "bullet-proof" out of the box. Here are the key considerations:

1. **Eventual Consistency & Latency**  
   - **Indexing lag:** ServiceControl doesn't make messages immediately searchable the moment they're processed—you'll usually see them show up within a few hundred milliseconds to a couple of seconds, depending on load and cluster size.  
   - **Retry/timeout logic:** Your test code should include a short retry loop (e.g. poll every 100–500 ms for up to 30 s) rather than assuming the message is instantly available.

2. **Infrastructure Dependencies**  
   - **Network calls:** Each API hit adds a dependency on HTTP availability, certificates/firewall rules, and the health of the ServiceControl instance. Tests may fail spuriously if the audit service is under heavy load or if the VM hosting it is being rebooted.  
   - **Authentication:** Make sure your tests run under a security principal (e.g. managed identity or service account) that has read permission on the ServiceControl API, and that credential management is robust.

3. **Test Isolation & Side-Effects**  
   - **Shared audit store:** You're querying a shared UAT audit instance, so tests running in parallel (or other deployments) may surface messages from unrelated runs. Use a unique correlation ID per test and scope your queries tightly.  
   - **Clean state:** Unlike an in-memory or mock bus, you cannot "reset" the audit store between tests. Plan for idempotent queries (e.g. always look for the specific correlation ID and message type).

4. **Performance & Throughput**  
   - **High-volume scenarios:** If your tests emit hundreds of messages per run, paging and filtering through the ServiceControl API can become slow. Consider indexing only the message types you need, or use a smaller test-only audit instance.  
   - **Parallelization limits:** Hitting the same HTTP endpoint from multiple test runners can lead to throttling or connection exhaustion. You may need to stagger or serialize your validation calls.

5. **Resilience & Error Handling**  
   - **Handle 5xx errors:** Transient server errors or timeouts should be retried with back-off, not treated as hard failures.  
   - **Clear diagnostics:** Wrap failures in clear exceptions (e.g. `MessageNotFoundException`) so when a test truly fails you know it's because the message didn't arrive or match, not because of a transient network hiccup.

**Bottom Line** 

**Yes**, integrating ServiceControl's REST API in your E2E tests to pull message bodies and metadata is a powerful way to validate real, end-to-end behavior.  

**But** treat it like any other external service dependency: account for eventual consistency, network reliability, shared state, and performance constraints. With proper retry logic, scoped queries (unique correlation IDs), and robust error handling, you'll have a reliable test harness—and one that mirrors exactly what your QA team does when they "Search Messages" in ServiceInsight.

---
*Generated for internal NServiceBus team documentation.*  
