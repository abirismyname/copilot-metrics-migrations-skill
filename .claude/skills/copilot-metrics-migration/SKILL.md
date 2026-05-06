---
name: copilot-metrics-migration
description: >
  Scan a repository or folder for calls to the deprecated GitHub Copilot Metrics API (/copilot/metrics),
  report every occurrence with file and line references, and suggest exact migration steps to the current
  Copilot usage metrics report endpoints. Use this skill whenever the user runs /compatibility-scan,
  asks about migrating from the Copilot Metrics API, mentions deprecated Copilot API endpoints,
  wants to know if their codebase still calls legacy /copilot/metrics endpoints, or asks about the
  sunset of the old Copilot metrics API. Also trigger when users mention "copilot metrics migration",
  "API compatibility", or "usage metrics upgrade".
---

# Copilot Metrics Migration Skill

Help users identify and migrate code that calls the deprecated GitHub Copilot Metrics API to the current
Copilot usage metrics report endpoints. The legacy metrics API sunset on **April 2, 2026**.

Two distinctions matter:

1. Legacy metrics calls such as `GET /orgs/{org}/copilot/metrics` are deprecated and should be flagged.
2. Current usage report endpoints such as `GET /orgs/{org}/copilot/metrics/reports/organization-28-day/latest`
   are valid and should **not** be flagged, even though the path still contains `/copilot/metrics`.

## Reference Files

Load these files when you need detail — don't paraphrase from memory:

| File | When to load |
|---|---|
| `references/endpoint-map.md` | For exact old→new endpoint mappings, report-link endpoint coverage, and team-level gaps |
| `references/api-gaps.md` | For response-shape changes, concept mappings, and metrics with no direct replacement |
| `references/operational-notes.md` | For reconciliation caveats, LoC telemetry limits, and dashboard vs API differences |

---

## Slash Command: /compatibility-scan

**Syntax:** `/compatibility-scan [path]`

`path` is optional. If omitted, scan the current workspace root. The user may pass a directory path,
a single file path, or a glob pattern.

---

## Scan Procedure

When `/compatibility-scan` is invoked, or when the user asks you to scan their code:

### Step 1 — Locate files

Search all files in the target path. Focus on:
- JavaScript / TypeScript (`.js`, `.ts`, `.mjs`, `.cjs`, `.jsx`, `.tsx`)
- Python (`.py`)
- Shell scripts (`.sh`, `.bash`, `.zsh`)
- Ruby (`.rb`), Go (`.go`), Java (`.java`), C# (`.cs`)
- GitHub Actions workflows (`.yml`, `.yaml` under `.github/`)
- Jupyter notebooks (`.ipynb`)
- Plain text or config files if they contain URLs (`.env`, `.md`, `.txt`, `.json`, `.toml`)

### Step 2 — Detect deprecated patterns

Flag legacy endpoints and legacy early-access APIs, but explicitly exclude current report endpoints.

#### Flag these legacy endpoint patterns

```
/enterprises/{enterprise}/copilot/metrics
/enterprises/{enterprise}/team/{team_slug}/copilot/metrics
/orgs/{org}/copilot/metrics
/orgs/{org}/team/{team_slug}/copilot/metrics
```

#### Flag these legacy early-access patterns

```
user_feature_engagement
direct_data_access
early-access/copilot
```

#### Do not flag these current usage report endpoint patterns

```
/copilot/metrics/reports/
/copilot/metrics/reports/enterprise-1-day
/copilot/metrics/reports/enterprise-28-day/latest
/copilot/metrics/reports/users-1-day
/copilot/metrics/reports/users-28-day/latest
/copilot/metrics/reports/organization-1-day
/copilot/metrics/reports/organization-28-day/latest
```

**Common call-site patterns (use regex):**
- Flag legacy calls:
  - `gh api .*?/copilot/metrics(["' ?]|$)`
  - `curl .*?/copilot/metrics(["' ?]|$)`
  - `fetch\(.*?/copilot/metrics(["' ?]|$)`
  - `axios\.(get|post)\(.*?/copilot/metrics(["' ?]|$)`
  - `octokit\.request.*?/copilot/metrics(["' ?]|$)`
  - `requests\.(get|post)\(.*?/copilot/metrics(["' ?]|$)`
- Exclude current calls:
  - any URL containing `/copilot/metrics/reports/`

### Step 3 — Classify each finding

For each match, identify whether it is a deprecated call, an early-access legacy API, or already-migrated code.

| Pattern found | Endpoint class |
|---|---|
| `/enterprises/.*/copilot/metrics` | Legacy enterprise metrics endpoint |
| `/enterprises/.*/team/.*/copilot/metrics` | Legacy enterprise-team metrics endpoint |
| `/orgs/.*/copilot/metrics` | Legacy organization metrics endpoint |
| `/orgs/.*/team/.*/copilot/metrics` | Legacy organization-team metrics endpoint |
| `user_feature_engagement` | User-level Engagement API (early-access) |
| `direct_data_access` | Direct Data Access API (early-access) |
| `/copilot/metrics/reports/` | Current usage metrics report endpoint |
| Any `copilot/metrics` not matching above | Generic / unknown pattern; inspect manually |

---

## Output Report Format

Always produce a structured report in this exact order. Do not skip sections even if empty.

---

### Section 1 — Summary

```
## Compatibility Scan Report

**Scan path:** [the path that was scanned]
**Total files scanned:** [N]
**Files with legacy API calls:** [N]
**Legacy call sites found:** [N]
**Current usage report call sites found:** [N]
**Status:** [✅ No legacy issues found | ⚠️ Migration required]
```

If there are no legacy findings and no current report findings, stop here and add:
> No Copilot metrics or early-access metrics API calls were detected.

If there are no legacy findings but current report endpoints are present, stop here and add:
> No deprecated Copilot metrics endpoints were detected. The codebase already appears to be using the
> current usage metrics report API.

---

### Section 2 — Findings by File

Only include if there are findings. List files in order of most legacy findings first.

For each file:
```
### `path/to/file.ext`

| Line | Snippet | Endpoint Type | Severity |
|------|---------|---------------|----------|
| 42   | `gh api /enterprises/acme/copilot/metrics` | Legacy enterprise metrics endpoint | 🔴 Broken |
| 87   | `fetch('/orgs/acme/copilot/metrics')` | Legacy organization metrics endpoint | 🔴 Broken |
| 113  | `gh api /orgs/acme/copilot/metrics/reports/users-28-day/latest` | Current usage report endpoint | 🟢 Current |
```

Severity levels:
- 🔴 **Broken** — Sunset already passed; the call is legacy and should be migrated.
- 🟠 **Gap** — No direct replacement exists; workaround is required.
- 🟢 **Current** — Endpoint is already on the current report API and should not be changed just because it contains `/copilot/metrics`.

---

### Section 3 — Migration Instructions

For each unique legacy endpoint type found, provide a migration block. Load `references/endpoint-map.md`
for the authoritative details.

**Template per endpoint type:**

```
#### [Endpoint Type Name]

**Old endpoint:**
GET /orgs/{org}/copilot/metrics

**Current replacement:**
GET /orgs/{org}/copilot/metrics/reports/organization-1-day?day=YYYY-MM-DD
or
GET /orgs/{org}/copilot/metrics/reports/organization-28-day/latest

**Migration pattern:**
1. Call the report-link endpoint.
2. Read `download_links` from the response.
3. Download each signed URL.
4. Parse the downloaded JSON report payload.

**Path note:** [Only add if there is no direct replacement or if scope changes]

**Policy note:** [Describe the required usage metrics policy enablement]

**Response changes:** [Describe the wrapper response and downloaded report payload shape]
```

If a team endpoint is found, explicitly state that the current metadata contains **no direct team-level report endpoint**.
Recommend the org- or enterprise-level user report endpoints plus an external team-membership join.

For **Direct Data Access API** findings, include this warning:
> ⚠️ **No direct replacement exists** for the Direct Data Access API. The new Usage Metrics API
> report surface provides signed report links and aggregated daily data, not raw per-event telemetry.
> See `references/api-gaps.md` for migration limits and `references/endpoint-map.md` for workaround guidance.

---

### Section 4 — Data Model Checklist

Only include if any legacy Metrics API calls were found. Load `references/api-gaps.md` for the full table.

Produce a checklist of parser and schema updates the developer will need to apply:

```
#### Data model updates required

- [ ] Replace inline response parsing with a two-step report-link download flow
- [ ] Map legacy `date` to downloaded report `day` or to `day_totals[].day`, depending report type
- [ ] Replace nested `copilot_ide_code_completions` parsing with `totals_by_feature`, `totals_by_ide`, and `totals_by_language_feature`
- [ ] Replace nested chat parsing with `user_initiated_interaction_count` and feature/model breakdown arrays
- [ ] Re-evaluate any logic depending on `total_engaged_users`; there is no exact equivalent in the provided report schemas
- [ ] Re-evaluate any logic depending on `total_code_suggestions` or `total_chats`; nearest report metrics are not 1:1 replacements
```

Then append a "New fields available" note:
```
#### New fields available (not in old API)

These fields are available in the current report payloads and may be valuable for new analytics:
- `pull_requests.*` — PR lifecycle and Copilot review/suggestion activity
- `totals_by_model_feature` — model-level chat activity breakdown
- `totals_by_language_model` — language/model breakdowns
- `totals_by_cli` and `daily_active_cli_users` — Copilot CLI usage
- `loc_deleted_sum` and `loc_suggested_to_delete_sum` — deletion-side LoC metrics
- `last_known_ide_version` and `last_known_plugin_version` — telemetry coverage diagnostics
- `used_chat`, `used_agent`, and `used_cli` — user-level daily usage booleans
```

---

### Section 5 — Next Steps

Always include this closing section:

```
## Next Steps

1. **Enable the Copilot Metrics API policy** in your organization settings:
   Settings → Copilot → Policies → Enable Copilot Metrics API

2. **Update endpoint URLs** using the substitutions in Section 3 above.

3. **Update parser logic** in your response-handling code using the checklist in Section 4.

4. **Validate** by calling the report-link endpoint, downloading one report part, and confirming the downloaded JSON shape.

5. **Remove legacy fallbacks** once validation passes.

**Resources:**
- Current report API docs: https://docs.github.com/rest/copilot/copilot-usage-metrics
- Migration guide: https://github.blog/changelog/2026-01-29-closing-down-notice-of-legacy-copilot-metrics-apis/
- Metrics reference: https://docs.github.com/en/enterprise-cloud/copilot/reference/copilot-usage-metrics/copilot-usage-metrics
```

---

## Edge Cases

**The user asks about a field that no longer exists:**
Consult `references/api-gaps.md`. If it's in the "Gaps — Old Fields With No New Equivalent" section,
tell the user there is no direct replacement and explain the closest alternative.

**The user asks about the Direct Data Access API:**
Always flag this as a gap. There is no direct replacement. Suggest the aggregated user-level records
as an approximation, or mention enterprise data export options.

**The scan finds `/copilot/metrics/reports/...`:**
Treat that as current usage metrics code, not as a legacy finding. Mention it in the report as already migrated.

**Mixed codebases (multiple languages):**
Report findings grouped by file. Provide language-appropriate code snippets for the migration
instructions (for example, show the new two-step `requests` or Octokit flow for the report-link API).

**The scan finds no matches:**
Report cleanly in Section 1 and stop. Do not fabricate findings.

**The user provides only a snippet, not a file path:**
Analyze the snippet directly and produce Sections 3, 4, and 5 (skip 1 and 2 since there's no file to report).
