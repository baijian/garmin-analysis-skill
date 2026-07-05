---
name: garmin-analysis-skill
slug: garmin-analysis-skill
displayName: 佳明数据同步与分析
version: 0.0.1
summary: 基于 cycling-health CLI，同步佳明国服活动到佳明国际服，并查询和分析佳明睡眠、骑行数据。
license: MIT
description: Garmin activity sync and wellness/cycling analysis workflows using the public cycling-health CLI. Use when an assistant needs to help users install or verify cycling-health, sync Garmin China activities to Garmin Global, check Garmin login/sync status, query Garmin China sleep and recovery data, analyze cycling activities, inspect local Garmin FIT/GPX files, or assess cycling ability with training metrics such as VO2 max, FTP, endurance score, hill score, weight, and W/kg.
---

# Garmin Analysis Skill

Use `cycling-health` as the command-line interface for Garmin data. The CLI is public and available from `https://github.com/baijian/cycling-health`.

## Principles

- Be environment-neutral. Do not assume a specific home directory, machine path, repository checkout, shell profile, or operating system.
- Prefer `cycling-health` from `PATH`. If it is missing, guide the user to install it from the public repository before continuing.
- Prefer JSON output for data collection and analysis. Use text output only when the user explicitly wants CLI-style output.
- Treat Garmin credentials, passwords, MFA codes, OAuth tokens, and exported activity files as sensitive local data.
- Let users type passwords and MFA codes into their terminal or trusted local prompt. Do not ask them to paste secrets into chat.
- Analyze only fields that exist in the CLI output. Do not invent HRV, sleep score, cadence, power, zones, or training conclusions.
- Preserve CLI warnings and data gaps in the answer.

Read `references/cli-workflows.md` when you need exact install guidance, commands, JSON fields, or failure handling.

## Workflow

1. Confirm `cycling-health` is available:
   - Run `cycling-health version` when command execution is available.
   - If unavailable, direct the user to install from `https://github.com/baijian/cycling-health`.

2. Check Garmin login state:
   - Run `cycling-health garmin status --output json`.
   - CN data queries require the `cn` region to be logged in.
   - CN-to-Global sync requires both `cn` and `global` regions to be logged in.

3. Select the task workflow:
   - Sync activities: inspect sync status, then run incremental sync by default.
   - Sleep analysis: query sleep records and recovery metrics for the requested date range.
   - Cycling analysis: list cycling activities, fetch selected activity details, and export/analyze FIT or GPX when point-level analysis is needed.

4. Report results:
   - Start with the conclusion.
   - Include date range, commands used, and any warnings.
   - Use compact units such as km, h:min, bpm, W, rpm, m, and percent.
   - Separate observed facts from interpretation and suggested next actions.

## Task Guidance

### Sync Garmin China Activities To Garmin Global

Use when the user asks to sync, migrate, mirror, upload, backfill, or keep Garmin China activities aligned with Garmin Global.

Default approach:
1. Check login status for both regions.
2. Inspect sync state.
3. Run incremental sync unless the user explicitly requests a full historical sync.
4. Report retried, synced, failed, and last activity time.

Do not run a full historical sync without making the scope clear.

### Sleep Query And Analysis

Use when the user asks about sleep quality, sleep score, sleep duration, sleep stages, overnight HRV, resting HR, stress, body battery, recovery, or recent sleep trends.

Default approach:
1. Resolve relative sleep dates by wake date. For "last night" or "today's early morning sleep", treat the requested day as the wake date and include the previous calendar day in sleep lookups.
2. Query sleep records for the requested range, defaulting to 7 days when unspecified. For a single wake date, query both the exact date and a previous-day-to-wake-date window before declaring records missing.
3. If sleep records are empty, run Garmin daily summary queries for the wake date and previous day, then use health metrics only as recovery context.
4. Add HRV, resting HR, stress, body battery, respiration, or SpO2 queries when recovery context is needed.
5. Summarize duration, quality, trend, recovery signals, missing data, and practical next actions.

Do not conclude that sleep details are unavailable until the sleep list, adjacent-date sleep window, and daily summary fallback have all been checked.

Avoid medical diagnosis. Present wellness observations only.

### Cycling Query And Analysis

Use when the user asks about recent rides, a specific cycling activity, cycling ability, training load from rides, heart rate, speed, elevation, cadence, power, pacing, VO2 max, FTP, endurance, climbing ability, W/kg, or local FIT/GPX files.

Default approach:
1. List cycling activities for the requested range. For "last N rides", use `--max-activities N` and widen `--days` if the default range does not include enough rides.
2. Fetch activity details for selected rides, using raw details when ability metrics, zones, training effects, or sensor fields may be hidden in the Garmin payload.
3. Query training/performance metrics and body metrics when assessing ability, especially VO2 max, FTP, endurance score, hill score, training status/readiness, weight, and W/kg.
4. Export and analyze FIT/GPX files when detailed time-series, per-distance samples, HR drift, pacing, power stability, or local file inspection is needed.
5. Summarize volume, intensity, terrain, pacing, technique signals, ability indicators, missing sensors, and anomalies.

Do not conclude that HR zones, VO2 max, FTP, weight, W/kg, or point-level data are unavailable until activity details/raw payloads, training metrics, body metrics, and FIT/GPX export analysis have been checked where relevant.
