# Adding Age for prediction

Here, we add age by joining Jia's [`transform-2` output](../transform-2_four-seasons_jia/last_four_years_playerID_H.csv) to the Master.csv before using WEKA to perform analysis, just in case age has an impact on the prediction of the fourth season.

```sql
sqlite> .mode csv
sqlite> .headers on
sqlite> .import ./Master.csv player_master
sqlite> .import last_four_years_playerID_RBI.csv aRBI

# Join tables and output with age
sqlite> .output aRBI.csv
sqlite> select playerID,(yearID-birthYear) as age, season1, season2, season3, season4 from aRBI join player_master using (playerID);

```
