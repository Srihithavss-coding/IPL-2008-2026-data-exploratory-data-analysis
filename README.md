# IPL Ball-by-Ball EDA and Power BI exports

## Objective and scope

This project audits the supplied IPL delivery file, calculates defensible batting, bowling, innings, and team-scoring measures, and exports documented datasets for Power BI. It is descriptive analysis, not a causal study. Season labels present in the file do not establish that a season is complete.

The source contains **288,226 delivery rows** across 19 observed season labels (2008–2026): **288,051 regulation deliveries** and **175 super-over deliveries**. The original CSV is not modified.

## Dataset schema and limitations

The 30 source fields cover season and match IDs, batter/bowler/non-striker, batting and bowling team IDs, over/ball indices, batter/extras/total runs and their extra components, player-type attributes, dismissal details, delivery flags, super-over status, and innings.

The supplied source does not include verified match dates, venues, winners, tosses, or title outcomes. No validated supplementary match metadata was available, so match-result and venue analyses are excluded. Team IDs remain numeric and unresolved because the repository does not establish an authoritative ID-to-name mapping. The source's over and ball indices are zero-based. Super overs are retained and flagged in delivery and innings-summary exports, but excluded from batting, bowling, and team-season statistics.

Delivery-level `total_runs` includes extras. Batter runs exclude extras; innings and team totals include extras. Wides do not count as balls faced, while no-balls do. Bowler-creditable wickets are restricted to bowled, caught, caught and bowled, lbw, stumped, and hit wicket. Innings wickets lost use `is_wicket_lost`, which excludes retired hurt.

The original publisher/source provenance and data licence are not documented in this repository. No third-party source citation or licence is claimed.

## Findings from the corrected notebook

- **V Kohli** has the highest aggregate regulation batter runs in the supplied data: **9,040**.
- **YS Chahal** has the most regulation bowler-creditable wickets: **228**.
- The fastest recorded fifty in the supplied data is **YBK Jaiswal's 13-ball fifty in 2023**.
- The corrected batting aggregation contains **2,959 season-player rows** and **732 batter/non-striker names**. It accounts for **14,290 regulation dismissals**, including the previously missed A Nel (2008), S Tyagi (2009), and DA Miller (2020) dismissals.

These are descriptive results for the supplied file, not claims about complete official season records.

## Power BI exports

All deliverables are in [`outputs/powerbi/`](outputs/powerbi/):

- `ipl_deliveries_clean.csv` — one row per original delivery, with source columns preserved and validated delivery features added.
- `ipl_innings_summary.csv` — one row per season, match, innings, and super-over flag.
- `ipl_batting_season_stats.csv` — one row per season and batter, including non-striker appearances in innings counts; scoring metrics accrue only from striker deliveries.
- `ipl_bowling_season_stats.csv` — one row per season and bowler.
- `ipl_team_season_stats.csv` — one row per season and batting-team ID.
- `dim_season.csv`, `dim_team.csv`, `dim_batter.csv`, and `dim_bowler.csv`.
- `data_quality_report.csv` and `unmapped_team_ids.csv`.
- `data_dictionary.md` — grains, column definitions, calculation rules, super-over guidance, and example DAX measures.

The delivery and innings-summary tables retain super overs. Exclude super overs by default in Power BI measures when comparing delivery data with regulation batting, bowling, and team-season tables. The data dictionary provides DAX examples; use `is_wicket_lost` for innings wickets and `bowler_wickets` for credited bowling wickets.

## Run and validate

Use Python 3 and install the tested dependencies:

```powershell
python -m pip install -r requirements.txt
```

Open `Exploratory_Data_Analysis_(EDA)_of_IPL_Matches_(2008_2026).ipynb` in Jupyter and run all cells from a fresh kernel. The notebook looks for `ball_by_ball_data.csv` in the working directory; set `IPL_DATA_PATH` to use another file. In Colab, place the CSV in the working directory or an already-mounted Drive path.

To execute all cells and save the outputs:

```powershell
jupyter nbconvert --to notebook --execute --inplace "Exploratory_Data_Analysis_(EDA)_of_IPL_Matches_(2008_2026).ipynb" --ExecutePreprocessor.timeout=300 --ExecutePreprocessor.kernel_name=python3
```

The export cell checks grouped batting, dismissal, bowling, and team reconciliations, reads generated CSVs back to validate schemas and row counts, and writes the outputs under `outputs/powerbi/`.
