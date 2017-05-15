# Aggregating Stats

Using the baseball-databank-master/core files,
we join all the tables together using the playerID.

Here are Cole's notes made while creating [`./batters_with_four_seasons_of_ten_hits.csv`](./batters_with_four_seasons_of_ten_hits.csv)

```sql
# SQL Baseball notes

# COLUMN = "HR"
# FILE = ".//Batting.csv"
# OUTPUT = ".//lame.csv"

# baseball-databank-master/core> sqlite3

# We can analyze:
# Batting.csv:
#  * G,AB,R,H,2B,3B,HR,RBI,SB,CS,BB,SO,IBB,HBP,SH,SF
# Pitching.csv:
#  * W,L,G,GS,CG,SHO,SV,IPouts,H,ER,HR,BB,SO,BAOpp,ERA,IBB,WP,HBP,BK,BFP,GF,R,SH,SF
# Fielding.csv:
#  * G,GS,InnOuts,PO,A,E,DP,PB,WP,SB,CS,ZR


# Importing everything:
# ======

.mode csv
# Import file "./Batting.csv" into table "batting"
.import Batting.csv batting
.import Pitching.csv pitching
.import Master.csv master

# 27 most seasons played anyway
SELECT playerID, yearID, SUM(H) \
  from batting WHERE playerID="brainas01" \
  GROUP BY playerID, yearID LIMIT 27;

# Experimenting with Batting:
# ======

# Aggregate all stats for each player-year
CREATE VIEW B50 AS
  SELECT playerID, yearID,
    SUM(G) AS B_G, SUM(AB) AS B_AB, SUM(R) AS B_R,
    SUM(H) AS B_H,
    SUM("2B") AS B_2B, SUM("3B") AS B_3B, SUM(HR) AS B_HR,
    SUM(RBI) AS B_RBI, SUM(SB) AS B_SB, SUM(CS) AS B_CS,
    SUM(BB) AS B_BB, SUM(SO) AS B_SO, SUM(IBB) AS B_IBB,
    SUM(HBP) AS B_HBP, SUM(SH) AS B_SH, SUM(SF) AS B_SF
  FROM batting WHERE yearID >= 1950
  GROUP BY playerID, yearID;

CREATE VIEW players_with_ten_hits AS SELECT playerID, yearID FROM B50 WHERE B_H >= 10;
CREATE VIEW players_with_four_seasons_of_ten_hits AS
  SELECT playerID FROM (
    SELECT playerID, COUNT(yearID) AS seasons
    FROM players_with_ten_hits GROUP BY playerID
  ) WHERE seasons >= 4;

# Turn on csv headers in output
.headers on

.output batters_with_four_seasons_of_ten_hits.csv
SELECT * FROM players_with_four_seasons_of_ten_hits INNER JOIN B50 USING (playerID); 

# output back to console
.output stdout

/* Let's check if it worked by looking up a player with fewer than 4 seasons of data. */
SELECT playerID FROM (
  SELECT playerID, COUNT(yearID) AS seasons
  FROM players_with_ten_hits GROUP BY playerID
) WHERE seasons < 4 LIMIT 10;

# Looks guten


# Experimenting with Pitching:
# ======

CREATE VIEW P50 AS
  SELECT playerID, yearID,
    SUM(W) AS P_W, SUM(L) AS P_L, SUM(G) AS P_G, SUM(GS) AS P_GS,
    SUM(CG) AS P_CG, SUM(SHO) AS P_SHO, SUM(SV) AS P_SV,
    SUM(IPouts) AS P_IPouts, SUM(H) AS P_H, SUM(ER) AS P_ER,
    SUM(HR) AS P_HR, SUM(BB) AS P_BB, SUM(SO) AS P_SO,
    SUM(BAOpp) AS P_BAOpp, SUM(ERA) AS P_ERA, SUM(IBB) AS P_IBB,
    SUM(WP) AS P_WP, SUM(HBP) AS P_HBP, SUM(BK) AS P_BK,
    SUM(BFP) AS P_BFP, SUM(GF) AS P_GF, SUM(R) AS P_R,
    SUM(SH) AS P_SH, SUM(SF) AS P_SF
  FROM pitching WHERE yearID >= 1950
  GROUP BY playerID, yearID;

# Who are the 5 heaviest players?
SELECT playerID, nameGiven, weight, birthYear from master ORDER BY CAST(weight as INTEGER) desc limit 5;

# Joined
CREATE VIEW PB50 AS
  SELECT * FROM B50 JOIN P50 USING (playerID, yearID);

# Info on table and columns
PRAGMA TABLE_INFO(B50);

# Example SQL:
# List 5 highest year totals for 2 base hits
SELECT playerID, yearID, "2B"
  FROM B50
  ORDER BY "2B" DESC
  LIMIT 5;
```
