# Background
The UEFA Champions League is one of the most prestigious football competitions in the world, showcasing top European football clubs competing for glory. Analyzing match data from the tournament can provide valuable insights into team performance, player strategies, and match dynamics.


![](https://i.pinimg.com/originals/d6/81/85/d681852ff7d04442e8da535a288968d8.png)


# Motivation
Understanding the patterns and trends within the UEFA Champions League matches can aid coaches, analysts, and enthusiasts in refining strategies, predicting outcomes, and gaining a deeper appreciation for the game.

# The Data
The data used for analysis encompasses match statistics from multiple seasons of the UEFA Champions League, including team names, scores, possession percentages, duels won, shots on target, and match locations.

# Analysis and Results
## 1. **Top Scoring Teams**: Identifying the top three scoring teams in the tournament based on their home game scores.
   
```sql
WITH COMBINED AS 
(SELECT TEAM_NAME_HOME AS TEAM, TEAM_HOME_SCORE AS GOALS_SCORED
FROM UEFA_ALL
UNION
SELECT TEAM_NAME_AWAY AS TEAM, TEAM_AWAY_SCORE AS GOALS_SCORED
FROM UEFA_ALL)

SELECT TEAM, SUM(GOALS_SCORED) AS TOTAL_GOALS, MAX(GOALS_SCORED) AS 1_GAME_MAX, ROUND(AVG(GOALS_SCORED),2) AS AVG_GOALS_PER_G,
COUNT(*) AS GAMES_PLAYED
FROM COMBINED
GROUP BY TEAM
ORDER BY TOTAL_GOALS DESC
LIMIT 5;
```

| Team           | Total Goals | 1 Game Max | Avg Goals Per Game | Games Played |
|----------------|-------------|------------|--------------------|--------------|
| Bayern Munich  | 28          | 7          | 3.50               | 8            |
| Man City       | 28          | 7          | 3.50               | 8            |
| PSG            | 22          | 7          | 3.14               | 7            |
| Benfica        | 21          | 6          | 3.00               | 7            |
| Liverpool      | 18          | 7          | 3.00               | 6            |


From the output, it's evident that Bayern Munich and Man City are the top-scoring teams, both with a total of 28 goals and an average of 3.50 goals per game across 8 games. PSG follows closely behind with 22 goals scored in 7 games, averaging 3.14 goals per game. Other notable teams include Benfica and Liverpool, with 21 and 18 goals respectively. This analysis helps in understanding the offensive prowess of each team in the tournament. This translates into such teams reaching the latter stages of the competition on a regular basis. 

## 2. **Teams Losing Despite Duel Wins**: Identifying teams that lost matches despite winning more duels.

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
| Atletico Madrid   | 6                    |
| PSG                | 5                    |
| Chelsea            | 5                    |
| Barcelona          | 5                    |
| Marseille          | 5                    |

Atletico Madrid stands out with 6 such losses, followed by PSG, Chelsea, Barcelona, and Marseille, each with 5 losses under similar circumstances. These numbers highlight the intriguing aspect of soccer where dominance in certain aspects of the game doesn't always translate into victory.

![](https://e00-marca.uecdn.es/assets/multimedia/imagenes/2020/01/13/15789472789047.jpg)

## 3. **Average Shots on Target & Winning Away from Home**: Computing the average shot percentage for away teams and analysing their wins.

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

The resulting table unveils that away teams, on average, managed 15.43 shots on target in matches they won, accumulating a total of 130 victories. Conversely, in matches where the away team faced defeat, the average shots on target dropped to 12.04, with a total of 168 losses recorded. This analysis sheds light on the significance of shot accuracy and offensive efficiency in determining match outcomes for away teams. It underscores the importance of clinical finishing and strategic execution, especially in away fixtures where teams face additional challenges. 

## 4. **Average Possession by Team**: Determine the team with the highest average possession throughout the season.

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

At the pinnacle here stands Man City, exhibiting remarkable ball control with an average possession of 59.94%, correlating with 26 wins. Barcelona closely trails with 59.55%, but only securing 9 victories. Bayern Munich follows suit with 58.92% possession and a cconsiderable 22 wins. Further down the list, Napoli and Ajax demonstrate formidable possession statistics of 56.4% and 56.2%, respectively, coupled with 7 and 10 victories. This analysis underscores the correlation between ball possession and match success, emphasizing the strategic importance of maintaining control over the game's tempo and rhythm.

## 5. **Home Team Win Percentage**: Calculating the overall win percentage for home teams.
   
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

This statistic underscores the significance of home-field advantage and the pivotal role it plays in shaping match outcomes.

## 6. **Draw Percentage**: Computing the percentage of matches that ended in draws.


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

Draws reflect the closely contested nature of matches and serve as a testament to the parity among participating teams.

## 7. **Away Team Win Percentage**: Determining the overall win percentage for away teams.

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

This statistic highlights the ability of teams to perform effectively in unfamiliar environments and underscores the importance of tactical acumen and adaptability in securing positive results away from home.


## 8. **Outliers in Possession Percentage**: Identifying any outliers in possession percentage.

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

| STAGE                   | TEAM_NAME_HOME | TEAM_HOME_SCORE | TEAM_AWAY_SCORE | TEAM_NAME_AWAY | TOTAL_SHOTS_HOME | TOTAL_SHOTS_AWAY | SHOTS_ON_TARGET_AWAY | SHOTS_ON_TARGET_AWAY | DUELS_WON_HOME | DUELS_WON_AWAY |
|-------------------------|-----------------|-----------------|-----------------|----------------|------------------|------------------|----------------------|----------------------|----------------|----------------|
| Group stage: Matchday 3 | Inter Milan     | 1               | 0               | Barcelona      | 5                | 7                | 2                    | 2                    | 37%            | 63%            |

One very noticeable fixture here is Inter Milan v Barcelona. Despite only having 28% possession, Inter managed to grind out a victory. Despite Barcelona's higher total shots, with 7 attempts compared to Inter Milan's 5, Inter managed to secure the win with efficient shot conversion. Notably, both teams had an equal number of shots on target, with 2 shots each. Inter Milan demonstrated prowess in duels, winning 37% of them, while Barcelona won the majority with 63%. This match exemplifies how efficiency in shot conversion and strategic dueling can outweigh mere shot quantity.


![](https://i.ytimg.com/vi/BYaz43TkeMQ/maxresdefault.jpg)

## 9. **Correlation Between Shots on Target and Score Difference**: Investigating if there's a correlation between the number of shots on target and the final score difference.


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

As we observe the data, there seems to be a positive correlation between the score difference and the average number of shots on target. Generally, as the score difference increases, so does the average number of shots on target by the winning team. For example, when the score difference is 1 goal, the average number of shots on target is 5.16, and it steadily increases to 16.00 when the score difference is 7 goals.

This suggests that teams tend to score more goals when they have a higher number of shots on target, resulting in a larger margin of victory. 

## 10. **Possession Percentage Across Tournament Stages**: Examining how the distribution of possession percentage varies across different stages of the tournament.


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

In this output, the figures show the percentage of possession held by the most dominant team(in terms of possession). We observe fluctuations in possession percentage across different stages. In the Group Stages, the dominant teams tend to have a larger possession percentage of approximately 58.53%, which slightly increases as the tournament progresses. Notably, possession percentages peak during the Round of 16 and Quarterfinals, with averages of 60.94% and 60.54%, respectively. This suggests that teams may prioritize ball possession during the knockout stages of the tournament, possibly to control the pace of the game and create scoring opportunities. Interestingly, possession percentages dip slightly during the Semifinals and Finals, possibly indicating more balanced gameplay and intense competition as teams vie for the championship title.

## 11. **Wins by Home Venue**: Identifying locations where teams have secured the most wins, excluding finals matches.

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

The Etihad Stadium emerges as the venue with the highest number of wins, with teams securing victory 13 times when playing at home. Manchester City notably dominates their home ground, showcasing a strong performance in front of their home crowd.

Following closely is the Allianz Arena, where Bayern Munich has clinched victory on 10 occasions. The Bavarian powerhouse demonstrates formidable strength when playing at home, leveraging their home advantage to secure significant wins.

Stamford Bridge, the home ground of Chelsea, ranks third in the list with 8 wins. The London-based club exhibits a solid performance on their home turf, making Stamford Bridge a formidable venue for visiting teams.

Santiago Bernabeu, the iconic stadium of Real Madrid, shares the third position with 8 wins. Real Madrid boasts a rich history of success at their home stadium, showcasing their dominance in UEFA matches.

Anfield, the legendary home ground of Liverpool, completes the top five list with 7 wins. Liverpool's passionate fan base and strong performances at Anfield contribute to their success on home soil.

![](https://assets.goal.com/images/v3/blte3bcf29c9f9d32b2/Champions_League_trophy.png) 


These analyses provide valuable insights into various aspects of UEFA Champions League matches, aiding in strategic planning, performance evaluation, and understanding match dynamics.
