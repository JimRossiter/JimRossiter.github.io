# Analysing the Effect of Social Media on Mental Health:
*03 May 2024*


![](https://images.stockcake.com/public/2/5/8/258f6d72-bcd2-4950-8fe1-28ff941f261f_large/social-media-analysis-stockcake.jpg)


## Background
I sourced this dataset from Kaggle, accessible [here](https://www.kaggle.com/datasets/shabdamocharla/social-media-mental-health). As an educator, I've observed a significant surge in phone usage and social media engagement. This piqued my interest, prompting an in-depth exploration into the quantitative impacts of these platforms on users. This dataset offered an ample opportunity to analyze and understand these effects.

## Exploratory Data Analysis
My first task was to again take in the data and get an overall feel for it. It is important for me that I familiarise myself with the data and its labels so that I am not misinterpreting any information at a later point. This would take away any credibility from my findings. 

It is worth mentioning the Likert Scale is employed for this trial of 479 participants. The Likert Scale serves as a measurement tool for assessing various mental health effects, including depression levels. For instance, participants may rate their depression experiences on a scale of 1 to 5, where 1 signifies minimal depression and 5 indicates severe depression. This structured approach allows us to quantitatively analyze not only depression but also other variables such as anxiety, restlessness, and concentration difficulty. By utilizing this standardized scale across different constructs, we can capture and compare participants' feelings, allowing for accurate insights into mental health within our dataset.

```sql
-- Inspecting the table for a broad view
SELECT*
FROM smmh;


-- 1: AVERAGE ANXIETY BY GENDER
SELECT Gender, ROUND(AVG(Anxiety),2) as avg_anxiety
FROM smmh
GROUP BY Gender
ORDER BY avg_anxiety DESC;
```
When examining average anxiety scores, it's crucial to consider context. For instance, transgender respondents showed a maximum average score of 5, but this statistic is heavily influenced by the small sample size of only one participant. Similarly, non-binary respondents, comprising only four individuals, can also skew the results. While these scores shouldn't be discounted, a more accurate reflection requires data collection from a more substantial sample group. Interestingly, female and male respondents demonstrated comparable scores in this analysis.

```sql
-- 2: RELATIONSHIP STATUS & ANXIETY
SELECT RelationshipStatus, ROUND(AVG(ANXIETY),2) AS avg_anxiety
FROM smmh
GROUP BY RelationshipStatus
ORDER BY avg_anxiety DESC;
```
Single individuals exhibited the highest average anxiety score at 3.79, whereas married respondents reported the lowest average of 2.85. This disparity could be attributed to factors such as age, with married individuals generally being older and potentially more resilient. However, an alternative perspective suggests that single individuals may experience heightened anxiety due to various factors, such as societal pressures, loneliness, or uncertainty about the future. Further exploration is warranted.

```sql
-- 3: AGE & DEPRESSION
ALTER TABLE smmh
CHANGE COLUMN `ï»¿Age` Age INT;
```
**Before looking at age, I altered the table. The original column name wasn't the most reader- or writer-friendly.**
```sql
SELECT MIN(Age), MAX(Age)
FROM smmh;

SELECT
    CASE
        WHEN AGE BETWEEN 10 AND 20 THEN '10-20'
        WHEN AGE BETWEEN 20 AND 30 THEN '20-30'
        WHEN AGE BETWEEN 30 AND 40 THEN '30-40'
        WHEN AGE BETWEEN 40 AND 50 THEN '40-50'
        WHEN AGE BETWEEN 50 AND 60 THEN '50-60'
        WHEN AGE BETWEEN 60 AND 70 THEN '60-70'
        WHEN AGE BETWEEN 70 AND 80 THEN '70-80'
        WHEN AGE BETWEEN 80 AND 90 THEN '80-90'
        ELSE '90 and above'
    END AS age_group,
    AVG(DEPRESSION) AS total
FROM smmh
GROUP BY age_group
ORDER BY total DESC;
```
Understanding the dataset nuances is crucial for accurate interpretation. While a single respondent over 90 with a maximum depression score of 5 skewed our results, excluding this outlier revealed insightful trends. Teenagers and those in their 20s exhibited the highest depression scores, averaging 3.39 and 3.54, respectively. This aligns with expectations, considering the tumultuous nature of this life stage, societal pressures, and future uncertainties. Conversely, the 50-60 age group displayed the lowest depression scores, potentially reflecting their accquired resilience. Notably, despite their active engagement with an average of 2.88 apps, they maintained positive mental health. These findings underscore the multifaceted relationship between age, mental health, and digital engagement, warranting further exploration.


![](https://images.stockcake.com/public/f/0/8/f0825676-e186-4225-a5e2-35154161cd2e_large/joyful-social-browsing-stockcake.jpg)

```sql
-- 4: NUMBER OF PLATFORMS USED & SLEEP QUALITY
SELECT NumberofSocialMediaPlatforms, AVG(Sleeplessness) AS avg_sleeplessness
FROM smmh
GROUP BY NumberofSocialMediaPlatforms
ORDER BY avg_sleeplessness DESC;
```
The rationale behind this specific analysis was straightforward: examining our habits of using smartphones before bedtime. My hypothesis was that individuals with a higher number of social media applications would tend to stay up later engaging with them. As anticipated, having between 6 and 9 active social media platforms proved to have a negative impact on sleep quality. The average scores for sleeplessness in these groups ranged from 3.5 to 3.9. Conversely, individuals with fewer apps experienced better sleep, with average sleeplessness scores ranging from 2.7 to 3.1.
```sql    
-- 5: ORGANIZATION TYPE AND LEVEL OF DISTRACTION
SELECT Organization, AVG(Distraction) AS avg_dist
FROM smmh
GROUP BY Organization
ORDER BY avg_dist DESC;
```
Likewise, my approach to this analysis was informed by practical experience. I speculated that within various organizations, some would naturally afford more leisure time for individuals to log onto their apps and become distracted. Conversely, workplaces typically uphold codes of conduct and a sense of professionalism, leading me to hypothesize that levels of distraction would be lower in these settings. My hypothesis was validated, as individuals in school and university reported higher levels of distraction, with scores of 3 and 3.5, respectively. This stands in stark contrast to those employed in government positions, who reported an average distraction score of 2.2.
```sql
-- 6: NUMBER OF SOCIAL MEDIA APPS & RELATIONSHIP STATUS
SELECT RelationshipStatus, AVG(NumberofSocialMediaPlatforms) AS avg_apps
FROM smmh
GROUP BY RelationshipStatus
ORDER BY avg_apps DESC;
``` 
