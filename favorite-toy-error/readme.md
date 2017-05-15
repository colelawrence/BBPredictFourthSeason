# Generating Favorite Toy Prediction RMSE

Here, we use Jia's [`transform-2` output](../transform-2_four-seasons_jia/last_four_years_playerID_H.csv) to create an easy way for us to evaluate a base error to reference for our own success.

```sql
sqlite> .mode csv
sqlite> .headers on
sqlite> .import ./Master.csv player_master

;# Create Homeruns from Jia's output
sqlite> .import last_four_years_playerID_H.csv aH

;# Favorite Toy algorithm test
sqlite> select playerID,((season3*3 + season2*2 + season1) * 0.166667) as guess,(.75 * season3) as at_least, season4 from aH limit 10;

;# Output
.output bH.csv
;# Run again without limit
select playerID,((season3*3 + season2*2 + season1) * 0.166667) as guess,(.75 * season3) as at_least, season4 from aH;
```