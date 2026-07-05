# Garmin CLI Workflows

Use this reference for public `cycling-health` installation guidance, Garmin commands, and analysis conventions.

## Install Or Locate The CLI

Public repository:

```text
https://github.com/baijian/cycling-health
```

Prefer an existing installation:

```bash
command -v cycling-health
cycling-health version
cycling-health doctor --output json
```

If `cycling-health` is missing:

1. Direct the user to `https://github.com/baijian/cycling-health`.
2. Prefer the repository's latest release/download instructions.
3. Ask the user to place the executable on `PATH` or provide its absolute path.
4. Verify with `cycling-health version`.

Do not assume a source checkout or machine-specific binary path.

## Auth And Status

Check status:

```bash
cycling-health garmin status --output json
cycling-health garmin status --region cn --output json
cycling-health garmin status --region global --output json
```

Interactive login:

```bash
cycling-health garmin login --region cn --email user@example.com
cycling-health garmin login --region global --email user@example.com
```

Notes:

- Garmin China data queries use region `cn`.
- Garmin Global login is required for China-to-Global activity sync.
- Password and MFA prompts are interactive. The user should enter them locally.
- Token files are local secrets; do not display or copy their contents.

## China To Global Activity Sync

Inspect sync state:

```bash
cycling-health garmin syncstatus --output json
```

Incremental sync:

```bash
cycling-health garmin sync --new-only --output json
```

Full historical sync:

```bash
cycling-health garmin sync --output json
```

Default to incremental sync. Use full sync only for first-time backfill or when the user explicitly asks for historical sync.

Report these fields when present:

- `retried`
- `synced`
- `failed`
- `lastActivityTime`
- failed record count from `syncstatus`
- state and failed-record paths only when useful for troubleshooting

## Sleep Data

Common queries:

```bash
cycling-health garmin sleep list --days 7 --output json
cycling-health garmin sleep list --start YYYY-MM-DD --end YYYY-MM-DD --output json
cycling-health garmin sleep list --start PREVIOUS_DATE --end WAKE_DATE --output json
cycling-health garmin summary get --date YYYY-MM-DD --output json
cycling-health garmin health get --days 7 --metrics hrv,rhr,stress,bb,respiration,spo2 --output json
cycling-health garmin health get --start PREVIOUS_DATE --end WAKE_DATE --metrics hrv,rhr,stress,bb,respiration,spo2 --output json
cycling-health garmin chart render --chart sleep --days 30 --output garmin-sleep.html
```

Date handling:

- Interpret "today's early morning sleep" and "last night" as sleep whose wake date is the requested date.
- For a single wake date, query `sleep list` with the exact date and also `PREVIOUS_DATE` through `WAKE_DATE`; Garmin sleep records can be keyed by either sleep start date or wake date.
- If the user asks for recent sleep without a specific date, start with `sleep list --days 7`, then add health metrics for the same range.
- If `data.records[]` is empty for the exact date, do not stop. Run the adjacent-date sleep window and `summary get` for both `WAKE_DATE` and `PREVIOUS_DATE`.

Sleep JSON shape:

- `data.records[]`: per-day sleep record
- `data.averages`: aggregate sleep averages
- `warnings[]`: endpoint or data gaps

Useful sleep fields:

- `date`
- `total_seconds`, `total_formatted`
- `deep_seconds`, `light_seconds`, `rem_seconds`, `awake_seconds`
- `sleep_score`
- `resting_heart_rate`
- `avg_overnight_hrv`, `hrv_last_night_avg`, `hrv_weekly_avg`, `hrv_status`
- `body_battery_high`, `body_battery_low`

Fallback sequence for empty sleep records:

1. Re-check a wider sleep window:

   ```bash
   cycling-health garmin sleep list --start PREVIOUS_DATE --end WAKE_DATE --output json
   cycling-health garmin sleep list --days 7 --output json
   ```

2. Check Garmin daily summaries for both possible record dates:

   ```bash
   cycling-health garmin summary get --date WAKE_DATE --output json
   cycling-health garmin summary get --date PREVIOUS_DATE --output json
   ```

3. Check recovery metrics across the same overnight window:

   ```bash
   cycling-health garmin health get --start PREVIOUS_DATE --end WAKE_DATE --metrics hrv,rhr,stress,bb,respiration,spo2 --output json
   ```

Only say that sleep-stage details are unavailable after these fallbacks return no sleep-duration, sleep-stage, or sleep-score fields. If health metrics such as HRV, respiration, SpO2, stress, or body battery are present, describe them as recovery context rather than a substitute for sleep-stage details.

Analysis checklist:

- Duration vs range average.
- Sleep score and trend.
- Deep, REM, light, and awake time when present.
- HRV status and last-night HRV vs weekly average.
- Resting HR and stress context.
- Body battery recharge/depletion.
- Missing days and warnings.

## Health And Recovery Context

Common queries:

```bash
cycling-health garmin health get --days 7 --metrics hrv,rhr,stress,bb --output json
cycling-health garmin health get --start YYYY-MM-DD --end YYYY-MM-DD --metrics hrv,rhr,stress,bb,respiration,spo2 --output json
cycling-health garmin health query --metric heart_rate --date YYYY-MM-DD --time HH:MM --output json
cycling-health garmin health query --metric stress --date YYYY-MM-DD --time HH:MM --output json
cycling-health garmin health query --metric body_battery --date YYYY-MM-DD --time HH:MM --output json
```

Use recovery metrics to explain sleep and cycling readiness only when data is present.

## Cycling Data

List recent cycling activities:

```bash
cycling-health garmin activity list --type cycling --days 30 --output json
cycling-health garmin activity list --type cycling --days 180 --max-activities 5 --output json
cycling-health garmin activity list --type cycling --start YYYY-MM-DD --end YYYY-MM-DD --max-activities 50 --output json
```

For "last N rides", start with `--max-activities N`. If fewer than N rides are returned, increase `--days` or ask for a wider date range before analyzing ability from an incomplete sample.

Fetch one activity:

```bash
cycling-health garmin activity get --activity-id ACTIVITY_ID --output json
cycling-health garmin activity get --activity-id ACTIVITY_ID --raw --output json
```

Training and body context for cycling ability:

```bash
cycling-health garmin training get --metric max_metrics --date YYYY-MM-DD --output json
cycling-health garmin training get --metric status --date YYYY-MM-DD --output json
cycling-health garmin training get --metric readiness --date YYYY-MM-DD --output json
cycling-health garmin training get --metric endurance_score --date YYYY-MM-DD --output json
cycling-health garmin training get --metric hill_score --date YYYY-MM-DD --output json
cycling-health garmin body get --metric weigh_ins --days 30 --output json
cycling-health garmin body get --metric composition --days 30 --output json
cycling-health garmin profile get --raw --output json
```

Use `training get --metric max_metrics` before saying VO2 max or FTP is unavailable. Use `body get --metric weigh_ins` and `body get --metric composition` before saying weight is unavailable. Use the latest reliable body weight for W/kg only when both weight and power or FTP are present; report the weight date if it is stale.

Export and analyze local files:

```bash
mkdir -p garmin-analysis
cycling-health garmin activity export --activity-id ACTIVITY_ID --format fit --output garmin-analysis
cycling-health garmin fit download --activity-id ACTIVITY_ID --path garmin-analysis/ACTIVITY.fit
cycling-health garmin activity analyze --file garmin-analysis/ACTIVITY.fit --output json
cycling-health garmin fit parse --file garmin-analysis/ACTIVITY.fit --targets 1,5,10,last --output json
cycling-health garmin activity query --file garmin-analysis/ACTIVITY.fit --distance 10000 --output json
cycling-health garmin activity dump --file garmin-analysis/ACTIVITY.fit --max-records 0 --output json
```

Activity list fields vary by Garmin payload. Common useful fields include:

- `activityId`
- `activityName`
- `startTimeLocal`
- `distance`
- `duration`, `movingDuration`
- `averageSpeed`, `maxSpeed`
- `averageHR`, `maxHR`
- `elevationGain`
- `averageBikeCadence`, `maxBikeCadence`
- `avgPower`, `maxPower`, `normalizedPower`
- `trainingEffect`, `aerobicTrainingEffect`, `anaerobicTrainingEffect`
- `hrTimeInZone`, `powerTimeInZone`, `timeInZones`

Training and body fields vary by Garmin payload. Search returned JSON for these concepts before reporting them missing:

- VO2 max: `vo2Max`, `generic.value`, `cyclingVo2Max`, or VO2-related keys in `max_metrics`
- FTP: `ftp`, `functionalThresholdPower`, `cyclingFTP`, or power threshold keys in `max_metrics`
- Endurance and climbing: `endurance_score`, `hill_score`, `score`, `classification`, or trend fields
- Weight: `weight`, `weightKg`, `bodyWeight`, or latest weigh-in/composition entry
- Zones: HR/power zone arrays, time-in-zone summaries, or zone settings in raw activity/profile payloads

Local file analysis fields:

- `durationSec`
- `distanceM`
- `heartRate`
- `elevation`
- `speed`
- `cadence`
- `power`
- `kmSamples` from `garmin fit parse`
- raw records from `activity dump` when drift, stability, or point-level pacing needs calculation

Cycling analysis checklist:

- Volume: ride count, total distance, total duration, weekly trend.
- Intensity: average/max HR, power if present, speed distribution.
- Terrain: elevation gain and climbing impact.
- Technique: cadence consistency if present.
- Pacing: first half vs second half, per-distance sample changes.
- Ability: VO2 max, FTP, W/kg, endurance score, hill score, training status/readiness, and ride-derived power/HR signals when present.
- Data quality: missing HR, cadence, power, GPS, or sparse records.

Fallback sequence for cycling ability gaps:

1. For missing VO2 max or FTP, run `training get --metric max_metrics` for the analysis date. If absent, inspect selected activity raw details for VO2, FTP, threshold, or performance-condition fields.
2. For missing weight or W/kg, run body `weigh_ins` and `composition`; optionally inspect `profile get --raw`. Do not calculate W/kg without both weight and a relevant power value.
3. For missing HR zones or power zones, inspect `activity get --raw`, `profile get --raw`, and FIT analysis output for time-in-zone or zone arrays. If still absent, analyze HR/power using observed averages, maxima, and distributions instead of inventing zone boundaries.
4. For missing per-km or point-level data, export or download FIT, then run `activity analyze`, `fit parse`, and, when needed, `activity dump --max-records 0` on selected activities.
5. Only after these checks should the answer list a data gap as unavailable. Phrase it as "not returned by the checked CLI calls" and include the commands checked.

## Failure Handling

- If `cycling-health` is unavailable, guide installation from `https://github.com/baijian/cycling-health`.
- If status says not logged in, run the appropriate `garmin login` command.
- If DNS or proxy errors appear, ask the user whether a proxy/VPN is expected, then retry with the correct network setup.
- If a CN endpoint returns warnings but the main payload exists, report available data and list warnings.
- If `profile get` fails for CN, do not use that endpoint as token validation; prefer `garmin status` and real data queries.
- If `sleep list` returns `records: []` for a single date, treat it as inconclusive until adjacent-date sleep and summary fallback queries are checked.
- If a command returns JSON with `warnings`, include those warnings in the final answer.
