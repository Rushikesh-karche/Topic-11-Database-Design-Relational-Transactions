# Topic 11: Database Design for Automation (SQLite, PostgreSQL & MySQL)
**n8n Hands-On Assessment - Level 2**

[![n8n Version](https://img.shields.io/badge/n8n-v1.0%2B-EA4B71?logo=n8n)](https://n8n.io/)
[![Status](https://img.shields.io/badge/Status-Completed-success)](https://github.com/Rushikesh-karche/Topic-11-Database-Design-Relational-Transactions)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

---

## 📌 Project Overview
This repository contains the complete implementation for **Topic 11: Database Design for Automation (SQLite, PostgreSQL & MySQL)**. It demonstrates enterprise-grade relational transaction handling in n8n.

The workflow implements a parent-child order processing pipeline that initializes relational tables, validates foreign keys, executes parameterized inserts, enforces customer credit limit constraints, performs automatic ACID rollbacks upon constraint violations, and retrieves completed orders via an indexed SQL JOIN.

---

## 📂 Repository Contents
| File | Description |
| :--- | :--- |
| **[`Topic 11 - Database Design for Automation.json`](./Topic%2011%20-%20Database%20Design%20for%20Automation.json)** | Complete 10-node n8n workflow with pre-configured pinData |
| **[`n8n_topic_11_requirement_document.md`](./n8n_topic_11_requirement_document.md)** | Full technical requirement specification & test matrix |
| **[`n8n_topic_11_requirement_document.pdf`](./n8n_topic_11_requirement_document.pdf)** | Formatted, print-ready requirement documentation PDF |
| **[`Topic_11_Implementation_Plan.md`](./Topic_11_Implementation_Plan.md)** | Step-by-step practical build guide and node blueprints |
| **[`Topic_11_Implementation_Plan.pdf`](./Topic_11_Implementation_Plan.pdf)** | Official assessment reference document |

---

## 🏗️ Workflow Architecture
```text
[ N11-01: Manual_Trigger ]
            |
            v
[ N11-02: Code_InitRelationalDB ] (Initializes tables & seed data)
            |
            v
[ N11-03: Code_StageIncomingOrder ] (Emits multi-item order)
            |
            v
[ N11-04: Code_InsertOrderHeader ] (Inserts parent header: PENDING)
            |
            v
[ N11-05: Code_InsertOrderItems ] (Inserts child items with Foreign Key)
            |
            v
[ N11-06: Code_UpdateCustomerBalance ] (ACID constraint check: balance <= limit)
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

---

## 🧪 Validation & Test Matrix
1. **Scenario 1 (Successful Commit - `CUST-001`):**
   - Current Balance: $1,200.00, Limit: $5,000.00, Order: $1,275.00.
   - Projected Balance: $2,475.00 $le$ $5,000.00.
   - Result: `isCommitted: true`, order committed, SQL JOIN executes, audit receipt generated.
2. **Scenario 2 (Credit Limit Breach & Rollback - `CUST-002`):**
   - Current Balance: $450.00, Limit: $500.00, Order: $1,275.00.
   - Projected Balance: $1,725.00 > $500.00.
   - Result: `isCommitted: false`, pending order and line items purged, logged to `Log_RollbackIncident`.

---

## 🚀 How to Import into n8n
1. Download **`Topic 11 - Database Design for Automation.json`**.
2. Open n8n, click **+ Add Workflow** $\rightarrow$ **Import from File**.
3. Click **Test Workflow**—all 10 nodes execute instantly using embedded test pinData!
