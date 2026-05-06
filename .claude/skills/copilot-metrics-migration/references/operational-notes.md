# Operational Notes

Use this file when the user asks why numbers differ between dashboards, APIs, and reports, or when a migration needs caveats around telemetry coverage, delayed data, or LoC interpretation.

---

## Reconciliation Rules

### Dashboard vs API vs report windows

The metadata describes different aggregation windows across surfaces:

| Surface | Window |
|---|---|
| Dashboard | rolling 28-day window |
| APIs and downloaded reports | daily records |

Migration implication:

- do not compare a daily report row directly to a dashboard tile without aligning the time window first
- for dashboard-like comparisons, prefer the 28-day report endpoints

### Recent days can look incomplete

Telemetry is processed asynchronously. The metadata says recent daily values may not fully settle for about three full UTC days.

Migration implication:

- avoid treating the most recent 1 to 3 days as final when validating a migration
- if a pipeline compares old and new values, compare older fully processed days

### `Unknown` values are expected

The metadata explains that `Unknown` can appear in language, feature, or model breakdowns when the client did not send enough detail.

Migration implication:

- do not treat `Unknown` as corruption
- note that dashboards may exclude `Unknown` while APIs and reports keep it for completeness

### Telemetry opt-out affects visibility

IDE-based Copilot metrics depend on client telemetry.

Migration implication:

- if a user or team is missing from reports, check IDE telemetry settings before assuming the API is broken

### CLI metrics are separate from IDE metrics

The metadata explicitly says CLI usage is collected separately and does not contribute to IDE active user counts.

Migration implication:

- do not add `daily_active_cli_users` into IDE DAU/WAU calculations unless you intentionally want a combined metric

---

## Lines Of Code Caveats

LoC metrics are powerful but easy to misuse.

### LoC coverage depends on client versions

The metadata lists minimum IDE and extension versions for LoC telemetry.

Migration implication:

- LoC fields may underreport if users are on older IDE or plugin versions
- use `last_known_ide_version` and `last_known_plugin_version` to assess rollout coverage

### Agent mode changes LoC semantics

The metadata distinguishes between:

- chat-visible suggested code, counted in `loc_suggested_to_add_sum`
- direct agent or edit-mode writes, counted in `loc_added_sum` and `loc_deleted_sum` under the `agent_edit` feature bucket

Migration implication:

- do not treat `loc_added_sum` as equivalent to legacy accepted completion lines
- expect higher totals once agent/edit-mode activity is included

---

## Validation Advice

When checking whether a migration is correct, validate in this order:

1. endpoint classification: confirm legacy `/copilot/metrics` calls were replaced or explicitly justified
2. report-link flow: confirm the code now handles `download_links`
3. downloaded schema parsing: confirm the code reads `day` or `day_totals[].day`, not legacy `date` at the same object level
4. metric semantics: confirm downstream dashboards were updated for any non-1:1 mappings
5. reconciliation: compare only fully processed days and aligned windows

---

## Good Explanations To Give Users

When users ask why the numbers changed after migration, the most defensible explanations from the metadata are:

- the current reports are daily while dashboards are rolling 28-day views
- recent days may be incomplete for up to about three UTC days
- LoC coverage depends on IDE and extension versions
- agent/edit mode now contributes added and deleted lines
- `Unknown` categories are normal in reports