# 2006 Congressional Election Data Analysis


### Summary

This data project was made with the goal of gaining insights into the performances of each party, nominee in the 2006 Congressional Elections, depending on their state, region, incumbency, gender, the type of congressional seat being contested (Senate, House of Representatives, Delegate), and district partisanship. Through analyzing each nominee, in each race, one is able to identify the overall trends that the country had when it came to voting in these elections.

### Data Sources

Nominee Data: The primary dataset used for the analysis is in the '2006_nominees.csv' file, which has all the nominees for the congressional elections 


```sql
SELECT state, race, SUM(votes) as total_votes
FROM 2006_nominees
GROUP BY state, race
ORDER BY total_votes DESC;
