# Moneyball Project: An Analysis of Efficiency in Baseball Teams' Spending

In this project, we examine Major League Baseball data to understand the efficiency of teams in terms of their spending and the resulting wins. Our particular interest lies in the Oakland A's strategy, popularly known as "Moneyball", which fundamentally redefined player valuation to exploit market inefficiencies.

## Background

Major League Baseball is a professional baseball league where teams pay players to play baseball. The goal of each team is to win as many games out of a 162 game season as possible. Teams win games by scoring more runs than their adversary. In principle, better players are costlier, so teams that want good players need to spend more money. Teams that spend the most, frequently win the most.

So, how can a team that can't spend as much, win? The Oakland A's approach, termed "Moneyball", was to redefine what makes a player good, i.e., figure out what player characteristics translated into wins. They realized that teams were not really pricing players using these characteristics, which allowed them to pay for undervalued players, players that were good according to their metrics, but were not recognized as such by other teams, and therefore not as expensive.

## Data

The data for this analysis comes from the Lahman's Baseball Database, specifically the 'Salaries' and 'Teams' tables. The database provides comprehensive data on baseball teams, players and seasons, and is available as a `sqlite` database.

In this project, we examine:
- Total payroll and winning percentage for each team over time (from 1990 to 2014)
- The distribution of payrolls across teams conditioned on time
- Correlation between payroll and winning percentage
- Standardization of payrolls across years to compare them more effectively
- Calculation of expected win percentage based on standardized payroll
- A new measure of each team's spending efficiency based on their winning percentage and their expected winning percentage

## Key Findings

- The Oakland A's efficiency during the "Moneyball" period is particularly notable. Despite spending less compared to other teams, they achieved a high win percentage, indicating a high level of efficiency in their spending.
- Over time, there is a positive correlation between team payrolls and winning percentage. However, the strength of this correlation varies across different time periods.
- The efficiency of teams' spending varies considerably. Some teams achieve high win percentages with relatively low payrolls, demonstrating high efficiency. In contrast, other teams with high payrolls do not achieve proportionately high win percentages, indicating lower efficiency.

This analysis provides a fascinating insight into the strategy and performance of baseball teams, highlighting the potential for data-driven decision making in sports.

This project's Jupyter notebook provides the detailed analysis, including all data wrangling, exploratory data analysis, and visualizations.

## Code snippets
Here are some snippets from the project:

```
# Connect to the sqlite database and fetch data
import sqlite3
import pandas as pd

sqlite_file = 'lahman2014.sqlite'
conn = sqlite3.connect(sqlite_file)

# Compute total payroll and winning percentage for each team
query = "SELECT teamID, yearID, sum(salary) as total_payroll, W as wins, G as games, ((W*100) / (G)) as winnings, franchID FROM Salaries JOIN Teams ON Salaries.yearID=Teams.yearID AND Salaries.teamID=Teams.teamID WHERE Salaries.yearID >= 1990 GROUP BY Salaries.teamID, Salaries.yearID ORDER BY Salaries.teamID"
team_data = pd.read_sql(query, conn)

# Discretize year into five time periods and plot mean winning percentage vs mean payroll

team_data['period'] = pd.cut(team_data['yearID'], bins=range(1990, 2016, 5), labels=['1990-1994', '1995-1999', '2000-2004', '2005-2009', '2010-2014'])
periods = ['1990-1994', '1995-1999', '2000-2004', '2005-2009', '2010-2014']

import matplotlib.pyplot as plt

for period in periods:
    subset = team_data[team_data['period'] == period]
    avg_payroll = subset.groupby('teamID')['total_payroll'].mean()
    avg_winning_percentage = subset.groupby('teamID')['winnings'].mean()

    plt.figure(figsize=(10,6))
    plt.scatter(avg_payroll, avg_winning_percentage)
    plt.title(f'Mean Payroll vs Winning Percentage ({period})')
    plt.xlabel('Mean Payroll')
    plt.ylabel('Mean Winning Percentage')
    plt.grid(True)
    plt.show()
```

This code creates scatter plots for mean payroll and winning percentage for each time period. It's clear that there is a positive correlation between the amount a team spends on salaries and the percentage of games it wins, but the correlation isn't perfect, indicating that spending efficiency can make a significant difference.

To further analyze efficiency, we can calculate the expected win percentage for each team based on their payroll, and then determine their spending efficiency as the ratio of their actual win percentage to their expected win percentage.

```
from sklearn.linear_model import LinearRegression

# Compute standardized payroll and expected win percentage for each team
for period in periods:
    subset = team_data[team_data['period'] == period]
    avg_payroll = subset.groupby('teamID')['total_payroll'].mean().values.reshape(-1, 1)
    avg_winning_percentage = subset.groupby('teamID')['winnings'].mean().values
    
    # Fit a linear regression model
    model = LinearRegression()
    model.fit(avg_payroll, avg_winning_percentage)
    
    # Calculate expected win percentage and efficiency
    subset['expected_winning_percentage'] = model.predict(avg_payroll)
    subset['efficiency'] = subset['winnings'] / subset['expected_winning_percentage']
```

This code computes the expected winning percentage based on a linear regression model trained on the average payroll data, and then calculates the efficiency as the ratio of the actual winning percentage to the expected winning percentage. Teams with efficiencies greater than 1 are more efficient in their spending than average, while those with efficiencies less than 1 are less efficient.

## Conclusion

In conclusion, it's clear that while there is a positive correlation between a team's payroll and its winning percentage, the relationship is not perfectly linear. Other factors play a significant role in a team's success. For example, how efficiently a team uses its payroll can significantly influence its performance. This analysis is just one way to quantify and visualize these relationships.

Further research could delve deeper into what factors contribute to a team's efficiency, such as management strategy, player development, or recruitment practices. The impact of other variables, such as changes in team or league policies, the introduction of new technologies, or socio-economic factors could also be interesting to explore.

Baseball, like many sports, is a complex system with many interdependent variables that can influence outcomes. By using data analysis tools and techniques, we can start to untangle these relationships and gain a deeper understanding of the game.
