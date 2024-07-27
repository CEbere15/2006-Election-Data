# 2006 Congressional Election Data Analysis


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


### Exploratory Data Analysis

EDA for getting a better feel for the datasets, 

```sql
SELECT state, race, SUM(votes) as total_votes
FROM 2006_nominees
GROUP BY state, race
ORDER BY total_votes DESC;
