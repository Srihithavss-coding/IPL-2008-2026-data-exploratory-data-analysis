# IPL Ball-by-Ball EDA (2008–2026)

This project analyzes the delivery-level records in `ball_by_ball_data.csv` and exports validated, documented tables for Power BI. Season coverage is determined from the file at runtime; a season's presence does not establish that its data is complete.

## Run the notebook

1. Install Python with `pandas`, `numpy`, `matplotlib`, `seaborn`, `ipython`, and Jupyter.
2. Open `Exploratory_Data_Analysis_(EDA)_of_IPL_Matches_(2008_2026).ipynb` and run all cells from a fresh kernel.
3. By default, the notebook looks for `ball_by_ball_data.csv` in the working directory. To use another file, set `IPL_DATA_PATH` to its path. In Colab, place the CSV in the working directory or an already-mounted Drive path.
4. The notebook creates `outputs/powerbi/` with cleaned delivery, innings, batting, bowling, team, dimension, quality-report, and unmapped-team CSVs plus `data_dictionary.md`.

The notebook leaves the source CSV unchanged, reads `season_id` as a source column, and creates a separate working copy. It does not invent or impute dates, venues, winners, tosses, targets, or team names.

## Data and analytical scope

The supplied CSV contains ball-by-ball scoring, team identifiers, player names, dismissal fields, extras, delivery flags, innings, and season. It does not contain genuine match dates, venues, tosses, match winners, or title outcomes. No supplementary source with a validated `match_id` relationship was available, so match-result and venue analyses are intentionally omitted.

Team IDs are preserved as identifiers. The prior notebook's team-name mapping is not treated as authoritative; unresolved IDs are written to `outputs/powerbi/unmapped_team_ids.csv`. Player strings are whitespace-trimmed only; names and initials are not title-cased or guessed.

Super overs are identified separately and excluded from regulation batting, bowling, team-season, and scoring analyses. Delivery features use the source's wide and no-ball flags. Wides do not count as balls faced; no-balls do. Bowler-creditable wickets exclude run outs, retired hurt/out, and obstructing the field. Retired hurt is excluded from innings wickets lost.

## Outputs

- `ipl_deliveries_clean.csv`: one row per source delivery with validated features.
- `ipl_innings_summary.csv`: one row per season, match, innings, and super-over flag.
- `ipl_batting_season_stats.csv` and `ipl_bowling_season_stats.csv`: season/player metrics.
- `ipl_team_season_stats.csv`: season/team-ID scoring metrics.
- `dim_season.csv`, `dim_team.csv`, `dim_batter.csv`, `dim_bowler.csv`.
- `data_quality_report.csv`, `unmapped_team_ids.csv`, and `data_dictionary.md`.

No match-level result export is created without verified metadata. See the notebook and data dictionary for metric definitions, validation checks, and source limitations.
