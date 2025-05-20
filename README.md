# IPL Data Analysis (2008–2020)

This repository contains a PySpark-based exploratory data analysis of the Indian Premier League (IPL) dataset from 2008 to 2020. The goal is to extract meaningful insights about teams, players, and matches using distributed computing with PySpark.

## Dataset

The dataset is sourced from Kaggle:  
**[IPL Complete Dataset (2008–2020)](https://www.kaggle.com/datasets/patrickb1912/ipl-complete-dataset-20082020)**

It contains two CSV files:

- `matches.csv`: Match-level data (team names, winner, season, toss, venue, etc.)
- `deliveries.csv`: Ball-by-ball records (batsman, bowler, runs, wickets, etc.)

## Technologies Used

- PySpark (Apache Spark DataFrame API)
- Jupyter Notebook
- Python 3


---

## Key Analyses

### 1. Team with Maximum and Minimum Match Wins
```python 
team_wins = matches.groupBy("winner") \
    .agg(count("*").alias("total_wins")) \
    .filter(col("winner").isNotNull()) \
    .orderBy(col("total_wins").desc())

Most Wins:
- Mumbai Indians: 120
- Chennai Super Kings: 111

Least Wins:
- Kochi Tuskers Kerala: 6
- Pune Warriors: 12
```

### 2. Team with Most and Least Trophies (Final Match Winners)
```
# Get final match of each season
window_spec = Window.partitionBy("season").orderBy(col("date").desc())
final_matches = matches.withColumn("row_num", row_number().over(window_spec)) \
    .filter(col("row_num") == 1).select("season", "winner")

# Count trophies
trophies = final_matches.groupBy("winner") \
    .agg(count("*").alias("trophies")) \
    .orderBy(col("trophies").desc())
Most Trophies:
- Chennai Super Kings: 5
- Mumbai Indians: 5

Least Trophies (1 each):
- Rajasthan Royals
- Deccan Chargers
- Sunrisers Hyderabad
- Gujarat Titans
```

### 3. Top Wicket-Takers
```
top_wicket_takers = deliveries.filter(col("is_wicket") == True) \
    .filter(col("dismissal_kind").isNotNull()) \
    .groupBy("bowler") \
    .agg(count("dismissal_kind").alias("wickets")) \
    .orderBy(col("wickets").desc())

- Lasith Malinga: 170
- DJ Bravo: 153
- Bhuvneshwar Kumar: 133
```

### 4. Economy Rate of Bowlers
```
economy_rate = deliveries.groupBy("bowler") \
    .agg(
        sum("total_runs").alias("runs_conceded"),
        count("ball").alias("balls_bowled")
    ) \
    .withColumn("economy_rate", col("runs_conceded") / (col("balls_bowled") / 6)) \
    .orderBy("economy_rate")

- Rashid Khan: 6.38
- Mujeeb Ur Rahman: 6.40
- Sunil Narine: 6.67
```

### 5. Strike Rate of Batsmen
```
batsman_strike_rate = deliveries.groupBy("batter") \
    .agg(
        sum("batsman_runs").alias("total_runs"),
        count("ball").alias("balls_faced")
    ) \
    .withColumn("strike_rate", (col("total_runs") / col("balls_faced")) * 100)

Results:
- Andre Russell: 182.33
- Glenn Maxwell: 161.13
- AB de Villiers: 151.68
```

### 6. Best Batting Partnerships
```
partnerships = deliveries.withColumn("pair", concat_ws(" & ", "batter", "non_striker")) \
    .groupBy("match_id", "pair") \
    .agg(sum("batsman_runs").alias("runs")) \
    .orderBy(col("runs").desc())

Results:
- Virat Kohli & AB de Villiers: 229 runs (RCB vs GL, 2016)
```

### Notes
This project focuses on statistical and historical insights; no machine learning is involved.
The notebook is optimized for PySpark on local mode.
All calculations assume the latest match by date is the final for each season.
