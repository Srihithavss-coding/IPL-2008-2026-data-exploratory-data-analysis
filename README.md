# **IPL Ball-by-Ball Exploratory Data Analysis (2008 - 2026)**

This repository/notebook contains a robust, end-to-end **Exploratory Data Analysis (EDA)** of the Indian Premier League (IPL) using comprehensive ball-by-ball match data from 2008 to 2026. 

## **Project Overview**
The IPL is one of the most dynamic and fast-paced T20 cricket leagues in the world. Decisions are made in fractions of a second, and match-winning strategies require deep tactical planning. The core objective of this project is to answer a fundamental sports analytics question:
> **What factors drive match-winning outcomes the most (e.g., batting first vs. chasing), and who are the historical key individual contributors (batsmen and bowlers) influencing these results?**

---

## **Key Features & Analyses**
1. **Data Schema Migration:** Standardized complex ball-by-ball variables (`match_id`, `team_batting`, `is_wicket`, etc.) to clean, standardized features across nearly two decades of seasonal transitions.
2. **Tactical Team Analysis:** Evaluated the historic success rate of **Batting First vs. Chasing Wins** over time, revealing that no single strategy guarantees dominance.
3. **Individual Milestones:** 
   * **Orange Cap Winners:** Highest seasonal run-scorers and modern scoring velocity.
   * **Purple Cap Winners:** Leading seasonal wicket-takers and tactical depth.
   * **Fastest Fifties:** Pinpointing explosive batting impacts.
   * **Power-Hitting & Dot Ball Efficiency:** Aggressive six-hitting trends mapped against bowlers who create high pressure through dot balls.
4. **Venue Impact:** Visualized which historic grounds hosted the most matches and how core home advantages are engineered.

---

## **Data Structure & Variables**
* `Match ID` / `Date` / `Venue`: Game identification and time series mapping.
* `Bat First` / `Bat Second` / `Winner`: Team categorical matches.
* `Batter Runs` / `Extra Runs` / `Runs From Ball`: Ball-level scoring mechanics.
* `Wicket` / `Method` / `Player Out`: Dismissal classifications.
* `Innings Runs` / `Target Score` / `Runs to Get`: Cumulative state metrics.
* `is_dot_ball` / `is_six` / `bowler_wickets`: Engineered features for core performance analytics.

---

## **Technologies Used**
* **Python** (v3.13)
* **Pandas** & **NumPy** (Data cleaning, state calculations, and aggregation)
* **Matplotlib** & **Seaborn** (Custom high-quality tactical visualizations)
* **Google Colab** (Interactive notebook runtime & Google Drive integrations)

---

## **How to Run**
1. Open this notebook in Google Colab.
2. Mount your Google Drive when prompted, or place the dataset at `/content/ball_by_ball_data.csv` to fallback automatically to local execution.
3. Run all cells sequentially to reproduce the data pipeline, engineered features, and visualizations.
