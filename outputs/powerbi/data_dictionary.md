# Power BI export data dictionary

Source: `ball_by_ball_data.csv` (288,226 delivery rows).

## Grain and scope

- `ipl_deliveries_clean.csv`: one row per original delivery; source rows are not dropped.
- `ipl_innings_summary.csv`: one row per season, match, innings, and super-over flag.
- Batting and bowling tables: one row per season and batter/bowler, regulation innings only.
- Team table: one row per season and numeric batting-team ID, regulation innings only.
- Team names remain unmapped because no authoritative mapping was validated.
- Delivery-level `total_runs` includes extras; batting `batter_runs_scored` excludes extras; innings and team totals include extras.
- Batting, bowling, and team-season tables exclude super overs. Delivery and innings-summary exports retain super overs with `is_super_over`.
- Power BI measures should exclude super overs by default when comparing delivery data with regulation batting, bowling, and team-season tables.
- Match results, dates, venues, tosses, and title counts are omitted because no validated metadata source was available.

## Definitions

- `is_legal_delivery`: not a wide and not a no-ball.
- `is_dot_ball`: legal delivery and zero total runs.
- `is_batter_ball_faced`: all deliveries except wides; no-balls count as faced.
- `bowler_wickets`: bowled, caught, caught and bowled, lbw, stumped, or hit wicket.
- `is_wicket_lost`: source wicket except retired hurt.
- `bowler_runs_conceded`: total runs less byes, leg-byes, and penalty runs.
- `run_rate` / `economy_rate`: runs per six legal balls; undefined denominators are null.
- Strike rate: batter runs / balls faced × 100. Bowling average: bowler runs / credited wickets.
- Bowling strike rate: legal balls / credited wickets. Dot-ball percentage: dot balls / legal balls × 100.
- Source over and ball indices are zero-based. Regulation phases: 0–5 powerplay, 6–14 middle, 15–19 death.
- `overs_faced` and `overs_bowled` are decimal legal balls divided by six, not cricket over notation.

## Example DAX measures

The following delivery measures exclude super overs. Relate `dim_batter` to the delivery `batter` column and `dim_bowler` to `bowler` as needed.

```DAX
Regulation Runs = CALCULATE(SUM('ipl_deliveries_clean'[total_runs]), KEEPFILTERS('ipl_deliveries_clean'[is_super_over] = FALSE()))
Regulation Wickets Lost = CALCULATE(COUNTROWS(FILTER('ipl_deliveries_clean', 'ipl_deliveries_clean'[is_wicket_lost] = TRUE())), KEEPFILTERS('ipl_deliveries_clean'[is_super_over] = FALSE()))
Bowler-Creditable Wickets = CALCULATE(COUNTROWS(FILTER('ipl_deliveries_clean', 'ipl_deliveries_clean'[bowler_wickets] = TRUE())), KEEPFILTERS('ipl_deliveries_clean'[is_super_over] = FALSE()))
Legal Balls = CALCULATE(COUNTROWS(FILTER('ipl_deliveries_clean', 'ipl_deliveries_clean'[is_legal_delivery] = TRUE())), KEEPFILTERS('ipl_deliveries_clean'[is_super_over] = FALSE()))
Run Rate = DIVIDE([Regulation Runs] * 6, [Legal Balls])
Batter Strike Rate = DIVIDE(CALCULATE(SUM('ipl_deliveries_clean'[batter_runs]), KEEPFILTERS('ipl_deliveries_clean'[is_super_over] = FALSE())) * 100, CALCULATE(COUNTROWS(FILTER('ipl_deliveries_clean', 'ipl_deliveries_clean'[is_batter_ball_faced] = TRUE())), KEEPFILTERS('ipl_deliveries_clean'[is_super_over] = FALSE())))
Bowling Economy = DIVIDE(CALCULATE(SUM('ipl_deliveries_clean'[bowler_runs_conceded]), KEEPFILTERS('ipl_deliveries_clean'[is_super_over] = FALSE())) * 6, [Legal Balls])
Dot-Ball Percentage = DIVIDE(CALCULATE(COUNTROWS(FILTER('ipl_deliveries_clean', 'ipl_deliveries_clean'[is_dot_ball] = TRUE())), KEEPFILTERS('ipl_deliveries_clean'[is_super_over] = FALSE())) * 100, [Legal Balls])
```

Use `is_wicket_lost` for innings wickets and `bowler_wickets` for credited bowling wickets; the generic `is_wicket` flag includes events that are not bowler-creditable.

## Exported files and columns

### `ipl_deliveries_clean.csv`

Grain: one row per source delivery.

| Column | Data type | Definition |
|---|---|---|
| `season_id` | `int64` | Season identifier as supplied in the source. |
| `match_id` | `string` | Match identifier; retained as text, not a numeric measure. |
| `batter` | `string` | Source batter name after whitespace trimming only. |
| `bowler` | `string` | Source bowler name after whitespace trimming only. |
| `non_striker` | `string` | Source non-striker name after whitespace trimming only. |
| `team_batting` | `int64` | Source batting-team ID; canonical name is unverified. |
| `team_bowling` | `int64` | Source bowling-team ID; canonical name is unverified. |
| `over_number` | `int64` | Zero-based source over index. |
| `ball_number` | `int64` | Zero-based source delivery index within the over. |
| `batter_runs` | `int64` | Runs credited to the batter on this delivery. |
| `extras` | `int64` | Total extra runs on this delivery. |
| `total_runs` | `int64` | Delivery total in the fact table; sum of batter runs in batting stats; actual innings/team runs in summaries. |
| `batsman_type` | `string` | Optional source batting-style attribute; missing values preserved. |
| `bowler_type` | `string` | Optional source bowling-style attribute; missing values preserved. |
| `player_out` | `string` | Dismissed player supplied by the source; null when absent. |
| `fielders_involved` | `string` | Fielder information supplied by the source; nulls preserved. |
| `is_wicket` | `boolean` | Source Boolean wicket-event flag. |
| `is_wide_ball` | `boolean` | Source Boolean wide-delivery flag. |
| `is_no_ball` | `boolean` | Source Boolean no-ball flag. |
| `is_leg_bye` | `boolean` | Source Boolean leg-bye flag. |
| `is_bye` | `boolean` | Source Boolean bye flag. |
| `is_penalty` | `boolean` | Source Boolean penalty-runs flag. |
| `wide_ball_runs` | `int64` | Source runs attributed to wides. |
| `no_ball_runs` | `int64` | Source runs attributed to no-balls. |
| `leg_bye_runs` | `int64` | Source runs attributed to leg-byes. |
| `bye_runs` | `int64` | Source runs attributed to byes. |
| `penalty_runs` | `int64` | Source penalty runs. |
| `wicket_kind` | `string` | Source dismissal classification; null when no dismissal is recorded. |
| `is_super_over` | `boolean` | Source Boolean distinguishing super-over deliveries. |
| `innings` | `int64` | Source innings identifier in delivery/innings tables; batter-match-innings appearance count in batting stats. |
| `is_legal_delivery` | `boolean` | True unless the delivery is a wide or no-ball. |
| `is_dot_ball` | `boolean` | True for a legal delivery with zero total runs. |
| `is_boundary_four` | `bool` | True when batter_runs equals four. |
| `is_six` | `bool` | True when batter_runs equals six. |
| `bowler_wickets` | `boolean` | True for bowled, caught, caught and bowled, lbw, stumped, or hit wicket. |
| `is_wicket_lost` | `boolean` | Source wicket excluding retired hurt; retired out remains included. |
| `is_batter_ball_faced` | `boolean` | False for wides; true for all other deliveries, including no-balls. |
| `bowler_runs_conceded` | `int64` | total_runs less byes, leg-byes, and penalty runs. |
| `innings_phase` | `object` | Powerplay (0–5), middle (6–14), death (15–19), super over, or outside defined phases. |
| `cumulative_innings_runs` | `int64` | Running total_runs in season, match, innings, and super-over group. |
| `cumulative_innings_wickets` | `Int64` | Running is_wicket_lost count in the same innings group. |
| `boundary_count` | `int64` | Number of batter-run fours plus sixes on the delivery/aggregate. |
| `boundary_runs` | `int64` | Four runs per batter-run four plus six runs per batter-run six. |

Rows: 288,226.

### `ipl_innings_summary.csv`

Grain: one row per season, match, innings, and super-over flag; super overs retained.

| Column | Data type | Definition |
|---|---|---|
| `season_id` | `int64` | Season identifier as supplied in the source. |
| `match_id` | `string` | Match identifier; retained as text, not a numeric measure. |
| `innings` | `int64` | Source innings identifier in delivery/innings tables; batter-match-innings appearance count in batting stats. |
| `is_super_over` | `boolean` | Source Boolean distinguishing super-over deliveries. |
| `team_batting_id` | `int64` | Source batting-team ID; canonical name is unverified. |
| `innings_runs` | `int64` | Actual innings total including extras; super overs are retained and explicitly flagged. |
| `wickets_lost` | `Int64` | Sum of is_wicket_lost for the innings. |
| `legal_balls` | `Int64` | Count of deliveries not flagged wide or no-ball. |
| `overs_faced` | `Float64` | legal_balls divided by six (decimal overs, not cricket notation). |
| `run_rate` | `float64` | Total runs per six legal balls; null where no legal balls were recorded. |
| `fours` | `int64` | Count of deliveries with batter_runs equal to four. |
| `sixes` | `int64` | Count of deliveries with batter_runs equal to six. |
| `boundary_count` | `int64` | Number of batter-run fours plus sixes on the delivery/aggregate. |
| `boundary_runs` | `int64` | Four runs per batter-run four plus six runs per batter-run six. |
| `extras` | `int64` | Total extra runs on this delivery. |
| `powerplay_runs` | `float64` | Regulation runs in zero-based over indices 0–5; null for super overs. |
| `middle_over_runs` | `float64` | Regulation runs in zero-based over indices 6–14; null for super overs. |
| `death_over_runs` | `float64` | Regulation runs in zero-based over indices 15–19; null for super overs. |

Rows: 2,452.

### `ipl_batting_season_stats.csv`

Grain: one row per season and batter; regulation participation includes batter and non-striker appearances.

| Column | Data type | Definition |
|---|---|---|
| `season_id` | `int64` | Season identifier as supplied in the source. |
| `batter` | `string` | Source batter name after whitespace trimming only. |
| `batter_runs_scored` | `int64` | Sum of striker batter_runs only; extras are excluded. |
| `innings_batted` | `int64` | Count of regulation match-innings in which the player appears as batter or non-striker. |
| `balls_faced` | `int64` | Count of batter deliveries excluding wides; no-balls count as faced. |
| `fours` | `int64` | Count of deliveries with batter_runs equal to four. |
| `sixes` | `int64` | Count of deliveries with batter_runs equal to six. |
| `dismissals` | `int64` | Count of dismissed-player records excluding retired hurt; includes dismissals while listed as non-striker. |
| `strike_rate` | `float64` | 100 × total_runs / balls_faced; null when balls faced is zero. |

Rows: 2,959.

### `ipl_bowling_season_stats.csv`

Grain: one row per season and bowler.

| Column | Data type | Definition |
|---|---|---|
| `season_id` | `int64` | Season identifier as supplied in the source. |
| `bowler` | `string` | Source bowler name after whitespace trimming only. |
| `wickets` | `Int64` | Count of bowler-creditable wickets under the documented dismissal set. |
| `legal_balls` | `Int64` | Count of deliveries not flagged wide or no-ball. |
| `overs_bowled` | `Float64` | legal_balls divided by six (decimal overs, not cricket notation). |
| `runs_conceded` | `int64` | Bowler-charged runs; excludes byes, leg-byes, and penalty runs. |
| `economy_rate` | `float64` | Runs conceded per six legal balls; null when no legal balls were recorded. |
| `bowling_average` | `float64` | Runs conceded per credited wicket; null when no credited wickets. |
| `bowling_strike_rate` | `float64` | Legal balls per credited wicket; null when no credited wickets. |
| `dot_balls` | `Int64` | Count of legal deliveries on which no total runs were scored. |
| `dot_ball_percentage` | `float64` | 100 × dot_balls / legal_balls; null when no legal balls. |

Rows: 2,181.

### `ipl_team_season_stats.csv`

Grain: one row per season and batting-team ID; regulation innings only.

| Column | Data type | Definition |
|---|---|---|
| `season_id` | `int64` | Season identifier as supplied in the source. |
| `team_batting_id` | `int64` | Source batting-team ID; canonical name is unverified. |
| `innings_count` | `int64` | Count of regulation innings for this season and batting-team ID. |
| `team_runs` | `int64` | Sum of actual regulation innings runs including extras. |
| `total_legal_balls` | `Int64` | Sum of legal balls across the team's regulation innings. |
| `total_wickets_lost` | `Int64` | Sum of regulation innings wickets_lost for the team-season. |
| `total_extras` | `int64` | Sum of extras in the team's regulation innings. |
| `fours` | `int64` | Count of deliveries with batter_runs equal to four. |
| `sixes` | `int64` | Count of deliveries with batter_runs equal to six. |
| `boundaries` | `int64` | Count of batter-run fours and sixes. |
| `boundary_runs` | `int64` | Four runs per batter-run four plus six runs per batter-run six. |
| `average_innings_score` | `float64` | Mean actual runs per regulation innings. |
| `run_rate` | `float64` | Total runs per six legal balls; null where no legal balls were recorded. |

Rows: 166.

### `dim_season.csv`

Grain: one row per observed season.

| Column | Data type | Definition |
|---|---|---|
| `season_id` | `int64` | Season identifier as supplied in the source. |
| `season_label` | `int64` | Display label copied from season_id; not independently inferred. |

Rows: 19.

### `dim_team.csv`

Grain: one row per observed source team ID.

| Column | Data type | Definition |
|---|---|---|
| `team_id` | `int64` | Source team ID; no canonical name assigned without a verified mapping. |
| `canonical_team_name` | `string` | Null because the team ID-to-name mapping was not verified. |
| `mapping_status` | `object` | Mapping state; all listed source team IDs are unverified. |

Rows: 14.

### `dim_batter.csv`

Grain: one row per batter name.

| Column | Data type | Definition |
|---|---|---|
| `batter` | `object` | Source batter name after whitespace trimming only. |

Rows: 732.

### `dim_bowler.csv`

Grain: one row per bowler name.

| Column | Data type | Definition |
|---|---|---|
| `bowler` | `object` | Source bowler name after whitespace trimming only. |

Rows: 568.

### `data_quality_report.csv`

Grain: one row per validation check or source-column missingness item.

| Column | Data type | Definition |
|---|---|---|
| `check_name` | `object` | Name of a validation or missingness check. |
| `status` | `object` | PASS, WARN, or INFO validation status. |
| `observed` | `object` | Observed value or result for the check. |
| `expected` | `object` | Expected value or validation criterion. |
| `details` | `object` | Interpretation or limitation associated with the check. |
| `source_path` | `object` | Source CSV filename, without a machine-specific absolute path. |

Rows: 91.

### `unmapped_team_ids.csv`

Grain: one row per unresolved source team ID.

| Column | Data type | Definition |
|---|---|---|
| `team_id` | `int64` | Source team ID; no canonical name assigned without a verified mapping. |
| `batting_deliveries` | `int64` | Count of source deliveries with this ID batting. |
| `bowling_deliveries` | `int64` | Count of source deliveries with this ID bowling. |
| `seasons_present` | `object` | Comma-separated observed seasons for this source team ID. |
| `canonical_team_name` | `string` | Null because the team ID-to-name mapping was not verified. |
| `mapping_status` | `object` | Mapping state; all listed source team IDs are unverified. |

Rows: 14.

## Source fields

Original delivery columns are preserved in `ipl_deliveries_clean.csv`. Runs, extra components, flags, team IDs, player names, and dismissal data are not imputed. Optional missing values remain null.
