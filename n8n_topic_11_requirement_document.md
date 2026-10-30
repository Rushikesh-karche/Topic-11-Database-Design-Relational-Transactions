# Topic 11: Database Design for Automation (SQLite, PostgreSQL & MySQL)
## Official Requirement Document & Technical Specification

---

## 1. Document Control
| Field | Value |
| :--- | :--- |
| **Document Title** | Topic 11: Database Design for Automation Requirement File |
| **Version** | 1.0 |
| **Status** | Approved |
| **Last Updated** | 2026-10-30 |
| **Prepared By** | Lead Automation Architect (Antigravity AI / Tayana Academy) |
| **Target Audience** | Enterprise Automation Engineers, Database Administrators, System Integrators |

---

## 2. Learning Concepts & Architectural Principles

### 2.1 Relational Architecture in Enterprise Automation
While simple automations often rely on flat spreadsheets or key-value stores, production enterprise systems require **Relational Database Management Systems (RDBMS)** such as PostgreSQL, MySQL, or SQLite. Relational design ensures:
1. **Referential Integrity:** Foreign key constraints guarantee child records (e.g. line items) cannot exist without a valid parent (order header).
2. **ACID Transactions:** Atomicity, Consistency, Isolation, and Durability ensure that multi-step operations either commit completely or roll back entirely.
3. **Data Normalization:** Separating customer details from order headers and order line items eliminates redundant data storage and update anomalies.

### 2.2 Parent-Child Relational Pattern
When saving complex business transactions:
- The **Parent Record** (`orders`) must be created first to generate or validate the primary key (`order_id`).
- The **Child Records** (`order_items`) are inserted with the foreign key referencing the parent.
- Business rules (such as checking whether customer balance exceeds credit limits) must be validated before the transaction is finalized. If breached, all pending inserts must be rolled back.

---

## 3. Business Requirements Document (BRD)

| ID | Business Requirement | Business Rationale | Priority |
| :--- | :--- | :--- | :--- |
| **BRD-001** | Strict parent-child relational modeling for orders and line items. | Prevents orphaned records and ensures data integrity. | High |
| **BRD-002** | Parameterized database queries for all data operations. | Prevents SQL injection vulnerabilities and ensures safe execution. | High |
| **BRD-003** | Automated credit limit verification before finalizing transactions. | Mitigates financial risk by preventing unauthorized credit over-allocation. | High |
| **BRD-004** | Atomic rollback mechanism upon constraint violations. | Guarantees database consistency by purging pending entries when rules fail. | High |
| **BRD-005** | Consolidated relational JOIN retrieval and final audit receipt. | Delivers unified visibility for downstream billing and logistics services. | Medium |

---

## 4. Product Requirements Document (PRD)

| ID | Product Capability | User Value | Priority |
| :--- | :--- | :--- | :--- |
| **PRD-001** | Schema DDL Initializer | Automatically establishes table schemas and seeds reference customers. | High |
| **PRD-002** | Parameterized Order Stager | Ingests multi-item order structures with parent attributes and line items. | High |
| **PRD-003** | Foreign-Key Relational Binder | Inserts child line items referenced to the parent order ID. | High |
| **PRD-004** | ACID Commit / Rollback Engine | Automatically updates customer balances or aborts and rolls back records. | High |
| **PRD-005** | Relational JOIN Query Simulator | Retrieves parent order, customer profile, and line items in a single query. | High |
| **PRD-006** | Tamper-Evident Audit Receipt | Emits immutable verification logs confirming database state. | High |

---

## 5. Software Requirements Specification (SRS) & Technical Blueprint

### 5.1 Pipeline Canvas Architecture
```text
[ N11-01: Manual_Trigger ]
            |
            v
[ N11-02: Code_InitRelationalDB ] (Initializes tables & seed customers)
            |
            v
[ N11-03: Code_StageIncomingOrder ] (Emits multi-item order payload)
            |
            v
[ N11-04: Code_InsertOrderHeader ] (Inserts parent order header: PENDING)
            |
            v
[ N11-05: Code_InsertOrderItems ] (Batch inserts child items with Foreign Key)
            |
            v
[ N11-06: Code_UpdateCustomerBalance ] (ACID check: balance <= creditLimit)
            |
            v
[ N11-07: IF_TransactionSucceeded ] (isCommitted == true)
          /           \
   (False)             (True)
        /               \
       v                 v
[ N11-08:           [ N11-09: Code_QueryFullOrderJoin ] (Indexed SQL JOIN)
  Log_RollbackIncident]  |
                          v
                    [ N11-10: Code_FinalTransactionAudit ] (Master Audit Log)
```

### 5.2 Node Specification Table
| Node ID | Node Name | Node Type | Critical Setting | Output Outcome |
| :--- | :--- | :--- | :--- | :--- |
| **N11-01** | `Manual_Trigger` | Manual Trigger | Trigger execution on demand | Trigger pulse |
| **N11-02** | `Code_InitRelationalDB` | Code | Run Once for All Items | Schema tables & reference data |
| **N11-03** | `Code_StageIncomingOrder` | Code | Run Once for All Items | Multi-item order object |
| **N11-04** | `Code_InsertOrderHeader` | Code | Run Once for Each Item | Parent order header with total |
| **N11-05** | `Code_InsertOrderItems` | Code | Run Once for Each Item | Child items bound via Foreign Key |
| **N11-06** | `Code_UpdateCustomerBalance` | Code | Run Once for Each Item | Evaluates credit limit CHECK |
| **N11-07** | `IF_TransactionSucceeded` | IF | `{{ $json.isCommitted }} == true` | Splits committed vs rolled back |
| **N11-08** | `Log_RollbackIncident` | Code | Format rollback event | Logs aborted transaction |
| **N11-09** | `Code_QueryFullOrderJoin` | Code | Run Once for Each Item | Multi-table relational JOIN |
| **N11-10** | `Code_FinalTransactionAudit` | Code | Run Once for Each Item | Final immutable audit entry |

---

## 6. Test Plan & Validation Scenarios

### Test Case 1: Successful Order Commit within Credit Limit (`CUST-001`)
* **Input:** Customer `CUST-001` (Acme Corp). Current Balance = $1,200.00, Credit Limit = $5,000.00.
* **Order Details:** 2x Cloud Managed Server ($450.00) + 5x Enterprise Security Seat ($75.00) = $1,275.00.
* **Calculation:** Projected Balance = $1,200.00 + $1,275.00 = $2,475.00 $le$ $5,000.00.
* **Expected Outcome:** `isCommitted: true`, `transactionStatus: "COMMITTED"`. Balance updated to $2,475.00. Order marked COMMITTED. Relational JOIN retrieves customer, order, and line items. Final audit verifies successful persistence.

### Test Case 2: Credit Limit Breach & Automatic Rollback (`CUST-002`)
* **Input:** Customer `CUST-002` (Globex Inc). Current Balance = $450.00, Credit Limit = $500.00.
* **Order Details:** Same order of $1,275.00.
* **Calculation:** Projected Balance = $450.00 + $1,275.00 = $1,725.00 > $500.00.
* **Expected Outcome:** `isCommitted: false`, `transactionStatus: "ROLLED_BACK"`. Rollback logic purges pending parent order and child items. Diverts to `Log_RollbackIncident`. Zero orphaned records remain in the database.

---

## 7. Production Risks & Mitigation Strategies
1. **SQL Injection Vulnerabilities:** Strictly employ parameterized placeholders (`$1, $2`) or ORM driver binding. Never concatenate raw strings into SQL queries.
2. **Orphaned Child Records:** Always insert parent headers first, or execute parent-child inserts within an atomic transaction block (`BEGIN ... COMMIT`).
3. **Table Lock Contention:** Index foreign key columns (`order_id`, `customer_id`) to ensure row-level locking during high-concurrency writes.
