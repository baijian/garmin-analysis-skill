---
name: garmin-analysis-skill
slug: garmin-analysis-skill
displayName: 佳明数据同步与分析
version: 0.0.4
summary: 基于 cycling-health CLI，先自升级，再分层分析佳明睡眠恢复、单次骑行与长期骑行能力，并将国服活动同步到国际服。
license: MIT
description: Garmin activity sync and structured sleep, recovery, ride, and cycling-ability analysis workflows using the public cycling-health CLI, including CLI install/verify/self-upgrade from GitHub and Discord feedback routing. Data queries select the region from login state; CN-to-Global sync requires both regions to be logged in. Use when an assistant needs to analyze last night's sleep or longer sleep trends, assess recovery/readiness, review a ride, evaluate cycling fitness and progression using activity history, training load, VO2 max, cycling FTP, W/kg, endurance score, hill score, zones, splits, or FIT/GPX data, inspect Garmin status, sync Garmin China activities to Garmin Global, or report cycling-health issues.
---

# Garmin Analysis Skill

Use `cycling-health` as the command-line interface for Garmin data. The CLI is public and available from `https://github.com/baijian/cycling-health`.

## Principles

- Be environment-neutral. Do not assume a specific home directory, machine path, repository checkout, shell profile, or operating system.
- Prefer `cycling-health` from `PATH`. If it is missing, guide the user to install it from the public repository before continuing.
- When command execution is available and `cycling-health` exists, run `cycling-health upgrade --output json` once before Garmin status, sync, or analysis commands.
- Prefer JSON output for data collection and analysis. Use text output only when the user explicitly wants CLI-style output.
- Treat Garmin credentials, passwords, MFA codes, OAuth tokens, and exported activity files as sensitive local data.
- Let users type passwords and MFA codes into their terminal or trusted local prompt. Do not ask them to paste secrets into chat.
- Analyze only fields that exist in the CLI output. Do not invent HRV, sleep score, cadence, power, zones, or training conclusions.
- Preserve CLI warnings and data gaps in the answer.
- Collect data in stages. Start with summary/list commands, then request raw payloads, extended splits, or exported files only when the requested conclusion needs them.
- Separate current readiness from durable cycling ability. A poor night can lower today's readiness without proving fitness loss; one strong ride does not establish a long-term ability gain.
- Region selection for data queries follows login state: if neither `cn` nor `global` is logged in, prompt the user to log in; if only one is logged in, use that region; if both are logged in and the user did not specify a region, default to `cn`. When the user explicitly names a region, use it (it must be logged in). Pass the selected region with `--region <cn|global>`; omitting the flag defaults to `cn`. CN-to-Global sync requires both `cn` and `global` to be logged in.

Read `references/cli-workflows.md` when you need exact install guidance, commands, JSON fields, or failure handling.

## Workflow

1. Confirm `cycling-health` is available:
   - Run `cycling-health version` when command execution is available.
   - If unavailable, direct the user to install from `https://github.com/baijian/cycling-health`.

2. Upgrade the CLI before using Garmin commands:
   - Run `cycling-health upgrade --output json` once per skill invocation when command execution is available.
   - If the upgrade succeeds or reports `up-to-date`, continue normally.
   - If the upgrade fails because of network, permission, checksum, or release-asset issues, keep the error visible in the final answer and ask before continuing with the installed version unless the user already told you to proceed.

3. Check Garmin login state and select the query region:
   - Run `cycling-health garmin status --output json` to see which of `cn` and `global` are logged in.
   - If neither is logged in, prompt the user to run `garmin login --region <cn|global>` before querying data.
   - If only one region is logged in, use that region for data queries.
   - If both are logged in and the user did not specify a region, default to `cn`.
   - If the user explicitly specified a region, use it (it must be logged in).
   - Pass the selected region to data commands via `--region <cn|global>`; omitting it defaults to `cn`.
   - For CN-to-Global sync, both `cn` and `global` must be logged in.

4. Select the task workflow:
   - Sync activities: inspect sync status, then run incremental sync by default.
   - Sleep analysis: choose a single-night snapshot or a multi-night trend, then add only the recovery context needed.
   - Cycling analysis: distinguish a single-ride review, a long-term ability assessment, and today's training readiness.

5. Report results:
   - Start with the conclusion.
   - Include date range, commands used, and any warnings.
   - Use compact units such as km, h:min, bpm, W, rpm, m, and percent.
   - Separate observed facts from interpretation and suggested next actions.

## Feedback And Issues

When users want to report bugs, installation problems, Garmin data gaps, or analysis issues, invite them to join the Discord server and use its feedback channel:

```text
https://discord.gg/R6xPZc5Dg
```

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
2. For a single-night snapshot, query the previous-date-to-wake-date sleep window and the wake-date daily summary. The summary already combines sleep, training readiness, training status/load, and weekly intensity when Garmin returns them.
3. For recent sleep with no range, default to 7 days. Use 14-30 days only for an explicit trend or baseline question.
4. Add `health get` for HRV, resting HR, stress, body battery, respiration, or SpO2 when recovery context is needed. Add `bb_events` only when explaining which activities or events charged or drained body battery.
5. If sleep records are empty, query the exact wake date, the adjacent-date window, and daily summaries for both dates before declaring sleep-stage data missing.
6. Compare the target night with the user's own available baseline: duration, timing consistency, score, stages, sleeping HR, HRV, respiration, stress, and body-battery recharge. Do not substitute population targets for missing personal context.
7. Summarize observed sleep, recovery interpretation, confidence/data gaps, and practical next actions. Render a sleep chart only when the user asks for a visual or a longer trend benefits from one.

Do not conclude that sleep details are unavailable until the sleep list, adjacent-date sleep window, and daily summary fallback have all been checked.

Avoid medical diagnosis. Present wellness observations only.

### Cycling Query And Analysis

Use when the user asks about recent rides, a specific cycling activity, cycling ability, training load from rides, heart rate, speed, elevation, cadence, power, pacing, VO2 max, FTP, endurance, climbing ability, W/kg, or local FIT/GPX files.

Data source: select the region by login state per Principles (no login → prompt login; one login → that region; both logins and unspecified → `cn`). Pass it with `--region <cn|global>` on commands that accept it (for example `activity list --region cn`).

Default approach:
1. Classify the request:
   - Single-ride review: list enough activities to identify the ride, then fetch that activity's details.
   - Cycling ability or progression: default to the last 90 days, using recent rides plus training/performance and body metrics.
   - Today's training decision: combine recent load with daily summary/training readiness and recovery; do not treat this as a durable fitness score.
2. For a single ride, use `activity get` first because it already collects summary, laps, HR/power zones, weather, and gear. Use `--raw` only when the simplified result omits a needed field.
3. For ability assessment, collect cycling activity history, then query training status/load, `max_metrics`, direct `cycling_ftp`, endurance score, and hill score. Query weight/composition only when body context or W/kg is relevant.
4. Use `activity progress` for explicit progression questions. Query only the requested axes (distance, duration/moving duration, or elevation gain) instead of calling every metric by default.
5. Use `activity extended` when typed splits or split summaries are needed but normal activity details are insufficient. Export/analyze FIT or GPX only for point-level pacing, HR drift/decoupling, power stability, cadence consistency, or local-file inspection.
6. Use personal records only when the user asks about peak performance or historical bests. Treat workouts, training plans, events, courses, and gear as planning/context data, not evidence of current ability.
7. Assess available evidence across consistency/volume, aerobic durability, threshold power, climbing, intensity distribution, pacing, and technique. Account for terrain, weather, equipment, and sensor coverage; speed alone is not a fitness measure.
8. Report observed metrics, interpretation, confidence/data gaps, and next actions. Do not invent a composite score or zone boundaries unless the user explicitly asks and the inputs are present.

Do not conclude that HR zones, VO2 max, FTP, weight, W/kg, splits, or point-level data are unavailable until the relevant staged fallbacks in `references/cli-workflows.md` have been checked.
