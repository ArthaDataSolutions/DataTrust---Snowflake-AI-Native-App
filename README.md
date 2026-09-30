# Snowflake Data Quality — Native App

A production-grade Data Quality platform delivered as a **Snowflake Native
Application** running on **Snowpark Container Services (SPCS)**, with a
React + FastAPI frontend and a Snowpark-Python rule engine.

## Features

- **Dynamic rule engine** covering NOT_NULL, UNIQUE, RANGE, REGEX,
  REFERENTIAL, CUSTOM_SQL, and STATISTICAL checks — bound to any table or
  column the consumer has granted to the application.
- **Definitions vs. applications** — `DQ_RULES` is the reusable catalog
  (definitions), `DQ_RULE_APPLICATIONS` holds the bindings a consumer
  creates when they click *Apply* in the UI (target table/column +
  resolved parameters). Execution runs against an application, so a
  single definition can be applied to many targets without duplication.
- **40 built-in system rules** across seven categories (Completeness,
  Uniqueness, Validity, Consistency, Timeliness, Accuracy, Referential)
  pre-seeded on install. System rules are immutable; consumers clone them
  to bind against their own tables.
- **Snowflake Cortex AI** integration — column profiling feeds
  `SNOWFLAKE.CORTEX.COMPLETE('mistral-large2', ...)` to generate rule
  suggestions with confidence scores. Cortex ML is used for z-score-based
  anomaly detection on historic pass-rate drift.
- **AI chat assistant (Cortex Analyst)** — a floating "Ask" widget lets you
  ask natural-language questions about your data quality (e.g. *"Which
  tables failed the most rules last week?"* or *"What's our DQ score for the
  Sales collection?"*) and get a plain-English answer, no SQL required.
  Questions are grounded on a built-in semantic view covering rules, rule
  applications, results, collections, and alerts, so answers stay scoped to
  your own data. You can `@`-mention a specific Rule, Collection, or
  Application to focus a question on it, and carry on a multi-turn
  conversation. Requires Cortex AI to be available in your account/region
  (see [Licensing tiers](#licensing-tiers) below).
- **ML-based pass-rate forecasting** — once a collection has at least 12 days
  of run history, the app trains a `SNOWFLAKE.ML.FORECAST` time-series model
  on its historical pass rates and generates a 14-day forward prediction with
  confidence intervals, shown as a trend line on the Collections page. If the
  forecast projects a drop below a 90% pass-rate threshold, the UI surfaces a
  "days until breach" warning so you can act before quality actually
  degrades. Forecasts regenerate automatically after each collection run.
- **Scheduling** via Snowflake Tasks with full lifecycle (create / pause /
  resume / delete) and a cron-expression validator.
- **Full audit trail** — every mutation is written to `DQ_AUDIT_LOG`.
- **Consumer-controlled access** — manifest `references` +
  `register_callback` pattern. The app only sees tables the consumer
  explicitly grants via `REFERENCE_USAGE` + `REGISTER_REFERENCE`.
- **Dashboard** — pass-rate trends, severity breakdowns, open alerts,
  scheduled-task health, audit feed, AI suggestions queue.

## Licensing tiers

The app runs in one of two tiers, detected automatically — there is nothing
to configure:

- **Trial** — install directly from the listing without purchasing. A small
  set of soft caps apply so you can evaluate the full feature set at low
  volume:

  | Capped resource | Trial limit |
  |---|---|
  | Collections | 1 |
  | Starter collections | 1 |
  | Alert channels | 1 |
  | Rule applications | 5 |

  Hitting a cap surfaces an in-app message telling you to upgrade; it never
  blocks you from viewing or exporting data you already have.
- **Paid** — once you purchase the listing, all caps above are lifted.

**Cortex AI is independent of licensing tier.** Whether the AI chat
assistant and AI rule-suggestion/anomaly-detection features are available
depends solely on whether Cortex AI is enabled and reachable in your
account/region (`SNOWFLAKE.CORTEX_USER` access) — not on whether you're on
trial or paid. A trial install with Cortex enabled gets the full AI
experience; a paid install without Cortex access will have those entry
points disabled until Cortex is available. If Cortex is unavailable, the
rest of the app (rule engine, scheduling, dashboard, forecasting) continues
to work normally.

## Privileges and references requested

| Privilege | Why |
|---|---|
| `EXECUTE TASK` | Create/manage Snowflake Tasks for scheduled rule execution. |
| `EXECUTE MANAGED TASK` | Snowflake-managed compute (Serverless Tasks). |
| `CREATE COMPUTE POOL` | Provision the SPCS compute pool that runs the backend/frontend containers. |
| `BIND SERVICE ENDPOINT` | Expose the frontend container as a public Snowflake service endpoint. |
| `CREATE WAREHOUSE` | Create a dedicated XS warehouse (`DQ_WH`) the backend uses to run Snowpark SQL. |
| `IMPORTED PRIVILEGES ON SNOWFLAKE DB` | Query `SNOWFLAKE.ACCOUNT_USAGE.QUERY_ATTRIBUTION_HISTORY` for actual credit-cost tracking per rule-evaluation run. |
| `SNOWFLAKE.CORTEX_USER` (database role) | Call Snowflake Cortex AI functions for rule-suggestion and anomaly detection. |

| Reference | Object type | Privilege on object | Purpose |
|---|---|---|---|
| `CONSUMER_TABLE_REF` | TABLE | SELECT | Grant the app read access to a specific table you want scanned (multi-valued — bind as many as you like). |
| `CONSUMER_JIRA_EAI_REF` | EXTERNAL_ACCESS_INTEGRATION | USAGE | Lets the app reach your Atlassian site for quick-create Jira tickets. Optional. |
| `CONSUMER_JIRA_SECRET_REF` | SECRET | USAGE, READ | Basic Auth secret (Atlassian email + API token) used to call the Jira REST API. Optional. |
| `CONSUMER_GITHUB_EAI_REF` | EXTERNAL_ACCESS_INTEGRATION | USAGE | Lets the app reach `api.github.com` for quick-create GitHub issues. Optional. |
| `CONSUMER_GITHUB_SECRET_REF` | SECRET | USAGE, READ | Fine-grained PAT used to call the GitHub REST API. Optional. |

Database- and schema-level access (as opposed to a single table via `CONSUMER_TABLE_REF`) is **not** a manifest reference — `DATABASE`/`SCHEMA` aren't valid manifest reference object types. Instead, grant `IMPORTED PRIVILEGES ON DATABASE <db>` (or the standard `GRANT SELECT`/`USAGE` for a non-shared database) to the application, then call `CALL <app>.DQ_CORE.REGISTER_REFERENCE('CONSUMER_DATABASE_REF', 'ADD', '<db>')` (or `'CONSUMER_SCHEMA_REF'` for a schema) so the app's Data Assets page can track it — this is app-side bookkeeping only, not a Snowflake-enforced reference.

All of the above are also declared in `manifest.yml` and surface in Snowsight's install/permissions UI — you do not need to run the raw SQL below unless you prefer the CLI/worksheet flow.

## Connectivity, data handling, and privacy disclosure

- **Internet endpoints**: None by default. `allowInternetEgress: false` on the deployed service spec — the app makes no outbound HTTP/S calls to the internet out of the box. All core communication is internal to your Snowflake account (Snowpark session, `SNOWFLAKE.CORTEX.COMPLETE`). The only exception is the **optional** Jira (`*.atlassian.net:443`) and GitHub (`api.github.com:443`) quick-ticket integrations, which only activate once you explicitly bind your own EAI + secret via the references above — the app cannot reach either service until you do.
- **External functions**: None used.
- **Cortex model**: The app calls `SNOWFLAKE.CORTEX.COMPLETE` with a fixed model, `mistral-large2` (not `'auto'`), for rule-suggestion and anomaly-detection features. Confirm `mistral-large2` is available in your region — see [Cross-region inference](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cross-region-inference).
- **Data logged/collected/stored**: Every mutation (rule create/update, rule application, execution, user/role change) is written to `DQ_AUDIT_LOG` inside the app's own `DQ_CORE` schema, which lives entirely inside your Snowflake account. Nothing is exported outside Snowflake or shared back to the provider.
- **Cookies**: None. The web UI authenticates via Snowflake/SPCS ingress identity headers, not browser cookies.

## Security guidance for consumers

- **Grant the narrowest scope you need.** `REFERENCE_USAGE` + `REGISTER_REFERENCE` accept a database, schema, or single table — prefer the table-level reference unless you genuinely need the app scanning an entire database.
- **Review requested privileges before granting them.** Snowsight's install/permission UI lists every privilege and reference this app requests, each with a description of why it's needed (see "Privileges and references requested" above) — review it rather than granting blanket access.
- **Use a dedicated warehouse.** The app provisions and uses its own `DQ_WH` (XS, auto-suspend) — it does not run on or consume your existing warehouses.
- **Report suspected security issues** to the provider contact in this listing, or via Snowflake's [app security appeal/report process](https://docs.snowflake.com/developer-guide/native-apps/security-appeal), rather than granting the app additional privileges to investigate.

## Architecture

```
┌────────────────────────────────────────────────┐
│ Consumer Account                               │
│  ┌──────────────────────────────────────────┐  │
│  │ Snowflake Native App (installed)         │  │
│  │  ├── DQ_CORE (schema)                    │  │
│  │  │    DQ_RULES, DQ_RESULTS, DQ_AUDIT …   │  │
│  │  │    SP_UPSERT_RULE, SP_EXECUTE_RULE …  │  │
│  │  └── SPCS Services                       │  │
│  │       ├── dq_backend  (FastAPI :8000)    │  │
│  │       └── dq_frontend (nginx  :80)       │  │
│  └──────────────────────────────────────────┘  │
│                 ▲                              │
│                 │ references (REGISTER_REFERENCE)
│  ┌──────────────────────────────────────────┐  │
│  │ Consumer data — ANY_DATABASE.ANY_SCHEMA… │  │
│  └──────────────────────────────────────────┘  │
└────────────────────────────────────────────────┘
```

## Directory layout

- `manifest.yml` — NAF manifest (privileges + references + containers)
- `setup.sql` — one-shot setup for application roles, tables, and SPs
- `service_specs/` — SPCS YAML specs for backend + frontend
- `container/backend/` — FastAPI + Snowpark-Python code and tests
- `container/frontend/` — React + Vite + Tailwind UI

## Installation (consumer side)

```sql
-- 1. Create the application from the share
CREATE APPLICATION dq_app FROM APPLICATION PACKAGE dq_app_pkg;

-- 2. Grant references for the objects you want scanned
GRANT REFERENCE_USAGE ON DATABASE SALES TO APPLICATION dq_app;
CALL dq_app.DQ_CORE.REGISTER_REFERENCE(
    'CONSUMER_TABLE_REF', 'ADD',
    SYSTEM$REFERENCE('TABLE', 'SALES.PUBLIC.ORDERS')
);

-- 3. Open the app UI via the URL shown in the install flow
SHOW ENDPOINTS IN SERVICE dq_app.DQ_CORE.DQ_FRONTEND;
```

See `docs/deployment-guide.md` for the provider-side packaging, image
build, and Marketplace publishing flow.

## Using the built-in sample rules

The app pre-seeds **40 built-in system rules** across seven categories
(Completeness, Uniqueness, Validity, Consistency, Timeliness, Accuracy,
Referential) on install (`SP_SEED_DEFAULT_RULES`) — no setup required to
try the app. System rules are read-only templates; to run one against
your own data:

1. Open the app UI (`SHOW ENDPOINTS IN SERVICE dq_app.DQ_CORE.DQ_FRONTEND`)
   and browse the **Rule Library**.
2. Pick a system rule (e.g. "Not Null Check") and click **Clone** — this
   creates an editable copy in your own rule catalog.
3. Click **Apply** on the cloned rule, choose the target table/column you
   granted via `REGISTER_REFERENCE` above, and resolve any parameters.
4. Run the rule application once manually, or attach it to a Snowflake
   Task schedule, to see pass/fail results on the Dashboard.
