# Endpoint Migration Map

Complete mapping of the retired Copilot Metrics API and the two retired early-access APIs to the
current usage metrics report surface described in the metadata.

The important nuance is that the current endpoints are **usage metrics endpoints that still live under
`/copilot/metrics/reports/...`**. Do not treat every `/copilot/metrics` path as legacy.

---

## Sunset Dates

| API | Sunset Date |
|---|---|
| User-level Feature Engagement Metrics API | March 2, 2026 |
| Direct Data Access API | March 2, 2026 |
| Copilot Metrics API | April 2, 2026 |

---

## Current Endpoint Families

These are the current usage metrics report endpoints from the metadata. They return signed download links,
not the metrics payload itself.

### Enterprise scope

| Purpose | Current endpoint |
|---|---|
| Aggregated report for one day | `GET /enterprises/{enterprise}/copilot/metrics/reports/enterprise-1-day?day=YYYY-MM-DD` |
| Aggregated latest 28-day report | `GET /enterprises/{enterprise}/copilot/metrics/reports/enterprise-28-day/latest` |
| User report for one day | `GET /enterprises/{enterprise}/copilot/metrics/reports/users-1-day?day=YYYY-MM-DD` |
| User latest 28-day report | `GET /enterprises/{enterprise}/copilot/metrics/reports/users-28-day/latest` |

### Organization scope

| Purpose | Current endpoint |
|---|---|
| Aggregated report for one day | `GET /orgs/{org}/copilot/metrics/reports/organization-1-day?day=YYYY-MM-DD` |
| Aggregated latest 28-day report | `GET /orgs/{org}/copilot/metrics/reports/organization-28-day/latest` |
| User report for one day | `GET /orgs/{org}/copilot/metrics/reports/users-1-day?day=YYYY-MM-DD` |
| User latest 28-day report | `GET /orgs/{org}/copilot/metrics/reports/users-28-day/latest` |

---

## Legacy to Current Mappings

### Enterprise-level metrics

| Item | Details |
|---|---|
| **Deprecated** | `GET /enterprises/{enterprise}/copilot/metrics` |
| **Current replacements** | `GET /enterprises/{enterprise}/copilot/metrics/reports/enterprise-1-day?day=YYYY-MM-DD` or `GET /enterprises/{enterprise}/copilot/metrics/reports/enterprise-28-day/latest` |
| **Parameter change** | Legacy `since`, `until`, `page`, `per_page` do not carry over. Current API uses either one required `day` query parameter or a fixed latest 28-day endpoint. |
| **Response change** | Legacy endpoint returned metrics inline. Current endpoint returns `download_links` plus `report_day` or `report_start_day` / `report_end_day`; you then fetch the signed URLs. |
| **Policy note** | The current metadata requires the `Copilot usage metrics` policy to be enabled. |

### Organization-level metrics

| Item | Details |
|---|---|
| **Deprecated** | `GET /orgs/{org}/copilot/metrics` |
| **Current replacements** | `GET /orgs/{org}/copilot/metrics/reports/organization-1-day?day=YYYY-MM-DD` or `GET /orgs/{org}/copilot/metrics/reports/organization-28-day/latest` |
| **Parameter change** | Same shift from paged `since` / `until` requests to one-day or latest-28-day report requests. |
| **Response change** | Same two-step report-link flow as enterprise endpoints. Org endpoints may return `204` with no content when no report is available. |
| **Policy note** | The current metadata requires the `Copilot usage metrics` policy to be enabled. |

### Enterprise-team metrics

| Item | Details |
|---|---|
| **Deprecated** | `GET /enterprises/{enterprise}/team/{team_slug}/copilot/metrics` |
| **Current replacement** | **No direct team-scoped replacement in the provided metadata** |
| **Workaround** | Use enterprise user reports (`users-1-day` or `users-28-day/latest`) and join them with team roster data externally. |
| **Implementation note** | The migration guide in metadata includes a `gh api` + `jq` script for exporting enterprise team memberships, which can be used for the join step. |

### Organization-team metrics

| Item | Details |
|---|---|
| **Deprecated** | `GET /orgs/{org}/team/{team_slug}/copilot/metrics` |
| **Current replacement** | **No direct team-scoped replacement in the provided metadata** |
| **Workaround** | Use organization user reports (`users-1-day` or `users-28-day/latest`) and filter or join the user rows against the team membership list outside the API. |

---

## Early-Access API Mappings

### User-level Feature Engagement Metrics API

| Item | Details |
|---|---|
| **Deprecated** | User-level Feature Engagement Metrics API |
| **Current replacement** | User report endpoints (`/copilot/metrics/reports/users-1-day` and `/users-28-day/latest`) |
| **Nearest fields** | `used_chat`, `used_agent`, and `used_cli` in downloaded user report records |
| **Gap** | Old feature-engagement data was a simpler adoption-oriented API. Current user reports provide daily usage booleans and richer telemetry rather than the same point-in-time binary model. |

### Direct Data Access API

| Item | Details |
|---|---|
| **Deprecated** | Direct Data Access API |
| **Current replacement** | **None** |
| **Gap** | The new report endpoints provide aggregated daily report data, not raw per-event telemetry. |
| **Workaround** | Use user report endpoints as a coarse proxy, or move team- and user-level joins into your own data pipeline. |

---

## Scan Guidance

### Always flag these patterns

```
/enterprises/{var}/copilot/metrics
/enterprises/{var}/team/{var}/copilot/metrics
/orgs/{var}/copilot/metrics
/orgs/{var}/team/{var}/copilot/metrics
user_feature_engagement
direct_data_access
```

### Always exclude these patterns

```
/copilot/metrics/reports/
/copilot/metrics/reports/enterprise-1-day
/copilot/metrics/reports/enterprise-28-day/latest
/copilot/metrics/reports/organization-1-day
/copilot/metrics/reports/organization-28-day/latest
/copilot/metrics/reports/users-1-day
/copilot/metrics/reports/users-28-day/latest
download_links
report_day
report_start_day
report_end_day
```

### Practical detection rule

Treat `/copilot/metrics` as legacy **only** when it terminates the endpoint path or is followed by query text.
If the path continues with `/reports/`, classify it as current usage metrics code.
