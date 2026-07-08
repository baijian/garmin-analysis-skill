---
name: garmin-analysis-skill
slug: garmin-analysis-skill
displayName: 佳明数据同步与分析
version: 0.0.2
summary: 基于 cycling-health CLI，按登录状态选区查询分析佳明睡眠与骑行数据（未登录提醒登录、单区登录用该区、双区登录且未指定时默认 CN），并将国服活动同步到国际服（需 CN 与 Global 均登录）。
license: MIT
description: Garmin activity sync and wellness/cycling analysis workflows using the public cycling-health CLI. Data queries (sleep, recovery, cycling) select the region by login state—prompt login when none is logged in, use the logged-in region when only one is available, and default to cn when both are logged in and the user did not specify a region; CN-to-Global sync requires both cn and global to be logged in. Use when an assistant needs to help users install or verify cycling-health, sync Garmin China activities to Garmin Global, check Garmin login/sync status, query sleep/recovery/cycling data, analyze cycling activities, inspect local Garmin FIT/GPX files, or assess cycling ability with training metrics such as VO2 max, FTP, endurance score, hill score, weight, and W/kg.
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
- Region selection for data queries follows login state: if neither `cn` nor `global` is logged in, prompt the user to log in; if only one is logged in, use that region; if both are logged in and the user did not specify a region, default to `cn`. When the user explicitly names a region, use it (it must be logged in). Pass the selected region with `--region <cn|global>`; omitting the flag defaults to `cn`. CN-to-Global sync requires both `cn` and `global` to be logged in.

Read `references/cli-workflows.md` when you need exact install guidance, commands, JSON fields, or failure handling.

## Workflow

1. Confirm `cycling-health` is available:
   - Run `cycling-health version` when command execution is available.
   - If unavailable, direct the user to install from `https://github.com/baijian/cycling-health`.

2. Check Garmin login state and select the query region:
   - Run `cycling-health garmin status --output json` to see which of `cn` and `global` are logged in.
   - If neither is logged in, prompt the user to run `garmin login --region <cn|global>` before querying data.
   - If only one region is logged in, use that region for data queries.
   - If both are logged in and the user did not specify a region, default to `cn`.
   - If the user explicitly specified a region, use it (it must be logged in).
   - Pass the selected region to data commands via `--region <cn|global>`; omitting it defaults to `cn`.
   - For CN-to-Global sync, both `cn` and `global` must be logged in.

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
1. Confirm both `cn` and `global` are logged in; sync cannot run otherwise.
2. Inspect sync state.
3. Run incremental sync unless the user explicitly requests a full historical sync.
4. Report retried, synced, failed, and last activity time.

Do not run a full historical sync without making the scope clear.

### Sleep Query And Analysis

Use when the user asks about sleep quality, sleep score, sleep duration, sleep stages, overnight HRV, resting HR, stress, body battery, recovery, or recent sleep trends.

Data source: select the region by login state per Principles (no login → prompt login; one login → that region; both logins and unspecified → `cn`). Pass it with `--region <cn|global>`.

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

Data source: select the region by login state per Principles (no login → prompt login; one login → that region; both logins and unspecified → `cn`). Pass it with `--region <cn|global>` on commands that accept it (for example `activity list --region cn`).

Default approach:
1. List cycling activities for the requested range. For "last N rides", use `--max-activities N` and widen `--days` if the default range does not include enough rides.
2. Fetch activity details for selected rides, using raw details when ability metrics, zones, training effects, or sensor fields may be hidden in the Garmin payload.
3. Query training/performance metrics and body metrics when assessing ability, especially VO2 max, FTP, endurance score, hill score, training status/readiness, weight, and W/kg.
4. Export and analyze FIT/GPX files when detailed time-series, per-distance samples, HR drift, pacing, power stability, or local file inspection is needed.
5. Summarize volume, intensity, terrain, pacing, technique signals, ability indicators, missing sensors, and anomalies.

Do not conclude that HR zones, VO2 max, FTP, weight, W/kg, or point-level data are unavailable until activity details/raw payloads, training metrics, body metrics, and FIT/GPX export analysis have been checked where relevant.
