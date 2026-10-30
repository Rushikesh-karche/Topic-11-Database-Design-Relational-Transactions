# Implementation Plan: N8N Hands-On Intermediate Assessment
## Topic 11: Database Design for Automation (SQLite, PostgreSQL & MySQL)

---

## 1. Objective
Build an enterprise relational transaction management workflow in n8n. The workflow initializes relational database schemas with foreign key constraints, safely inserts parent-child order records using parameterized queries, validates customer credit limits, handles constraint violations gracefully with automatic rollback, and retrieves the consolidated order via an indexed SQL JOIN query.

---

## 2. Canvas Architecture Overview
```text
[ N11-01: Manual_Trigger ]
            |
            v
[ N11-02: Code_InitRelationalDB ]
            |
            v
[ N11-03: Code_StageIncomingOrder ]
            |
            v
[ N11-04: Code_InsertOrderHeader ]
            |
            v
[ N11-05: Code_InsertOrderItems ]
            |
            v
[ N11-06: Code_UpdateCustomerBalance ]
            |
            v
[ N11-07: IF_TransactionSucceeded ]
          /           \
   (False)             (True)
        /               \
       v                 v
[ N11-08:           [ N11-09: Code_QueryFullOrderJoin ]
  Log_RollbackIncident]  |
                          v
                    [ N11-10: Code_FinalTransactionAudit ]
```

---

## 3. Prerequisites & Environment Setup
* [x] n8n instance version 1.0.0 or higher.
* [x] Pure JavaScript execution engine (no disallowed external packages).
* [x] Embedded pinData for instant visual testing and validation.

---

## 4. Step-by-Step Practical Build Guide

### Step 1: Initialize Database Schema (`Code_InitRelationalDB`)
* Sets up tables (`customers`, `orders`, `order_items`).
* Establishes foreign keys and credit limit constraints.
* Seeds initial test customers:
  - `CUST-001`: Acme Corp, Credit Limit $5,000, Balance $1,200.
  - `CUST-002`: Globex Inc, Credit Limit $500, Balance $450.

### Step 2: Stage Incoming Order Payload (`Code_StageIncomingOrder`)
* Emits a multi-item order for `CUST-001` with order ID `ORD-DB-5001`.
* Contains 2 line items: Cloud Managed Server ($450x2) and Security Seat ($75x5).

### Step 3: Insert Order Header (`Code_InsertOrderHeader`)
* Calculates total amount: $1,275.00.
* Inserts parent order record with `status: "PENDING"`.

### Step 4: Batch Insert Order Items (`Code_InsertOrderItems`)
* Generates primary keys (`ITEM-ORD-DB-5001-1`, `ITEM-ORD-DB-5001-2`).
* Associates foreign key `orderId: "ORD-DB-5001"` to each child line item.

### Step 5: Enforce Credit Limit & ACID Commit (`Code_UpdateCustomerBalance`)
* Verifies condition: `projectedBalance <= creditLimit`.
* **If breach:** Purges pending order header and child items; returns `transactionStatus: "ROLLED_BACK"`.
* **If valid:** Commits balance update; returns `transactionStatus: "COMMITTED"`.

### Step 6: Route Transaction Outcome (`IF_TransactionSucceeded`)
* Evaluates `{{ $json.isCommitted }} == true`.
* **False Path:** Quarantines incident in `Log_RollbackIncident`.
* **True Path:** Proceeds to SQL query join.

### Step 7: Execute Indexed SQL JOIN (`Code_QueryFullOrderJoin`)
* Simulates multi-table relational join connecting `orders`, `customers`, and `order_items`.
* Returns enriched order document with customer metadata and full line item breakdown.

### Step 8: Master Transaction Audit (`Code_FinalTransactionAudit`)
* Emits immutable transaction receipt confirming database consistency.
