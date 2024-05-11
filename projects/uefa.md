## Background
The UEFA Champions League is one of the most prestigious football competitions in the world, showcasing top European football clubs competing for glory. Analyzing match data from the tournament can provide valuable insights into team performance, player strategies, and match dynamics.


![](https://i.pinimg.com/originals/d6/81/85/d681852ff7d04442e8da535a288968d8.png)


## Motivation
Understanding the patterns and trends within the UEFA Champions League matches can aid coaches, analysts, and enthusiasts in refining strategies, predicting outcomes, and gaining a deeper appreciation for the game.

## The Data
The data used for analysis encompasses match statistics from multiple seasons of the UEFA Champions League, including team names, scores, possession percentages, duels won, shots on target, and match locations.

## Analysis and Results
1. **Top Scoring Teams**: Identifying the top three scoring teams in the tournament based on their home game scores.
   
```sql
SELECT TEAM_NAME_HOME, TEAM_HOME_SCORE
FROM uefa2020
ORDER BY TEAM_HOME_SCORE DESC
LIMIT 3;
```

| Team           | Total Goals | 1 Game Max | Avg Goals Per Game | Games Played |
|----------------|-------------|------------|--------------------|--------------|
| Bayern Munich  | 28          | 7          | 3.50               | 8            |
| Man City       | 28          | 7          | 3.50               | 8            |
| PSG            | 22          | 7          | 3.14               | 7            |
| Benfica        | 21          | 6          | 3.00               | 7            |
| Liverpool      | 18          | 7          | 3.00               | 6            |


2. **Teams Losing Despite Duel Wins**: Identifying teams that lost matches despite winning more duels.

```sql
SELECT TEAM_LOST, COUNT(*) AS LOST_WITH_DUELS_WON
FROM (
    SELECT STAGE,
        CASE 
            WHEN DUELS_WON_HOME < DUELS_WON_AWAY AND TEAM_HOME_SCORE > TEAM_AWAY_SCORE THEN TEAM_NAME_AWAY
            WHEN DUELS_WON_HOME > DUELS_WON_AWAY AND TEAM_HOME_SCORE < TEAM_AWAY_SCORE THEN TEAM_NAME_HOME
            ELSE NULL 
        END AS TEAM_LOST
    FROM UEFA_ALL
) AS SUB
WHERE TEAM_LOST IS NOT NULL
GROUP BY TEAM_LOST
LIMIT 5;
```

| Team               | Lost with Duels Won |
|--------------------|----------------------|
| PSG                | 5                    |
| Chelsea            | 5                    |
| FC Porto           | 3                    |
| Bayern Munich      | 2                    |
| RB Leipzig         | 4                    |


![](https://assets.goal.com/images/v3/blte092c3a8fb02c79b/Kylian_Mbappe_PSG_2023-24_(13).jpg?auto=webp&format=pjpg&width=3840&quality=60)

3. **Average Shots on Target & Winning Away from Home**: Computing the average shot percentage for away teams and analysing their wins.

```sql
WITH COMBINE1 AS (
    SELECT ROUND(AVG(TOTAL_SHOTS_AWAY),2) AS AVG_AWAY_SHOTS_AWAY_WINS, 
           COUNT(*) AS AWAY_WINS
    FROM UEFA_ALL
    WHERE TEAM_AWAY_SCORE > TEAM_HOME_SCORE
),
COMBINE2 AS (
    SELECT ROUND(AVG(TOTAL_SHOTS_AWAY),2) AS AVG_AWAY_SHOTS_AWAY_LOSSES, 
           COUNT(*) AS AWAY_LOSSES
    FROM UEFA_ALL
    WHERE TEAM_HOME_SCORE > TEAM_AWAY_SCORE
)
SELECT COMBINE1.AVG_AWAY_SHOTS_AWAY_WINS, 
       COMBINE1.AWAY_WINS,
       COMBINE2.AVG_AWAY_SHOTS_AWAY_LOSSES,
       COMBINE2.AWAY_LOSSES
FROM COMBINE1
CROSS JOIN COMBINE2;
```

| AVG_AWAY_SHOTS_AWAY_WINS | AWAY_WINS | AVG_AWAY_SHOTS_AWAY_LOSSES | AWAY_LOSSES |
|---------------------------|-----------|----------------------------|-------------|
| 15.43                     | 130       | 12.04                      | 168         |



4. **Average Possession by Team**: Determine the team with the highest average possession throughout the season.

```sql
   WITH POSSESSION AS (
SELECT TEAM_NAME_HOME AS TEAM_NAME, 
    ROUND(AVG(POSSESSION_HOME),2) AS POSS, 
    SUM(CASE WHEN TEAM_HOME_SCORE > TEAM_AWAY_SCORE THEN 1 ELSE 0 END) AS WINS
FROM UEFA_ALL
GROUP BY TEAM_NAME
UNION ALL
SELECT TEAM_NAME_AWAY AS TEAM_NAME, 
    ROUND(AVG(POSSESSION_AWAY),2) AS POSS, 
    SUM(CASE WHEN TEAM_HOME_SCORE < TEAM_AWAY_SCORE THEN 1 ELSE 0 END) AS WINS
FROM UEFA_ALL
GROUP BY TEAM_NAME)

SELECT TEAM_NAME, ROUND((SUM(POSS)/2),2) AS AVG_POSS_PCT, SUM(WINS)
FROM POSSESSION
GROUP BY TEAM_NAME
ORDER BY AVG_POSS_PCT DESC
LIMIT 5;
```

| TEAM_NAME      | AVG_POSS_PCT | SUM(WINS) |
|----------------|--------------|-----------|
| Man City       | 59.94        | 26        |
| Barcelona      | 59.55        | 9         |
| Bayern Munich  | 58.92        | 22        |
| Napoli         | 56.4         | 7         |
| Ajax           | 56.2         | 10        |

5. **Home Team Win Percentage**: Calculating the overall win percentage for home teams.
   
```sql
SELECT 
SUM(CASE WHEN TEAM_HOME_SCORE > TEAM_AWAY_SCORE THEN 1 ELSE 0 END) AS HOME_WINS,
COUNT(*) AS ALL_MATCHES,
ROUND(SUM(CASE WHEN TEAM_HOME_SCORE > TEAM_AWAY_SCORE THEN 1 ELSE 0 END)*100/COUNT(*),2) AS HOME_WIN_PCT
FROM UEFA_ALL;
```

| HOME_WINS | ALL_MATCHES | HOME_WIN_PCT |
|-----------|-------------|--------------|
| 168       | 374         | 44.92        |

6. **Draw Percentage**: Computing the percentage of matches that ended in draws.


```sql
SELECT 
SUM(CASE WHEN TEAM_HOME_SCORE = TEAM_AWAY_SCORE THEN 1 ELSE 0 END) AS DRAWS,
COUNT(*) AS ALL_MATCHES,
ROUND(SUM(CASE WHEN TEAM_HOME_SCORE = TEAM_AWAY_SCORE THEN 1 ELSE 0 END)*100/COUNT(*),2) AS DRAWS_PCT
FROM UEFA_ALL;
```

| DRAWS | ALL_MATCHES | DRAWS_PCT |
|-------|-------------|-----------|
| 76    | 374         | 20.32     |



7. **Away Team Win Percentage**: Determining the overall win percentage for away teams.

```sql
SELECT 
SUM(CASE WHEN TEAM_HOME_SCORE < TEAM_AWAY_SCORE THEN 1 ELSE 0 END) AS AWAY_WINS,
COUNT(*) AS ALL_MATCHES,
ROUND(SUM(CASE WHEN TEAM_HOME_SCORE < TEAM_AWAY_SCORE THEN 1 ELSE 0 END)*100/COUNT(*),2) AS AWAY_WIN_PCT
FROM UEFA_ALL;
```


| AWAY_WINS | ALL_MATCHES | AWAY_WIN_PCT |
|-----------|-------------|--------------|
| 130       | 374         | 34.76        |




8. **Outliers in Possession Percentage**: Identifying any outliers in possession percentage.

```sql
SELECT ROUND(AVG(POSSESSION_HOME),2) AS AVG_POSS_H
FROM UEFA_ALL
WHERE TEAM_HOME_SCORE>TEAM_AWAY_SCORE;

SELECT TEAM_NAME_HOME, TEAM_HOME_SCORE, TEAM_AWAY_SCORE, TEAM_NAME_AWAY, POSSESSION_HOME, POSSESSION_AWAY
FROM UEFA_ALL
WHERE TEAM_HOME_SCORE>=TEAM_AWAY_SCORE AND POSSESSION_HOME<30;
```

| AVG_POSS_H |
|------------|
| 51.56      |


| TEAM_NAME_HOME     | TEAM_HOME_SCORE | TEAM_AWAY_SCORE | TEAM_NAME_AWAY    | POSSESSION_HOME | POSSESSION_AWAY |
|--------------------|-----------------|-----------------|-------------------|-----------------|-----------------|
| PSG                | 1               | 1               | Barcelona         | 27%             | 73%             |
| Salzburg           | 1               | 1               | Bayern Munich     | 28%             | 72%             |
| Inter Milan        | 1               | 0               | Barcelona         | 28%             | 72%             |
| Borussia Dortmund  | 0               | 0               | Man City          | 27%             | 73%             |


![](https://i.ytimg.com/vi/BYaz43TkeMQ/maxresdefault.jpg)

9. **Correlation Between Shots on Target and Score Difference**: Investigating if there's a correlation between the number of shots on target and the final score difference.


```sql
WITH WinningTeamShots AS (
    SELECT
        CASE WHEN TEAM_HOME_SCORE > TEAM_AWAY_SCORE THEN SHOTS_ON_TARGET_HOME
		ELSE SHOTS_ON_TARGET_AWAY END AS WINNING_TEAM_SHOTS,
        ABS(TEAM_HOME_SCORE - TEAM_AWAY_SCORE) AS SCORE_DIFF
    FROM UEFA_ALL
    WHERE TEAM_HOME_SCORE != TEAM_AWAY_SCORE)

SELECT ROUND(AVG(WINNING_TEAM_SHOTS),2) AS AVG_WINNING_TEAM_SHOTS, SCORE_DIFF
FROM WinningTeamShots
GROUP BY SCORE_DIFF
ORDER BY SCORE_DIFF;
```

| AVG_WINNING_TEAM_SHOTS | SCORE_DIFF |
|-------------------------|------------|
| 5.16                    | 1          |
| 5.48                    | 2          |
| 6.21                    | 3          |
| 7.03                    | 4          |
| 9.25                    | 5          |
| 9.33                    | 6          |
| 16.00                   | 7          |


10. **Possession Percentage Across Tournament Stages**: Examining how the distribution of possession percentage varies across different stages of the tournament.


```sql
WITH POSS_DIFF AS
(SELECT STAGE,
CASE WHEN POSSESSION_HOME>=POSSESSION_AWAY THEN POSSESSION_HOME
ELSE POSSESSION_AWAY END AS MOST_POSS_SHARE
FROM UEFA_ALL)

SELECT CASE WHEN STAGE LIKE '%GROUP%' THEN 'GROUP STAGES'
WHEN STAGE LIKE '%16%' THEN 'ROUND OF 16'
WHEN STAGE LIKE '%quarter%' THEN 'QUARTERS'
WHEN STAGE LIKE '%SEMI%' THEN 'SEMIS'
WHEN STAGE LIKE '%FINAL%' THEN 'FINALS'
ELSE 'OTHER' END AS STAGE_NAME,
ROUND(AVG(MOST_POSS_SHARE),2) AS TEAM_WITH_HIGHEST_POSS_PCT
FROM POSS_DIFF
GROUP BY STAGE_NAME;
```


| STAGE_NAME    | TEAM_WITH_HIGHEST_POSS_PCT |
|---------------|----------------------------|
| FINALS        | 56                         |
| SEMIS         | 59.33                      |
| QUARTERS      | 60.54                      |
| ROUND OF 16   | 60.94                      |
| GROUP STAGES  | 58.53                      |



11. **Wins by Home Venue**: Identifying locations where teams have secured the most wins, excluding finals matches.

```sql
SELECT LOCATION, COUNT(*) AS WINS, TEAM_NAME_HOME
FROM UEFA_ALL
WHERE TEAM_HOME_SCORE>TEAM_AWAY_SCORE AND STAGE NOT LIKE 'FINAL'
GROUP BY LOCATION, TEAM_NAME_HOME
ORDER BY WINS DESC
LIMIT 5;
```

| LOCATION           | WINS | TEAM_NAME_HOME  |
|--------------------|------|-----------------|
| Etihad Stadium     | 13   | Man City        |
| Allianz Arena      | 10   | Bayern Munich   |
| Stamford Bridge    | 8    | Chelsea         |
| Santiago Bernabeu  | 8    | Real Madrid     |
| Anfield            | 7    | Liverpool       |

![](https://res.cloudinary.com/brentford-fc/image/upload/f_auto,q_auto:best,f_auto,q_100,c_fill,ar_16:9/Production/Etihad_GV_2_girunv.jpg) 


These analyses provide valuable insights into various aspects of UEFA Champions League matches, aiding in strategic planning, performance evaluation, and understanding match dynamics.
