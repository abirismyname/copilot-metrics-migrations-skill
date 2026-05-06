# Copilot Metrics Migration Skill

This repository contains an agent skill that scans a repository for legacy GitHub Copilot metrics integrations and helps migrate them to the current usage metrics report endpoints.

It is designed for:

- deprecated `GET /copilot/metrics` integrations
- retired early-access Copilot metrics APIs such as feature engagement and direct data access
- migrations to the current report-link endpoints under `/copilot/metrics/reports/...`
- explaining schema changes, telemetry caveats, and team-reporting gaps

## What The Skill Does

The skill looks for legacy Copilot metrics calls in source code, scripts, workflows, and config files, then produces a structured migration report that:

1. lists each deprecated call site with file and line context
2. distinguishes legacy `/copilot/metrics` endpoints from current `/copilot/metrics/reports/...` endpoints
3. maps each legacy endpoint to the nearest current replacement
4. highlights schema and parser changes needed for the new report-download flow
5. calls out hard gaps, such as the missing direct replacement for team-scoped reports and the Direct Data Access API

## Repository Layout

```text
.claude/skills/copilot-metrics-migration/
├── SKILL.md
└── references/
    ├── api-gaps.md
    ├── endpoint-map.md
    └── operational-notes.md

evals/
└── evals.json

metadata/
└── source material used to build the skill
```

## Install On Claude Code

Claude Code reads project skills from the `.claude/skills` directory in your repository and personal skills from `~/.claude/skills` in your home directory.

### Project install

Copy this folder into the target repository:

```bash
mkdir -p .claude/skills
cp -R .claude/skills/copilot-metrics-migration /path/to/your-project/.claude/skills/
```

Or, if you already cloned this repo and want to reuse the folder directly:

```bash
cp -R /path/to/copilot-metrics-migrations-skill/.claude/skills/copilot-metrics-migration \
  /path/to/your-project/.claude/skills/
```

### Personal install

Install it once for all your local projects:

```bash
mkdir -p ~/.claude/skills
cp -R /path/to/copilot-metrics-migrations-skill/.claude/skills/copilot-metrics-migration \
  ~/.claude/skills/
```

### Use it in Claude Code

Start Claude Code in the target repository:

```bash
cd /path/to/your-project
claude
```

Then ask for a scan in natural language, for example:

```text
Scan this repo for legacy Copilot metrics API usage and show me the migration plan.
```

Or use the slash-style prompt text the skill is written around:

```text
/compatibility-scan .
```

If the skill does not seem to load, open a fresh Claude Code session in the repository and use Claude Code's config-debugging commands such as `/context` or `/doctor` to confirm the `.claude/skills` directory was picked up.

## Install On GitHub Copilot In VS Code

GitHub Copilot supports project skills stored in any of these repository locations:

- `.github/skills/<skill-name>/SKILL.md`
- `.claude/skills/<skill-name>/SKILL.md`
- `.agents/skills/<skill-name>/SKILL.md`

Because this repository already uses `.claude/skills`, the easiest path is to copy the existing skill directory as-is.

### Option 1: keep the Claude-compatible layout

```bash
mkdir -p /path/to/your-project/.claude/skills
cp -R /path/to/copilot-metrics-migrations-skill/.claude/skills/copilot-metrics-migration \
  /path/to/your-project/.claude/skills/
```

### Option 2: use the GitHub-specific layout

```bash
mkdir -p /path/to/your-project/.github/skills
cp -R /path/to/copilot-metrics-migrations-skill/.claude/skills/copilot-metrics-migration \
  /path/to/your-project/.github/skills/
```

### Use it in VS Code

1. Open the target repository in VS Code.
2. Start a new GitHub Copilot agent or chat session.
3. Ask for the migration directly, for example:

```text
Run a compatibility scan for legacy Copilot metrics API usage in this repo.
```

or:

```text
/compatibility-scan .
```

If the skill was added while VS Code was already open, start a fresh chat or reload the window before testing.

## Install On GitHub Copilot CLI

GitHub Copilot CLI supports project and personal skill directories.

Project locations:

- `.github/skills/`
- `.agents/skills/`
- `.claude/skills/`

Personal locations:

- `~/.copilot/skills/`
- `~/.agents/skills/`

### Project install for Copilot CLI

```bash
mkdir -p /path/to/your-project/.github/skills
cp -R /path/to/copilot-metrics-migrations-skill/.claude/skills/copilot-metrics-migration \
  /path/to/your-project/.github/skills/
```

You can also place it in `.claude/skills` or `.agents/skills` if that fits your setup better.

### Personal install for Copilot CLI

```bash
mkdir -p ~/.copilot/skills
cp -R /path/to/copilot-metrics-migrations-skill/.claude/skills/copilot-metrics-migration \
  ~/.copilot/skills/
```

### Reload and verify in Copilot CLI

Start a new CLI session, or if a session is already running, reload skills:

```text
/skills reload
```

Then verify that the skill is available:

```text
/skills info copilot-metrics-migration
```

Then run it with a prompt such as:

```text
Scan this repository for deprecated Copilot metrics endpoints and show me the migration steps.
```

## Alternative Install Path With GitHub CLI

GitHub's docs also note that `gh skill` can discover, install, update, and publish agent skills. This repository is already a normal skill directory, so copying the folder is the simplest path right now.

## Example Prompts

Use any of these after installation:

```text
/compatibility-scan .
```

```text
Find all legacy Copilot metrics API calls in this repo and tell me which ones are already migrated.
```

```text
Scan our GitHub Actions workflows for deprecated Copilot metrics endpoints and show the replacement report endpoints.
```

```text
Explain how to migrate this old /orgs/{org}/copilot/metrics parser to the new downloaded report schema.
```

## Notes And Limits

- The current metadata-backed replacement surface is the usage metrics report API under `/copilot/metrics/reports/...`.
- Team-scoped legacy metrics endpoints do not have a direct current report endpoint in the provided metadata.
- The Direct Data Access API does not have a direct replacement.
- Some legacy metrics only have near matches in the new downloaded report schemas, not exact field-for-field replacements.

## Development

The repository includes eval cases in [evals/evals.json](evals/evals.json) and source reference material under [metadata](metadata).