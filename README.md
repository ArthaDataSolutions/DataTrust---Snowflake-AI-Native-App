# Artha DataTrust — Snowflake Native Application

**Artha DataTrust** is a comprehensive Data Quality & Observability platform delivered as a Snowflake Native Application. Powered by Snowpark compute and Snowflake Cortex AI, Artha DataTrust enables continuous data quality monitoring, automated anomaly detection, natural-language triage with an AI assistant, and role-based data governance—all operating natively inside your Snowflake account.

---

## Key Features

- **Dynamic Rule Engine**: Evaluate Completeness, Validity, Uniqueness, Consistency, Timeliness, Accuracy, and Referential integrity directly against Snowflake tables.
- **40+ Built-in System Rules**: Pre-seeded production-ready rules (e.g. *No Null Or Zero*, *Valid Email Format*, *Positive Value Check*, *Duplicate Key Detection*).
- **Snowflake Cortex AI Assistant**: Natural-language inquiries ("Ask the DQ Assistant") powered by Cortex Analyst and grounded on your data quality results.
- **Automated Anomaly Detection & Forecasting**: Time-series pass rate forecasting using `SNOWFLAKE.ML.FORECAST` to warn about quality degradation before breaches occur.
- **Governance & RBAC**: In-app user invitation, role management (`ADMIN`, `EDITOR`, `VIEWER`), and team-based issue assignment.
- **Zero Data Egress**: All data processing, rule execution, and audit logging happen entirely within your Snowflake security boundary.

---

## Post-Installation Quick Start

Follow these steps immediately after installing **Artha DataTrust** from the Snowflake Marketplace.

### Step 1: Provision the Application Compute & Services

When installed on a fresh account, initialize the backend compute and table schemas:

```sql
-- Step 1: Start the application services and initialize the database objects
CALL ARTHA_DATATRUST.DQ_CORE.START_APP();

-- (Optional) If you ever need to retrieve the web interface URL via SQL:
CALL ARTHA_DATATRUST.DQ_CORE.GET_FRONTEND_URL();
```

*Note: Initial service provisioning takes approximately 1–3 minutes while the compute pool and container services initialize.*

### Step 2: Grant Table Access via Snowsight Permissions UI (Recommended)

Artha DataTrust utilizes the Snowflake **Permissions SDK Reference Model**, allowing you to grant table access directly from the Snowsight UI without writing SQL commands:

1. In Snowsight, navigate to **Data Products** > **Apps** and click **Artha DataTrust**.
2. Select the **Permissions** tab.
3. Under **Required Privileges & Object References**, locate `CONSUMER_TABLE_REF` (**Table to scan**).
4. Click **+ Add**, select your target database, schema, and table (for example, `SNOWFLAKE_SAMPLE_DATA.TPCH_SF1.CUSTOMER`), and click **Grant**.

### Step 3: Open the Web UI

Click the **Launch app** button on the application details page in Snowsight, or navigate to the web endpoint generated for your account.

---

## End-to-End Workflow: From Install to First Result

1. **Discover Assets**: Navigate to **Observe** > **Data Assets** in the app sidebar to confirm your granted tables and inspect column schemas.
2. **Browse & Apply Rules**:
   - Go to **Manage** > **Rule Library** to browse the 40+ pre-built system rules.
   - Click on a rule (e.g., *No Null Or Zero*) and click **Apply Rule**.
   - Select your target table (`CUSTOMER`) and column (`C_ACCTBAL`), then click **Apply to target**.
3. **Execute & Inspect Results**:
   - Open **Observe** > **Collections** and click **Run now** on your collection.
   - Go to **Observe** > **Runs** to view pass rates, duration, and execution history.
   - Go to **Observe** > **Issues** to triage failing records and assign remediation tasks.
4. **Interact with the AI Assistant**:
   - Click the floating **Ask the DQ Assistant** button in the bottom right corner.
   - Ask natural language questions like: *"Which tables had the most failures this week?"* or *"Show me all rules failing on the CUSTOMER table."*

---

## Privileges & Permissions Requested

| Privilege / Reference | Object Type | Purpose |
|---|---|---|
| `EXECUTE TASK` | Account | Schedule automated recurring data quality checks via Snowflake Tasks. |
| `EXECUTE MANAGED TASK` | Account | Serverless task execution for scheduled quality runs. |
| `CREATE COMPUTE POOL` | Account | Provision the Snowpark Container Services (SPCS) compute pool running the app UI & backend. |
| `BIND SERVICE ENDPOINT` | Account | Expose the secure web UI endpoint directly inside Snowsight. |
| `CREATE WAREHOUSE` | Account | Create a dedicated XS compute warehouse (`DQ_WH`) for running validation queries. |
| `IMPORTED PRIVILEGES ON SNOWFLAKE DB` | Database | Read query attribution history to track exact credit consumption per rule execution. |
| `SNOWFLAKE.CORTEX_USER` | Database Role | Access Snowflake Cortex AI functions for AI rule suggestions and the assistant chatbot. |
| `CONSUMER_TABLE_REF` | Table Reference | Secure `SELECT` access granted by the consumer on specific tables to evaluate data quality. |

---

## Consumer Stored Procedures & UDF Reference

Artha DataTrust provides consumer-accessible stored procedures in the `DQ_CORE` schema for automation, orchestration, and programmatic quality management:

### Application Lifecycle & Setup

- `START_APP()`: Initializes compute pools, backend/frontend container services, and seeds default rule definitions.
  ```sql
  CALL ARTHA_DATATRUST.DQ_CORE.START_APP();
  ```
- `STOP_APP()`: Suspends application services to conserve compute resources when not in use.
  ```sql
  CALL ARTHA_DATATRUST.DQ_CORE.STOP_APP();
  ```
- `FINALIZE_SETUP()`: Ensures core database tables (`DQ_RULES`, `DQ_RULE_APPLICATIONS`, `DQ_RESULTS`, `DQ_AUDIT_LOG`) and system rules are initialized.
  ```sql
  CALL ARTHA_DATATRUST.DQ_CORE.FINALIZE_SETUP();
  ```
- `GET_FRONTEND_URL()`: Returns the live web application ingress URL.
  ```sql
  CALL ARTHA_DATATRUST.DQ_CORE.GET_FRONTEND_URL();
  ```
- `GET_TRIAL_LIMIT(resource_name VARCHAR)` (UDF): Returns the usage limits for the trial tier.
  ```sql
  SELECT ARTHA_DATATRUST.DQ_CORE.GET_TRIAL_LIMIT('RULE_APPLICATIONS');
  ```

### Rule & Execution Management

- `SP_EXECUTE_RULE(application_id VARCHAR)`: Programmatically executes a specific rule application against its target table/column.
  ```sql
  CALL ARTHA_DATATRUST.DQ_CORE.SP_EXECUTE_RULE('app-guid-here');
  ```
- `SP_EXECUTE_COLLECTION(collection_id VARCHAR)`: Executes all rule applications grouped under a collection.
  ```sql
  CALL ARTHA_DATATRUST.DQ_CORE.SP_EXECUTE_COLLECTION('collection-guid-here');
  ```
- `SP_UPSERT_RULE(rule_payload VARCHAR)`: Creates or updates a custom rule definition using JSON specification.
  ```sql
  CALL ARTHA_DATATRUST.DQ_CORE.SP_UPSERT_RULE('{"name": "Custom Range Check", "category": "Validity", ...}');
  ```
- `SP_CREATE_RULE_APPLICATION(payload VARCHAR)`: Binds an existing rule definition to a target table and column.
  ```sql
  CALL ARTHA_DATATRUST.DQ_CORE.SP_CREATE_RULE_APPLICATION('{"rule_id": "...", "table_name": "CUSTOMER", "column_name": "C_ACCTBAL"}');
  ```
- `SP_GET_RUN_SUMMARY(run_id VARCHAR)`: Retrieves execution status, total checks evaluated, pass count, and error summary for a run.
  ```sql
  CALL ARTHA_DATATRUST.DQ_CORE.SP_GET_RUN_SUMMARY('run-guid-here');
  ```

### Licensing & Auditing

- `LICENSE_STATUS` (View): View current license tier (`TRIAL` or `PAID`), limits, and active counts.
  ```sql
  SELECT * FROM ARTHA_DATATRUST.DQ_CORE.LICENSE_STATUS;
  ```
- `DQ_AUDIT_LOG` (Table): Full audit trail of all rule mutations, executions, and user assignments.
  ```sql
  SELECT * FROM ARTHA_DATATRUST.DQ_CORE.DQ_AUDIT_LOG ORDER BY CREATED_AT DESC LIMIT 100;
  ```

---

## Network & Privacy Disclosures

- **No Outbound Internet Egress**: The application operates with `allowInternetEgress: false`. All operations are strictly local to your Snowflake account.
- **Model Inference**: AI features call Snowflake Cortex AI (`SNOWFLAKE.CORTEX.COMPLETE` using `mistral-large2`). No consumer data leaves Snowflake.
- **Data Governance**: Data scanned by rules is evaluated in-place in Snowflake memory. No customer table rows are permanently replicated or extracted outside the consumer's environment.

---

## Detailed User Tutorial & Documentation

For step-by-step visual guides with screenshots, refer to the documentation in [`docs/tutorial/`](../docs/tutorial/):
- [01. Installation & Table Permissions](../docs/tutorial/01-installation.md)
- [02. Data Assets Discovery](../docs/tutorial/02-data-assets.md)
- [03. Users, Roles & Teams Management](../docs/tutorial/03-users-and-teams.md)
- [04. Rule Library & Collections](../docs/tutorial/04-rule-library-and-collections.md)
- [05. Executing Runs & AI Assistant](../docs/tutorial/05-running-and-chatbot.md)

Online Documentation: [Artha DataTrust Docs](https://arthadatasolutions.github.io/DataTrust---Snowflake-AI-Native-App/)
