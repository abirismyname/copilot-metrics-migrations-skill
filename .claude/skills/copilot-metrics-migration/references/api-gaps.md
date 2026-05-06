# Schema Transform and Metric Gaps

This migration is not a simple rename from one inline JSON schema to another. The metadata shows a
deeper change:

1. The legacy Copilot Metrics API returned aggregated metrics inline as nested day objects.
2. The current usage metrics surface returns report links first.
3. The actual downloaded report payloads use flatter aggregated or user-level records.

---

## Legacy vs Current Response Shape

### Legacy metrics response

The legacy API returns an array of `CopilotMetricsDay` objects. Each object has:

- `date`
- `total_active_users`
- `total_engaged_users`
- nested feature trees such as:
	- `copilot_ide_code_completions`
	- `copilot_ide_chat`
	- `copilot_dotcom_chat`
	- `copilot_dotcom_pull_requests`

The nesting is feature-first, then often editor/model/language/repository.

### Current report-link response

The current endpoints return a small wrapper object first:

- `download_links`
- either `report_day`
- or `report_start_day` and `report_end_day`

You must then fetch each signed URL in `download_links` to obtain the actual metrics JSON.

### Current downloaded report payloads

The metadata defines two downloaded payload families:

| Report type | Shape |
|---|---|
| Aggregated enterprise/org reports | Array of wrapper records with `day_totals[]`, `report_start_day`, `report_end_day` |
| User reports | Array of per-user daily records with `day`, `user_id`, `user_login`, `used_*`, `totals_by_*` |

---

## Concept Mapping

These are the closest concept mappings supported by the metadata.

| Legacy concept / field | Closest current field(s) | Notes |
|---|---|---|
| `date` | Aggregated `day_totals[].day`, user `day`, wrapper `report_day` | Not a direct rename. The report wrapper and the downloaded data use different date fields. |
| `total_active_users` | `day_totals[].daily_active_users`, `weekly_active_users`, `monthly_active_users` | Pick the field that matches the time window you actually need. |
| `total_engaged_users` | No exact equivalent in the provided report schemas | Nearest proxies are user-level `user_initiated_interaction_count > 0`, plus feature-specific actives such as `monthly_active_chat_users` and `monthly_active_agent_users`. |
| `copilot_ide_code_completions.languages[].total_code_suggestions` | `code_generation_activity_count`, `loc_suggested_to_add_sum` | Not 1:1. One is generation-event count; the other is suggested LoC. |
| `copilot_ide_code_completions.languages[].total_code_acceptances` | `code_acceptance_activity_count` | Broader than completion acceptance alone. |
| `copilot_ide_code_completions.languages[].total_code_lines_suggested` | `loc_suggested_to_add_sum` | Broader LoC metric across more surfaces. |
| `copilot_ide_code_completions.languages[].total_code_lines_accepted` | `loc_added_sum` | Broader than accepted completion lines; can include chat/apply and agent-edit writes. |
| `copilot_ide_chat.models[].total_chats` | `user_initiated_interaction_count` plus feature/mode rows in `totals_by_feature` | Not a direct chat-turn counter. |
| `copilot_ide_chat.models[].total_chat_insertion_events` | No exact equivalent | Nearest proxies are `code_acceptance_activity_count` and LoC-added metrics, but semantics differ. |
| `copilot_ide_chat.models[].total_chat_copy_events` | No exact equivalent | No dedicated copy-event field appears in the provided report schemas. |
| `copilot_dotcom_pull_requests.repositories[].models[].total_pr_summaries_created` | No exact equivalent | Current report schema exposes `pull_requests.*` lifecycle and suggestion fields instead. |
| `custom_model_training_date` | No exact equivalent in downloaded report schemas | Present in the legacy schema, absent from the current report schemas provided. |

---

## Structure Changes That Break Parsers

### Legacy nested objects are gone

If old code parses any of these objects directly, it must be rewritten:

- `copilot_ide_code_completions`
- `copilot_ide_chat`
- `copilot_dotcom_chat`
- `copilot_dotcom_pull_requests`

The current report schemas instead expose reusable flat breakdown arrays:

- `totals_by_feature`
- `totals_by_ide`
- `totals_by_language_feature`
- `totals_by_language_model`
- `totals_by_model_feature`
- `totals_by_cli`

### The old one-call flow is gone

If old code assumes:

1. call endpoint
2. parse metrics array directly

that logic must change to:

1. call report-link endpoint
2. read `download_links`
3. download each part
4. parse downloaded JSON

---

## New Fields and Dimensions Available in Current Reports

These capabilities are present in the current metadata and did not exist in the same form in the legacy API:

| Current field / concept | Why it matters |
|---|---|
| `pull_requests.*` | Adds PR lifecycle and Copilot review/suggestion metrics |
| `totals_by_cli`, `daily_active_cli_users` | CLI usage is now measurable |
| `totals_by_model_feature` | Model-level breakdowns for chat activity |
| `totals_by_language_model` | Language/model cross-sections |
| `used_chat`, `used_agent`, `used_cli` | Daily per-user usage booleans |
| `loc_deleted_sum`, `loc_suggested_to_delete_sum` | Delete-side LoC visibility |
| `last_known_ide_version`, `last_known_plugin_version` | Lets you reason about LoC telemetry coverage and client-version drift |
| `report_start_day`, `report_end_day` | Explicit report-window metadata |

---

## Gaps With No Direct Replacement

| Legacy field / API | Status | Migration note |
|---|---|---|
| Direct Data Access API | No replacement | Current metadata only covers report-link APIs and aggregated downloaded report data, not raw event streams. |
| Team-scoped metrics endpoints | No direct replacement | Build team views by joining user reports with team membership data outside the API. |
| `total_engaged_users` | No exact replacement in the provided report schemas | Rebuild the logic from active-user and user-level interaction signals, or document the semantic break. |
| `total_chat_insertion_events` | No exact replacement | Use LoC and acceptance proxies only if approximation is acceptable. |
| `total_chat_copy_events` | No exact replacement | No dedicated copy-event counter is present in the current report schemas provided. |
| `total_pr_summaries_created` | No exact replacement in the provided report schemas | Current report schema emphasizes pull request lifecycle and suggestion counts instead. |
| `custom_model_training_date` | No exact replacement in downloaded report schemas | Present in legacy nested model metrics, absent from the provided report schemas. |

---

## Practical Migration Rule

If old code is keyed to exact legacy field names, do not attempt a blind rename. First decide which current
report payload you need:

1. aggregated enterprise/org report
2. user report
3. one-day report
4. latest 28-day report

Then remap each consumer to the nearest current concept, documenting any semantics that changed.
*** Add File: /Users/abirmajumdar/workspaces/copilot-metrics-migrations-skill/.claude/skills/copilot-metrics-migration/references/operational-notes.md
# Operational Notes for Usage Metrics Reports

These notes come from the newer metadata files that explain how to interpret and reconcile usage metrics.
Use them whenever a migration question is really about trust, reconciliation, or data quality rather than
endpoint syntax.

---

## Reconciliation Rules

### Dashboards vs APIs vs downloaded reports

The metadata says these sources share underlying telemetry but aggregate it differently:

| Source | Window / granularity |
|---|---|
| Dashboard | 28-day rolling windows |
| API report endpoints | One day or latest 28-day report link |
| Downloaded report payloads | Daily rows or day-bucket arrays |

Do not promise that dashboard totals will exactly match a one-day downloaded report without aligning the time window.

### Recent days can look incomplete

Telemetry is processed asynchronously. The metadata says recent daily data typically finalizes within three full UTC days.
If a user reports missing recent activity, mention telemetry lag before assuming the migration is broken.

### `Unknown` values are normal

`Unknown` language, feature, or model values are expected when older clients do not emit enough detail.
The dashboard may hide those rows while API or report data still includes them.

---

## Telemetry Coverage Caveats

### IDE telemetry can suppress usage entirely

IDE-based Copilot usage metrics depend on telemetry from users' IDEs. If telemetry is disabled, that IDE activity
will not appear in dashboards, report endpoints, or downloaded report payloads.

### CLI usage is separate from IDE usage

`daily_active_cli_users` and `totals_by_cli` are collected separately. Do not add CLI activity into IDE-based active-user counts.

---

## LoC Metric Caveats

The Lines of Code metadata adds two important constraints:

1. LoC metrics are directional, not perfect source-of-truth accounting.
2. Coverage depends on IDE and plugin versions.

The report payloads include:

- `last_known_ide_version`
- `last_known_plugin_version`

Use those fields when a user asks why LoC totals look low or inconsistent after migration.

### Agent mode changes LoC interpretation

Agent workflows do not follow a simple suggest-then-accept model. Agent edits can add to:

- `loc_added_sum`
- `loc_deleted_sum`

without incrementing suggestion-style fields the way completion flows do. This is a semantic break from older
completion-centric metrics pipelines.

---

## Practical Guidance for Reporting

When a user compares old metrics pipelines to current usage reports, remind them of these likely causes for drift:

1. different time windows
2. report-link wrapper vs downloaded payload confusion
3. telemetry lag on recent days
4. hidden `Unknown` buckets in dashboards
5. IDE/plugin version coverage differences for LoC
6. CLI usage being separate from IDE activity

If the question is about field meaning rather than endpoint syntax, prefer citing these notes before proposing code changes.
