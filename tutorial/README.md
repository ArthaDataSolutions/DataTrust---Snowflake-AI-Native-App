# Artha DataTrust — End-to-End User Tutorial

Welcome to the **Artha DataTrust** step-by-step user guide. This walkthrough guides you through installing the application, granting table access, configuring users and teams, applying data quality rules, executing checks, and leveraging the AI assistant.

---

## Tutorial Steps

| Step | Guide | Summary |
|---|---|---|
| **01** | **[Installation & Table Permissions](01-installation)** | Marketplace installation and granting table access using the Snowsight Permissions SDK interface. |
| **02** | **[Data Assets Discovery](02-data-assets)** | Inspecting registered databases, schemas, tables, and column metadata. |
| **03** | **[Users, Roles & Teams](03-users-and-teams)** | Inviting Snowflake users, assigning roles (`ADMIN`, `EDITOR`, `VIEWER`), and creating governance teams. |
| **04** | **[Rule Library & Collections](04-rule-library-and-collections)** | Browsing 40+ built-in system rules, applying rules to columns, and setting up check collections. |
| **05** | **[Executing Runs & AI Assistant](05-running-and-chatbot)** | Running check suites, viewing pass/fail issue breakdowns, and querying the Ask DQ Assistant chatbot. |

---

## Quick Reference Workflow

```
[ Snowflake Marketplace ] -> Install Artha DataTrust
           │
           ▼
[ Snowsight Permissions Tab ] -> Grant SELECT on Consumer Tables (e.g. CUSTOMER)
           │
           ▼
[ Artha DataTrust UI ] -> Verify Data Assets & Assign Teams
           │
           ▼
[ Rule Library ] -> Apply Pre-built Rules (e.g. No Null Or Zero on C_ACCTBAL)
           │
           ▼
[ Collections / Runs ] -> Execute Quality Checks & Review Issues
           │
           ▼
[ Ask DQ Assistant ] -> Natural Language Quality Inquiries
```
