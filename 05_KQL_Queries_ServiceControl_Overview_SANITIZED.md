[[_TOC_]]
# Saved KQL Queries for Tracking Operation IDs

This document provides two methods to locate the `operation_Id` associated with a given test-case tracking GUID in Azure Application Insights. Replace `INSERT_TEST_CASE_TRACKING_ID_HERE` with your actual GUID when running the query.

---

## 1. Literal GUID Search

**Description:**  
This treats the GUID as literal text and searches across all columns in the specified tables.

```kql
search in (traces, dependencies, requests, customEvents, customMetrics, exceptions) 
    "INSERT_TEST_CASE_TRACKING_ID_HERE"
| where timestamp > ago(2h)
| distinct operation_Id
```

---

## 2. Tagged Source-Table Search

**Description:**  
1. Tag each row with its source table.  
2. Filter by timestamp and by any column containing your GUID.  
3. Project and distinct your `operation_Id` (plus table if you need it).

```kql
let guid = "INSERT_TEST_CASE_TRACKING_ID_HERE";
union
  (traces       | extend SourceTable = "traces"),
  (dependencies | extend SourceTable = "dependencies"),
  (requests     | extend SourceTable = "requests"),
  (customEvents | extend SourceTable = "customEvents"),
  (customMetrics| extend SourceTable = "customMetrics"),
  (exceptions   | extend SourceTable = "exceptions")
| where timestamp > ago(2h)
  and (
       tostring(customDimensions) has guid
    or tostring(message)           has guid
    or tostring(name)              has guid
  )
| project SourceTable, operation_Id, timestamp
| distinct operation_Id, SourceTable
```

---

# ServiceControl & ServiceInsight Overview

A concise reference guide to how ServiceControl and ServiceInsight operate within our NServiceBus ecosystem.

---

## 1) Message Ingestion Flow

**Description**: Processed and failed messages are audited and forwarded into ServiceControl.

```
[Sender Endpoint]
      |
   NServiceBus
      |
 [Audit/Error Queue]
      |
[ServiceControl Audit] → RavenDB
      |
[HTTP API (/messages)]
```

---

## 2) Endpoint Discovery in ServiceInsight

**Discovery API**: `GET /api/endpoints`  
**Mechanism**: ServiceControl records each endpoint it sees from audit/error messages and exposes them via its API.

```
ServiceInsight App
      |
    HTTP GET /api/endpoints
      ↓
  List of Registered Endpoints
```

---

## 3) Endpoint Visibility

- **Visible**: Endpoints that send/receive audited messages.  
- **Hidden**: Send-only endpoints (unless explicitly audited), or endpoints filtered out via ServiceControl config.

---

## 4) Message Display

- **Flow Diagrams**: Visualizes send/publish arrows between endpoint lifecycles.  
- **Sequence Diagrams**: Shows the chronological order of message handling.  
- **APIs Utilized**: 
  - `/api/messageflows`
  - `/api/messages/{id}/sequence`

---

## 5) Storage Backend

- **Database**: Embedded RavenDB.  
- **Storage Model**: Each message and its metadata are stored as JSON documents, indexed by message ID, correlation ID, headers, etc.

---

## 6) Control & Retention

- **Auditing Settings**: Configured via NServiceBus `AuditConfig` or `ServiceControl/AuditQueue`.  
- **Error Queue**: Failed messages are forwarded here.  
- **ServiceControl Retention Policies**: Define how long messages remain before purging.  
- **TTBR Headers**: Message expiration before reaching the audit/error queues.
