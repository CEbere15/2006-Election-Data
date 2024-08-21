# 2006 Congressional Election Data Analysis

### Table of Contents

1. [Overview](#overview)
2. [Summary](#summary)
3. [Data Sources](#data-sources)
4. [Tools](#tools)
5. [Dataset Making Process](#dataset-making-process)
   - [Sources](#sources)
6. [Data Manipulation](#data-manipulation)
   - [Categorizing Parties and Ranking Nominees](#categorizing-parties-and-ranking-nominees)
   - [Races and Margins](#races-and-margins)
   - [Joining with the Incumbent Data](#joining-with-the-incumbent-data)
7. [Exploratory Data Analysis](#exploratory-data-analysis)
   - [SQL](#sql)
     - [Election Data](#election-data)
     - [Incumbent Data](#incumbent-data)
   - [Python](#python)
8. [Data Analysis](#data-analysis)

## Overview

### Summary

This data project was made with the goal of gaining insights into the performances of each party, nominee in the 2006 Congressional Elections, depending on their state, region, incumbency, gender, the type of congressional seat being contested (Senate, House of Representatives, Delegate), and district partisanship. Through analyzing each nominee, in each race, one is able to identify the overall trends that the country had when it came to voting in these elections.


### Data Sources

Nominee Data: The primary dataset used for the analysis is in the '2006_nominees.csv' file, which has all the nominees for the congressional elections, whether they are the incumbent, the race that they participated in, their vote share, along with information on which party they are affiliated with, what party the seat their contesting belongs to, and more.

Incumbent Data: A secondary dataset that for the analysis is in 'incumbent_data.csv' file, which shows who the most recent seat holder is for each race, when their term started, their decision in the upcoming election, and the partisan ratings for their contest. 


### Tools
- Excel - Data Cleaning / Preperation
  - Used to compile datasets from multiple sources and organize them in worksheets.
- DB Browser (SQLite) - Data Analysis, Exploration and Manipulation
  - Used for querying, exploring, and manipulating the data into a more usable format for insights.
- Python - Data Analysis and Visualization
  - Employed to create charts and visualizations that would be challenging to produce in Tableau, along with performing statistical analyses to understand how variables might have affected the number of votes.
- Tableau - Data Visualization
  - Utilized for visualizing the dataset.


### Dataset Making Process

The steps taken for making the dataset included:
  1. Selecting all of the variables that were needed for the dataset.
  2. Utilizing the sources to fill in the variables.
  3. Doing further inspection on the nominee or race where data might be missing from the sources.

#### Sources:
- [United States House of Representatives History - Election Statistics (2006)](https://history.house.gov/Institution/Election-Statistics/Election-Statistics/)
- [Federal Election Commission - 2006 Federal Elections](https://www.fec.gov/resources/cms-content/documents/federalelections2006.pdf)
- [Wikipedia - 2006 House of Representatives Elections](https://en.wikipedia.org/wiki/2006_United_States_House_of_Representatives_elections)
- [Wikipedia - 2006 Senate Elections](https://en.wikipedia.org/wiki/2006_United_States_Senate_elections)
- [Our Campaigns](https://www.ourcampaigns.com/)
- [Votesmart - 2006 Congressional General Election Nationwide Candidates](https://justfacts.votesmart.org/election/2006/C/NA/2006-congressional-election?stageId=G&p=1)
- [Library of Congress](https://www.loc.gov/)


### Data Manipulation

#### Categorizing Parties and Ranking Nominees

This query creates a new table for manipulating the data, creating two new values: Category and Standing. The Category value is for grouping the Party value into smaller groups, and Standing is a ranking of each candidate by the race they participated in.


```sql
-- Creating a table with the new values with the original data
CREATE Table Manip AS 
SELECT Nominee, Party, Gender, State, Region, WiderRegion, Type, Race, Class, Incumbent, Victor, Votes, Share, Total, 
    dense_rank() OVER (PARTITION BY Race ORDER BY Votes DESC) AS Standing, Heldby, Hometown, EDay, EYear, YearType, Elast, 
    Final, CASE
        WHEN Party IN ('Democratic', 'Democratic-Farmer-Labor', 'Democratic-Nonpartisan League', 'Democratic (Write-In)') THEN 'Democratic Party'
        WHEN Party IN ('Republican', 'Republican (Write-In)') THEN 'Republican Party'
        WHEN Party IN ('Free Libertarian', 'Libertarian') THEN 'Libertarian Party'
        WHEN Party in ('Reform') then 'Reform Party'
        WHEN Party in ('Constitution', 'American Independent', 'American Constitution Party', 'U.S. Taxpayers', 'Independent American') THEN 'Constitution Party'
        WHEN Party in ('Green', 'Mountain', 'Pacific Green', 'Independent Green', 'Desert Greens') then 'Green Party'
        WHEN Party = 'Conservative' THEN 'Conservative Party'
        WHEN Party IN ('Young Socialist Alliance', 'Communist', 'Socialist Equality', 'Socialist', 'Socialist Labor', 'Peace And Freedom', 'Socialist Workers', 'Liberty Union') THEN 'Socialist Parties'
        WHEN Party IN ('No Party', 'No Party Affiliation', 'Other', 'Nominated by Petition', 'Conneticut for Lieberman', 'Write-In', 'Independent Political Choice', 'No Party Preference', 'Independent', 'Independent Constitutional Candidate', 'Independent Party of Delaware', 'Independence', 'Nonpartisan', 'No Political Party', 'Unaffiliated') THEN 'Independent'
        ELSE 'Other Minor Parties'
    END AS Category, Special, Runoff 
FROM '2006_nominees' 
ORDER BY Race, State
```

#### Races and Margins

The first three queries selects the first and second place nominees from each race, and uses them to find the margin for in said race; the final query joins the main manipulated table with the table made for the margin for each race, and makes numbers them in rows by State, Race and Total Votes

```sql
-- All the first place nominees per race
Create Table NRank1 AS SELECT * FROM Manip WHERE Standing = 1;


-- All the second place nominees per race
Create Table NRank2 AS SELECT * FROM Manip WHERE Standing = 2;

-- Making the margins
Create Table Marg AS SELECT a.Race,  a.Share - b.Share AS Win
FROM NRank1 a 
JOIN NRank2 b ON a.Race = b.Race;

-- Joining the tables to the rest of the manipulation table
Create Table Margined as 
SELECT row_number() over(order by State, a.Race, Votes desc) Row, Nominee, Party, Gender, State, Region, WiderRegion, Type, a.Race, Class, 
Incumbent, Victor, Votes, Share, Total, Standing, HeldBy, Hometown, printf('%.2f%%', b.Win) AS Margin, Category, EDay, EYear, YearType, Elast, 
Final, Special, Runoff
FROM Manip a
LEFT JOIN Marg b
ON b.Race = a.Race;
```

#### Joining with the Incumbent Data

Takes the margined table and joins it with the 'incumbent_data' to add the last incumbent, their decision, gender and their tenure to the margined table

```sql
-- Joins values from both tables
Create Table FullManipulation as Select Row, Nominee, Party, Gender, State, Region, WiderRegion, Type, a.Race, Class, 
Incumbent, Victor, Votes, Share, Total, Standing, HeldBy, Hometown, Margin, Category, EDay, EYear,
YearType, ELast, Final, Special, Runoff, SeatHolder, HolderGender, TermStart, Decision, Days, Rating,
PVI, PVIR
FROM Margined a
left join incumbent_data b
on a.Race = b.Race;
```
### Exploratory Data Analysis

#### SQL


##### Election Data
EDA for getting a better feel for the datasets:


###### Query

  1. How many different states and territories were represented in the House, Senate and Delegate elections?
```sql
Select Type, count(distinct State) from '2006_nominees'
group by Type; 
```

###### Visualization





  2. How many parties had nominees in the Congressional Elections?

     ###### Query
   ```sql
-- Amount of parties that had nominees in the 2006 Congressional Elections
Select Type, count(distinct Party) as Parties from '2006_nominees'
group by type;
```


###### Visualization


   ![image](https://github.com/user-attachments/assets/8d2bfcfa-0c70-482c-b45e-84627ea15c71)


 3. What's the ratio for gender of nominees in each region the race takes place, and by the type of election?

###### Query
```sql
-- Gender distribution of nominees by the type of election and the region
Select Type, Region, Gender, count(distinct Nominee) as Nominees, 
printf('%.2f%%', ROUND(COUNT(DISTINCT Nominee) * 100.0 / SUM(COUNT(DISTINCT Nominee)) 
OVER (PARTITION BY Type, Region), 2)) AS 'Nominee Percent' from '2006_nominees'
group by Type, Region, Gender
order by Type desc, Region, Nominees desc;
```

###### Visualization
![image](https://github.com/user-attachments/assets/cab8a479-b47d-41cc-b8fa-27cceb319ff6)



  4. Which parties had incumbents lose their reelection bids in the general election, and how many did they lose?
###### Query

```sql
-- Amount of Incumbents who lost reelection by Chamber
Select Type, Party, count(*) as 'Incumbents Lost' from '2006_nominees' 
where Incumbent = TRUE and Victor = FALSE
group by Type, Party;
```


###### Visualization

![image](https://github.com/user-attachments/assets/584df5bb-e63e-414d-be33-dddcbaad02c4)


  5. How many seats did each party hold from the last Congress, in each Chamber?


###### Query

```sql
-- Party strength of previous Congress by chamber
Select Type, HeldBy, Count(*) as Seats from '2006_nominees'
where Final = TRUE and Special = FALSE and Victor = TRUE
group by Type, HeldBy;

```

###### Visualization

![image](https://github.com/user-attachments/assets/15afaa26-df7f-40d2-b5e9-d2da4fa5148a)


  6. How many seats did each party get after these elections?
###### Query

```sql
-- Party strength for the next Congress by chamber
Select Type, Party, Count(*) as Seats from '2006_nominees'
where Final = TRUE and Special = FALSE and Victor = TRUE
group by Type, Party;

```


###### Visualization


![image](https://github.com/user-attachments/assets/f42377a1-44b8-48dd-89ab-07e2a4440ec7)





   7. What census division does each state belong to?

###### Query


```sql
-- Each state's census division
Select Region, State from '2006_nominees'
group by State;
```


###### Visualization

![image](https://github.com/user-attachments/assets/c7770b2c-e94d-4301-a120-84a3d02392c6)


   8. What census region does each state belong to?

###### Query


```sql
-- Each state's census region
Select WiderRegion, State from '2006_nominees'
group by State;
```


###### Visualization

![image](https://github.com/user-attachments/assets/affe6e0e-3b28-457d-aa0a-7ce234ced927)

![image](https://github.com/user-attachments/assets/76822fe3-eb21-4b6e-a77d-ef738084f006)



##### Incumbent Data 
  1. What is the average tenure of Senators up for reelection by the choice and outcome they made for reelection?

```sql
-- Average years since the last incumbent Senator was sworn in to their term before Election Day
Select Case when Decision in ('Reelection (Lost Renomination)', 'Reelection (Lost Renomination; Changed Parties)') then 'Lost Renomination'
when Decision in ('Resignation (Charges)','Resignation','Resignation (Appointment)') then 'Resigned'
when Decision in ('Retired (Higher Office)','Retired') then 'Retired'
when Decision in ('Reelection (Renominated)') then 'Renominated'
else 'Death' end as Choice, round(avg(days) / 365.25, 2) as 'Average Tenure' from incumbent_data
where Race like '%Sen%' and Race not like '%Special%' and Race not like '%Runoff%'
group by Choice;
```

     
  2. What is the average tenure of Representatives/Delegates by the choice and outcome they made for reelection?

  ```sql
-- Average years since the last incumbent Representative or Delegate was sworn in to their term before Election Day
Select Case when Decision in ('Reelection (Lost Renomination)') then 'Lost Renomination'
when Decision in ('Resignation (Charges)','Resignation','Resignation (Appointment)') then 'Resigned'
when Decision in ('Retired (Higher Office)','Retired') then 'Retired'
when Decision in ('Reelection (Renominated)') then 'Renominated'
else 'Death' end as Choice, round(avg(days) / 365.25, 2) as 'Average Tenure' from incumbent_data
where Race not like '%Sen%' and Race not like '%Special%' and Race not like '%Runoff%'
group by Choice;
```

  3. How many of the incumbents from either chamber were elected by special election or appointed and which were elected normally to their seat?
     
```sql    
-- List of incumbents elected normally and the amount elected by Special Election in the House & Senate
Select Case when Race like '%Sen%' then 'Senate'
else 'House' end as Chamber, Case when TermStart like '%01-03%' then 'Elected Normally'
else 'Elected by Special Election / Appointed' end as Electeds, Count(*)
from incumbent_data
where Race not like '%Special%' and Race not like '%Runoff%'
group by Chamber, Electeds;

```
  4. What's the percentage of men and women for each chamber when it comes to incumbents?
  ```sql
    -- Calculating the amount of men and women incumbents whose seats are up for reelection and then showing the percentage of them
    WITH DistinctSeatHolders AS (
    SELECT CASE WHEN Race LIKE '%Sen%' THEN 'Senate' ELSE 'House' END AS Chamber, HolderGender AS Sex, COUNT(DISTINCT SeatHolder) AS DistinctSeatHolders
    FROM incumbent_data
    WHERE Race NOT LIKE '%Special%' AND Race NOT LIKE '%Runoff%'
    GROUP BY CASE 
            WHEN Race LIKE '%Sen%' THEN 'Senate'
            ELSE 'House'
        END, HolderGender
)

SELECT Chamber, Sex, DistinctSeatHolders,
printf('%.2f%%', DistinctSeatHolders * 100.0 / SUM(DistinctSeatHolders) OVER (PARTITION BY Chamber)) AS Percent
FROM DistinctSeatHolders;


```
  5. For each Cook Rating, how many races were rated that by each Chamber

```sql
-- Select each Cook Rating, and how many races were rated that way
Select CASE WHEN Race LIKE '%Sen%' THEN 'Senate' ELSE 'House' END AS Chamber, Rating, Count(*) as Races
from incumbent_data
where Race not LIKE '%Special%' AND Race NOT LIKE '%Runoff%' 
group by Chamber, Rating
order by Chamber, Races desc;
```

#### Python
```py
```

### Data Analysis


```sql
SELECT state, race, SUM(votes) as total_votes
FROM 2006_nominees
GROUP BY state, race
ORDER BY total_votes DESC;
