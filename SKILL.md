---
name: garmin-analysis-skill
slug: garmin-analysis-skill
displayName: 骑行健康数据同步和分析(Garmin/Intervals.icu)
version: 0.0.5
summary: 基于 cycling-health CLI，先自升级，再同步 Garmin CN 健康数据到 Intervals.icu，并分层分析佳明睡眠恢复、单次骑行、长期骑行能力及 Intervals.icu 骑行健康度、功率、统计与比较。
license: MIT
description: Garmin and Intervals.icu health synchronization plus structured sleep, recovery, ride, power, fitness, statistics, and cycling-comparison workflows using the public cycling-health CLI. Use when an assistant needs to sync Garmin China wellness data to Intervals.icu; inspect Garmin or Intervals authentication; analyze Garmin sleep, readiness, activities, FTP, VO2 max, endurance, hill score, zones, splits, or FIT/GPX data; query Intervals.icu wellness, cycling activities, intervals, streams, best efforts, power curves, MMP models, fitness, fatigue, form, eFTP, or aggregate statistics; compare rides, power periods, or training blocks; sync Garmin China activities to Garmin Global; or report cycling-health issues.
---

# Garmin Analysis Skill

Use `cycling-health` as the source of truth for Garmin and Intervals.icu data. The public CLI is available from `https://github.com/baijian/cycling-health`.

## Principles

- Be environment-neutral. Do not assume a home directory, repository checkout, shell profile, or operating system.
- Prefer `cycling-health` from `PATH`. If it is missing, guide installation from the public repository.
- When command execution is available, run `cycling-health upgrade --output json` once before status, synchronization, or analysis commands.
- Prefer `--output json` for collection and analysis.
- Treat Garmin credentials, Intervals.icu API keys, OAuth tokens, passwords, MFA codes, and exported activity files as sensitive local data. Never ask the user to paste secrets into chat.
- Analyze only returned fields. Do not invent HRV, power, zones, fitness, fatigue, form, FTP, W/kg, or causal relationships.
- Collect in stages. Start with summary/list/statistics calls, then request intervals, streams, curves, raw payloads, or exported files only when the conclusion needs them.
- Separate current recovery from durable ability. A poor night or negative form can affect today's decision without proving fitness loss.
- Preserve genuine CLI warnings and unresolved gaps. Expected no-data responses are already normalized by the CLI and do not need to be restated as failures.
- Keep sources distinct. Garmin is the primary source for device-specific sleep, recovery, training effect, and raw activity detail. Intervals.icu is the primary source for consolidated activity history, server-calculated fitness/fatigue/form, power curves/models, statistics, and cross-activity comparison.
- Do not upload Garmin activities directly to Intervals.icu. Preserve the existing Garmin CN to Garmin Global activity sync and the user's Garmin Global to Intervals.icu automatic connection.
- Treat writes conservatively. Preview Garmin Wellness synchronization first; use `--confirm` only after the user has clearly authorized a live sync, and never add `--overwrite` without explicit approval after reviewing conflicts.

Garmin query-region rules:

- If neither Garmin region is logged in, prompt for login.
- If one region is logged in, use it for Garmin queries.
- If both are logged in and the user did not specify a region, default Garmin queries to `cn`.
- Pass the selected region with `--region cn|global`.
- Garmin CN to Global activity sync requires both regions.
- Garmin to Intervals.icu Wellness sync defaults to `--source garmin-cn`. Use `garmin-global` only when explicitly requested because Intervals.icu can already ingest Garmin Global data through its native connection.

Read the references as needed:

- Read `references/cli-workflows.md` for installation, Garmin authentication, CN-to-Global activity sync, sleep/recovery queries, Garmin ride analysis, file analysis, and Garmin failure handling.
- Read `references/intervals-workflows.md` for Intervals.icu authentication, Garmin CN Wellness sync, Wellness and training-health analysis, activities, power, statistics, comparisons, output fields, and Intervals-specific failure handling.

## Workflow

1. Verify and upgrade the CLI:
   - Run `cycling-health version`.
   - Run `cycling-health upgrade --output json` once per invocation.
   - If upgrade fails, preserve the error and ask before continuing with the installed version unless the user already authorized that fallback.
   - For an Intervals.icu task, verify `cycling-health intervals --help` and the requested subcommand's `--help` after upgrade. If missing, report that the installed public CLI is too old instead of improvising another client.

2. Check only the authentication needed for the task:
   - Garmin query or CN-to-Global activity sync: run `cycling-health garmin status --output json` and apply the region rules above.
   - Intervals.icu query: run `cycling-health intervals athlete --output json`.
   - Garmin Wellness to Intervals.icu sync: verify Intervals.icu with `intervals athlete` and Garmin CN with `garmin status --region cn`.

3. Route the task:
   - Garmin CN activity sync: inspect sync status, then run incremental sync by default.
   - Garmin Wellness sync: preview Garmin CN changes, inspect conflicts/warnings, then confirm only when authorized.
   - Garmin sleep/recovery: use wake-date-aware sleep and summary queries, then add selected health metrics.
   - Garmin single ride or device-specific analysis: start with Garmin activity details and add extended/raw/file data only as needed.
   - Intervals.icu riding health, long-term fitness, power, statistics, or comparison: start with Wellness, activity list, power model/curves, and aggregate statistics appropriate to the question.

4. Report results:
   - Lead with the conclusion.
   - State date ranges and data source for each conclusion.
   - Separate observed values, interpretation, confidence/gaps, and next actions.
   - Use compact units such as km, h:min, bpm, W, W/kg, rpm, m, CTL, ATL, and percent.
   - For writes, report previewed changes, conflicts, uploaded records, verified records, and warnings.

## Task Guidance

### Sync Garmin China Activities To Garmin Global

Use when the user asks to mirror or backfill Garmin China activities into Garmin Global.

1. Confirm both regions are logged in.
2. Inspect `garmin syncstatus`.
3. Run `garmin sync --new-only` unless the user explicitly requests historical backfill.
4. Report retried, synced, failed, and last activity time.

Do not treat this as a direct Intervals.icu upload. The user's Garmin Global connection handles subsequent activity ingestion into Intervals.icu.

### Sync Garmin Health To Intervals.icu

Use when the user asks to sync Garmin health, sleep, recovery, or Wellness data to Intervals.icu.

1. Default to Garmin CN and a seven-day range unless the user provides dates.
2. Verify Garmin CN and Intervals.icu authentication.
3. Run `intervals wellness sync --source garmin-cn` without `--confirm` first.
4. Inspect field-level `changes`, `conflicts`, `warnings`, and record actions.
5. If there are no changes, stop; do not issue a confirmed no-op merely for appearance.
6. If the user already clearly requested a live sync, or confirms after seeing the preview, rerun the same range with `--confirm`.
7. Skip conflicting existing values by default. Use `--overwrite --confirm` only after the user explicitly approves replacing them.
8. Report `changedRecords`, `changedFields`, `conflictFields`, `uploadedRecords`, and `verifiedRecords`.

The automatic mapping currently covers sleep duration, sleep score, average sleeping heart rate, overnight HRV, resting heart rate, daily average SpO2, average sleeping respiration, and steps. Missing Garmin values are omitted. Do not reinterpret Garmin stress, Body Battery, training readiness, calories burned, or training load as Intervals.icu Wellness fields.

### Sleep Query And Analysis

Use when the user asks about sleep duration, stages, score, overnight HRV, resting heart rate, stress, Body Battery, readiness, or recent sleep trends.

1. Resolve a night by wake date and include the adjacent prior date in sleep lookup.
2. Start with sleep list plus wake-date daily summary.
3. Default recent trends to seven days; use 14-30 days for an explicit baseline/trend question.
4. Add only the health metrics needed for the question.
5. Check adjacent-date and summary fallbacks before declaring sleep details unavailable.
6. Compare with the user's available baseline and avoid medical diagnosis.

### Garmin Cycling Query And Analysis

Use Garmin when the question depends on Garmin training effect/status/readiness, device zones, detailed laps/splits, gear/weather, or local FIT/GPX records.

1. Classify the request as single ride, durable ability/progression, or today's training decision.
2. Start a single ride with `garmin activity get`; use extended/raw/file analysis only for missing detail.
3. For ability, default to 90 days and add direct cycling FTP, max metrics, endurance score, hill score, and body weight only when relevant.
4. Use activity progress only for requested axes.
5. Keep recovery/readiness separate from durable ability.
6. Account for terrain, weather, equipment, and sensor coverage; speed alone is not fitness.

### Intervals.icu Cycling Query And Analysis

Use Intervals.icu when the user asks about riding health/fitness, consolidated training history, eFTP, CTL/ATL/form, power curves/models, aggregate statistics, or comparisons.

1. Choose the smallest useful date range and identify `Ride` records when activity lists include multiple sports.
2. For daily health/recovery, query Wellness. For training health, combine aggregate fitness, fatigue, form, ramp rate, load, and eFTP from statistics; do not conflate these with subjective Wellness values.
3. For one ride, identify it with list/search, fetch detail, then add intervals, streams, best efforts, or activity-analysis endpoints only as needed.
4. For power ability, start with the current MMP model and `42d,1y,all` curves; add per-activity duration values or power-versus-heart-rate data for progression or efficiency questions.
5. For statistics, use a recent range and, when comparison is requested, a preceding non-overlapping range of equal length.
6. Compare like with like: same duration, sport, units, and comparable terrain/workout type. Show absolute and percentage change only when both values are valid.
7. Use Intervals.icu server-calculated values as returned. Do not silently recompute or combine duplicate Garmin/Intervals activity records.
8. Correlate Wellness and performance cautiously; date alignment supports context, not causation.

## Feedback And Issues

For installation problems, Garmin or Intervals.icu data gaps, synchronization failures, or analysis issues, direct users to the Discord feedback channel:

```text
https://discord.gg/R6xPZc5Dg
```
