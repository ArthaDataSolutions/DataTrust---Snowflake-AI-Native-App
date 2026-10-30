# Artha DataTrust

**Artha DataTrust** is an enterprise Data Quality & Observability platform delivered as a Snowflake Native Application. Powered by Snowpark compute and Snowflake Cortex AI, Artha DataTrust enables continuous data quality monitoring, automated anomaly detection, natural-language triage with an AI assistant, and role-based data governance—operating natively inside your Snowflake account with zero data egress.

This guide takes you from Snowflake Marketplace installation to your first data quality check. Plan for about 10–15 minutes.

---

## Before You Start

| Requirement | Why It Is Needed |
|---|---|
| **`ACCOUNTADMIN`** (or a role with privileges to install Native Apps) | The app provisions SPCS compute pools, dedicated XS query warehouses, and background tasks. |
| **Snowpark Container Services (SPCS)** | The app UI (React) and backend (FastAPI) run in secure container services. |
| **Snowflake Cortex AI** (`SNOWFLAKE.CORTEX_USER`) | Required for the natural language DQ Assistant, automated collection planning, and anomaly insights. |
| **At least one table to monitor** | E.g. your business tables or the sample `SNOWFLAKE_SAMPLE_DATA.TPCH_SF1.CUSTOMER` dataset. |

---

## Step-by-Step Installation & Quick Start

### 1. Get the App from Snowflake Marketplace
1. In Snowsight, navigate to **Snowflake Marketplace** in the left sidebar.
2. Search for **Artha DataTrust** and click **Get**.
3. If prompted:
   - Complete your consumer profile (First Name, Last Name, Email).
   - Select or create a warehouse (`X-Small` recommended for installation).
4. When installation finishes, navigate to **Data Products** > **Apps** and open **Artha DataTrust**.

---

### 2. Grant Required Permissions (Permissions Tab)
Before the app can provision its compute containers or evaluate tables, review and grant permissions in the Snowsight UI:

1. On the app details page in Snowsight, click the **Permissions** tab.
2. Grant the requested **Account level privileges**:
   - `EXECUTE TASK` & `EXECUTE MANAGED TASK`: For automated and serverless scheduled quality runs.
   - `CREATE COMPUTE POOL`: To run the backend and frontend web application containers in Snowpark Container Services (SPCS).
   - `BIND SERVICE ENDPOINT`: To expose the secure web UI in Snowsight.
   - `CREATE WAREHOUSE`: To manage the dedicated XS query warehouse (`DQ_WH`).
   - `IMPORTED PRIVILEGES ON SNOWFLAKE DB`: To track execution cost and query telemetry.
3. Grant the **Database Role**:
   - `SNOWFLAKE.CORTEX_USER`: To enable the Cortex AI Assistant, rule recommendations, and anomaly detection.

---

### 3. Grant Table or Schema Access
Artha DataTrust supports granting access either via the UI Object Picker or SQL commands:

#### Option A: Snowsight UI Object Picker (Recommended)
1. Under **Object access privileges**, locate **Table to scan** (`CONSUMER_TABLE_REF`).
2. Click **+ Add** (or **Review**).
3. Browse your database hierarchy and pick the target table(s) to monitor (e.g. `SNOWFLAKE_SAMPLE_DATA.TPCH_SF1.CUSTOMER` or your own test data).
4. Click **Done** and confirm **Grant**.

#### Option B: Schema / Database USAGE Grants (SQL)
To monitor entire schemas or databases, run:
```sql
-- 1. Grant USAGE on Database to the Application
GRANT USAGE ON DATABASE <YOUR_DB> TO APPLICATION ARTHA_DATATRUST;

-- 2. Grant USAGE and SELECT on Schema
GRANT USAGE ON SCHEMA <YOUR_DB>.<YOUR_SCHEMA> TO APPLICATION ARTHA_DATATRUST;
GRANT SELECT ON ALL TABLES IN SCHEMA <YOUR_DB>.<YOUR_SCHEMA> TO APPLICATION ARTHA_DATATRUST;

-- 3. Register the Schema Reference inside the App
CALL ARTHA_DATATRUST.DQ_CORE.REGISTER_REFERENCE('CONSUMER_SCHEMA_REF', 'ADD', '<YOUR_DB>.<YOUR_SCHEMA>');
```
*Note: Registering a schema automatically triggers the auto-generation of an initial starter collection in Data Assets.*

---

### 4. Launch App & Run Your First Quality Check
1. Click **Launch app** (or retrieve the web URL via `CALL ARTHA_DATATRUST.DQ_CORE.GET_FRONTEND_URL()`).
2. **Data Assets**: Verify your granted tables and columns appear under **Observe** > **Data Assets**.
3. **Rule Library**: Browse 40+ system rules (Completeness, Uniqueness, Validity, Consistency, Timeliness) and apply them to target columns.
4. **Collections**: Click **Run now** on your collection to evaluate rules on Snowflake compute.
5. **Issues & Runs**: View pass/fail scores, failed record samples, and assign remediation tasks.
6. **Ask the DQ Assistant**: Click the AI Assistant widget in the bottom right corner to ask questions in plain English (*"Which tables failed rules this week?"*) or ask it to generate and schedule new collections for you.

---

## Granting Access to Other Users

By default, the role that installed the application has administrative access. To grant access to other Snowflake users and roles in your organization:

```sql
-- Grant Admin role to a data lead role:
GRANT APPLICATION ROLE ARTHA_DATATRUST.DQ_ADMIN TO ROLE DATA_ENGINEER;

-- Grant Read-Only viewer role to business analyst roles:
GRANT APPLICATION ROLE ARTHA_DATATRUST.DQ_VIEWER TO ROLE ANALYST;
```

Inside the app UI, administrators can also invite team members under **Access & Users** and organize them into Governance Teams.

---

## Managing Costs & Stopping the App

- **Suspension**: You can suspend the web compute services at any time when not actively using the UI to conserve Snowflake credits:
  ```sql
  CALL ARTHA_DATATRUST.DQ_CORE.STOP_APP();
  ```
- **Resume**: Start the services again with:
  ```sql
  CALL ARTHA_DATATRUST.DQ_CORE.START_APP();
  ```
- All your configured rules, collections, issues, and historical test results remain saved in database storage while services are stopped.

---

## Troubleshooting Quick Guide

| Issue | Cause | Solution |
|---|---|---|
| **App fails on first launch / start** | Missing account privileges | Ensure all account privileges in the **Permissions** tab are granted. |
| **Assistant doesn't respond** | Missing Cortex role | Ensure `SNOWFLAKE.CORTEX_USER` role is granted to the application. |
| **No tables in Data Assets** | No tables granted yet | Use the **Permissions** tab or run `REGISTER_REFERENCE` for your schema. |
| **`GET_FRONTEND_URL` returns empty/pending** | Services still provisioning | SPCS containers take 1–3 minutes on initial start. Wait 60 seconds and retry. |

---

## Detailed Step-by-Step Visual Tutorial

For a complete walkthrough with high-resolution screenshots for each page, check out the tutorial modules in [`docs/tutorial/`](docs/tutorial/):

- **[01. Installation & Table Permissions](docs/tutorial/01-installation.md)**
- **[02. Data Assets Discovery](docs/tutorial/02-data-assets.md)**
- **[03. Users, Roles & Teams Management](docs/tutorial/03-users-and-teams.md)**
- **[04. Rule Library & Collections](docs/tutorial/04-rule-library-and-collections.md)**
- **[05. Executing Runs & AI Assistant](docs/tutorial/05-running-and-chatbot.md)**

Online Documentation: [Artha DataTrust Documentation Site](https://arthadatasolutions.github.io/DataTrust---Snowflake-AI-Native-App/)

---

## Stored Procedures & UDF Reference

| Command | Purpose |
|---|---|
| `CALL ARTHA_DATATRUST.DQ_CORE.START_APP();` | Initializes SPCS compute pools, backend/frontend services, and seeds system rules. |
| `CALL ARTHA_DATATRUST.DQ_CORE.STOP_APP();` | Suspends application services to conserve compute when not in use. |
| `CALL ARTHA_DATATRUST.DQ_CORE.GET_FRONTEND_URL();` | Returns the live web ingress URL. |
| `CALL ARTHA_DATATRUST.DQ_CORE.SP_EXECUTE_COLLECTION('collection_id');` | Runs all rules grouped under a collection on demand. |
| `SELECT * FROM ARTHA_DATATRUST.DQ_CORE.LICENSE_STATUS;` | Checks active license tier (`TRIAL` vs. `PAID`) and quota usage. |
| `SELECT * FROM ARTHA_DATATRUST.DQ_CORE.DQ_AUDIT_LOG ORDER BY CREATED_AT DESC;` | Audits quality checks, rule modifications, and user actions. |

---

## Privacy & Security

- **Zero Data Egress**: All data scanning and evaluation execute strictly within your Snowflake account. No customer table rows are transferred outside your security boundary (`allowInternetEgress: false`).
- **Governed AI Inference**: AI features run exclusively on Snowflake Cortex AI (`SNOWFLAKE.CORTEX.COMPLETE` / Cortex Analyst). Data stays protected under Snowflake Horizon governance.
