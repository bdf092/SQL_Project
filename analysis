-- PART I: SCHOOL ANALYSIS
-- 1. View the schools and school details tables
USE maven_advanced_sql;
SELECT * FROM schools;
SELECT * FROM school_details;
SELECT * FROM players;
SELECT * FROM salaries;

-- 2. In each decade, how many schools were there that produced players?

SELECT  FLOOR(yearID/10) * 10 AS decade, COUNT(DISTINCT schoolID) AS num_schools
FROM schools
GROUP BY  decade
ORDER BY decade;



-- 3. What are the names of the top 5 schools that produced the most players?


SELECT sd.name_full, COUNT(distinct s.playerID) AS num_players
FROM schools s LEFT JOIN school_details sd
	 ON s.schoolID = sd.schoolId
GROUP BY sd.schoolID
ORDER BY num_players DESC
LIMIT 5;


SELECT sd.name_full, COUNT(distinct s.playerID) AS num_players
FROM schools s LEFT JOIN school_details sd
	 ON s.schoolID = sd.schoolId
GROUP BY sd.name_full
ORDER BY num_players DESC
LIMIT 5;
-- 4. For each decade, what were the names of the top 3 schools that produced the most players?


WITH ds AS ( SELECT FLOOR(s.yearID/10) * 10 AS decade, sd.name_full,
						COUNT(distinct s.playerID) AS num_players
				FROM schools s LEFT JOIN school_details sd
					 ON s.schoolID = sd.schoolId
				GROUP BY decade, s.schoolID),
	ranking AS (SELECT decade, name_full, num_players,
					   ROW_NUMBER() OVER(PARTITION BY decade ORDER BY num_players DESC) AS row_num
				FROM ds)

SELECT decade, name_full, num_players 
FROM ranking
WHERE row_num <= 3
ORDER BY decade DESC, row_num;

-- PART II: SALARY ANALYSIS
-- 1. View the salaries table


SELECT *
FROM salaries;
-- 2. Return the top 20% of teams in terms of average annual spending

WITH ans AS (SELECT teamID, yearID, SUM(salary) AS annual_spending
			FROM salaries
			GROUP BY teamID, yearID),
	pct AS (SELECT teamID,
			AVG(annual_spending) AS avg_spend,
			NTILE(5) OVER(ORDER BY AVG(annual_spending) DESC) AS pct
			FROM ans
			GROUP BY teamID)

SELECT teamID, avg_spend
FROM pct
WHERE pct = 1;

WITH ts AS (SELECT teamID, yearID, sum(salary) AS total_spend
			FROM salaries
			GROUP BY teamID, yearID
            ORDER BY teamID, yearID),
	 sp AS (SELECT teamID, avg(total_spend) AS avg_spend,
					NTILE(5) OVER(ORDER BY avg(total_spend) DESC) AS spend_pct
			FROM ts
			GROUP BY teamID)

SELECT teamID, ROUND(avg_spend/1000000,1) AS avg_spend_millions
FROM sp
WHERE spend_pct = 1;
            
            

-- 3. For each team, show the cumulative sum of spending over the years

WITH yp AS (SELECT teamID, yearID, SUM(salary) AS year_spent
			FROM salaries
			GROUP BY teamID, yearID)
            
SELECT teamID, yearID,
		ROUND(SUM(year_spent) OVER(PARTITION BY teamID ORDER BY yearID)/1000000,1) AS cummulative_spend_millions
FROM yp;


WITH ts AS (SELECT teamID, yearID, SUM(salary) AS total_spend
			FROM salaries
			GROUP BY teamID, yearID
			ORDER BY teamID, yearID)

SELECT teamID, yearID,
		ROUND(SUM(total_spend) OVER (PARTITION BY teamID ORDER BY yearID)/1000000,1)
        AS cumulative_sum_millions
FROM ts;




-- 4. Return the first year that each team's cumulative spending surpassed 1 billion

WITH yp AS (SELECT teamID, yearID, SUM(salary) AS year_spent
			FROM salaries
			GROUP BY teamID, yearID),            
		cs AS (SELECT teamID, yearID,
						SUM(year_spent) OVER(PARTITION BY teamID ORDER BY yearID) AS cummulative_spend_millions
				FROM yp),
		rn AS (SELECT teamID, yearID, cummulative_spend_millions,
				   ROW_NUMBER() OVER(PARTITION BY teamID ORDER BY cummulative_spend_millions) AS ranking
			FROM cs
            WHERE cummulative_spend_millions >  1000000000)
            
SELECT teamID, yearID, cummulative_spend_millions 
FROM rn
WHERE ranking = 1;      




WITH ts AS (SELECT teamID, yearID, SUM(salary) AS total_spend
			FROM salaries
			GROUP BY teamID, yearID
			ORDER BY teamID, yearID),
	  cs AS (SELECT teamID, yearID,
						SUM(total_spend) OVER (PARTITION BY teamID ORDER BY yearID)
						AS cumulative_sum
				FROM ts),
                
		rn AS (

				SELECT teamID, yearID, cumulative_sum,
						ROW_NUMBER() OVER(PARTITION BY teamID ORDER BY cumulative_sum) AS rn
				FROM cs
				WHERE cumulative_sum > 1000000000)


SELECT teamID, yearID, ROUND(cumulative_sum/1000000000, 2) AS cumulative_sum_billions
FROM rn
WHERE rn = 1;





 

-- PART III: PLAYER CAREER ANALYSIS
-- 1. View the players table and find the number of players in the table

SELECT COUNT(DISTINCT playerID) AS num_players
FROM players;

SELECT * FROM players;
-- 2. For each player, calculate their age at their first game, their last game, and their career length (all in years). Sort from longest career to shortest career.

SELECT playerID, nameGiven, year(debut) - birthYear	AS age_first_game,
		year(finalGame) - birthYear	AS age_last_game,
		year(finalGame) - year(debut) AS career_length_years
FROM players
ORDER BY career_length_years DESC;


SELECT nameGiven,
       TIMESTAMPDIFF(YEAR, CAST(CONCAT(birthYear, '-', birthMonth, '-', birthDay) AS DATE),debut) AS age_first_game, 
       TIMESTAMPDIFF(YEAR, CAST(CONCAT(birthYear, '-', birthMonth, '-', birthDay) AS DATE),finalGame) AS age_last_game, 
	   TIMESTAMPDIFF(YEAR, debut, finalGame) AS career_length 
FROM  
     players
ORDER BY career_length DESC;


-- 3. What team did each player play on for their starting and ending years?

SELECT  p.nameGiven, 
		s.yearID AS starting_year , s.teamID AS starting_team,
        e.yearID  AS ending_year , e.teamID AS ending_team
FROM players p INNER JOIN salaries s
					 ON p.playerId = s.playerID
					 AND YEAR(p.debut) = s.yearID
			   INNER JOIN salaries e
                     ON p.playerId = e.playerID
					 AND YEAR(p.finalGame) = e.yearID;
                     
SELECT 	p.nameGiven,
		s.yearID AS starting_year, s.teamID AS starting_team,
        e.yearID AS ending_year, e.teamID AS ending_team
FROM	players p INNER JOIN salaries s
							ON p.playerID = s.playerID
							AND YEAR(p.debut) = s.yearID
				  INNER JOIN salaries e
							ON p.playerID = e.playerID
							AND YEAR(p.finalGame) = e.yearID;
                     

SELECT * FROM salaries;
SELECT * FROM players;
-- 4. How many players started and ended on the same team and also played for over a decade?

WITH tse AS (SELECT  p.nameGiven, 
						s.yearID AS starting_year , s.teamID AS starting_team,
						e.yearID  AS ending_year , e.teamID AS ending_team
				FROM players p INNER JOIN salaries s
									 ON p.playerId = s.playerID
									 AND YEAR(p.debut) = s.yearID
							   INNER JOIN salaries e
									 ON p.playerId = e.playerID
									 AND YEAR(p.finalGame) = e.yearID),
	pse AS (SELECT starting_team, ending_team, COUNT(nameGiven) AS num_players, 
				   ending_year - starting_year AS career_length
			FROM tse
			GROUP BY starting_team, ending_team, ending_year, starting_year
			HAVING starting_team = ending_team)

SELECT starting_team,  ending_team, num_players
FROM pse
WHERE career_length > 10;
                                     
                                     
                                     




-- PART IV: PLAYER COMPARISON ANALYSIS
-- 1. View the players table

SELECT * FROM players;
-- 2. Which players have the same birthday?

SELECT nameGiven, 
       CAST(CONCAT(birthYear, '-', birthMonth, '-', birthDay) AS DATE) AS birthday_date 
FROM players;


WITH bt AS (SELECT nameGiven, 
       CAST(CONCAT(birthYear, '-', birthMonth, '-', birthDay) AS DATE) AS birthday_date 
FROM players)

SELECT birthday_date, 
       GROUP_CONCAT(nameGiven  SEPARATOR ', ') AS list_birthday
FROM bt
WHERE	YEAR(birthday_date) BETWEEN 1980 AND 1990
GROUP BY birthday_date
ORDER BY birthday_date;

-- 3. Create a summary table that shows for each team, what percent of players bat right, left and both

SELECT  s.teamID,
	    ROUND(SUM(CASE WHEN p.bats = 'R' THEN 1 ELSE 0 END)/COUNT(s.playerID)*100, 1) AS right_bat,
        ROUND(SUM(CASE WHEN p.bats = 'L' THEN 1 ELSE 0 END)/COUNT(s.playerID)*100, 1)  AS left_bat,
        ROUND(SUM(CASE WHEN p.bats = 'B' THEN 1 ELSE 0 END)/COUNT(s.playerID)*100, 1)  AS both_bat
FROM players p LEFT JOIN salaries s
    ON p.playerID = s.playerID
GROUP BY s.teamID;


SELECT	s.teamID,
		ROUND(SUM(CASE WHEN p.bats = 'R' THEN 1 ELSE 0 END) / COUNT(s.playerID) * 100, 1) AS bats_right,
        ROUND(SUM(CASE WHEN p.bats = 'L' THEN 1 ELSE 0 END) / COUNT(s.playerID) * 100, 1) AS bats_left,
        ROUND(SUM(CASE WHEN p.bats = 'B' THEN 1 ELSE 0 END) / COUNT(s.playerID) * 100, 1) AS bats_both
FROM	salaries s LEFT JOIN players p
		ON s.playerID = p.playerID
GROUP BY s.teamID;

SELECT teamID, playerID
FROM salaries;
-- 4. How have average height and weight at debut game changed over the years, and what's the decade-over-decade difference?

WITH ds AS (SELECT
		FLOOR(YEAR(debut)/10)*10 AS decade, 
		AVG(height) AS avg_height, 
        AVG(weight) AS avg_weight
FROM players
GROUP BY decade)

SELECT decade,
		avg_height - LAG(avg_height) OVER(ORDER BY decade) AS height_change,
        avg_weight - LAG(avg_weight) OVER(ORDER BY decade) AS weight_change
FROM ds
WHERE decade IS NOT NULL;
