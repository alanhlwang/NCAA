# NCAA

Collecting March Madness data to evaluate trends between what defines a winning versus a losing team and introduce a model aimed at predicting the championship every year.

# NCAA March Madness Predictions
Every time March rolls around, the one thing that dominates headlines is March Madness. It might be your overly-crazed sports dad, your rowdy college roommates, or even your unapologetically homer mother who gives you endless updates on the tournament. This intensity comes from uncertainty: March Madness is a single-elimination tournament where 68 of the best college basketball teams across the country go head-to-head. Due to the single-elimination format, upsets are much more common than in the professional equivalents (I'm looking at you, the NBA's 7 game series). I want to take a deep-dive into the March Madness data, evaluate trends between what defines a winning versus a losing team, and introduce a model aimed at predicting the championship every year.
Context
March Madness is the culmination of a nationwide college Division 1 Basketball season. The tournament pits 68 of the best teams in the nation against each other in a single-elimination tournament and most recently has drawn in more than $500 million in revenue for the NCAA. 
The tournament splits teams into four separate regions: the South, West, East, and Midwest.There are a total of 6 rounds, 7 if you include the play-in tournament for the four lowest ranked teams. The tournament itself has existed since the 1930's, where its first game was played in 1939 and UCLA has the most championship wins at 11!
Data Collection
To gather the necessary team and tournament data, I utilized the rvest library in R along with a CSS selector in SelectorGadget to scrape sports-reference for all champions and 68 tournament teams for roughly 15 years of tournament data. Let's run through a quick example.
First, we open up a session in R that directs to sports-reference's postseason statistics with the aptly-named "session" function.
This page has links of every year's March Madness tournament. This is what we'll be using as our base page to iterate through each year's data. So, let's use the year 2019 as an example. 
Here, we follow the link on our base page titled "2019 NCAA Tournament," which then leads us to a link of all the teams' statistics for that season. We choose to follow the link for the 2018–2019 Virginia Cavalier's regular season statistics. 
After storing the html information for the 2018–19 Cavalier's season, we can then use our CSS selector tool to see which node we should extract from. Pictured below is an example of utilizing the CSS selector tool and the output it provides.

Once we have the ideal node to draw our data from, we can then use extract and convert the output from the page using html_text. 
After some data restructuring, we are able to extract the 2018–19 Virginia Cavalier's regular season statistics. So, we automate this process, allowing for a vector of years which you want to draw data from and using their page data to access the teams that made it to the tournament that year.
Once applying this process to all teams from the years we want, I end up with a final list of 15 data frames and 68 rows: 15 for the number of seasons and 68 for the number of teams that participated in the tournament that year. We can subsequently collapse this list into one data frame to run our analysis on.

# Exploratory Data Analysis
One of the first things I wanted to explore was what difference, if any, could we see between the winners of each season and the rest of the teams in terms of their offensive and defensive stats. The eye-test would have you believe that the winners of each season should has the best offense and best defense in the entire nation, so I wanted to see if that was reflected in the data.
I started off with the bread and butter of team statistics: their overall points per game. By creating a time-series plot with the winning team of each year being one line, and the average of the losing teams on the other line, we are able to see clear trends and differences between the two categories. 
PPG by Year (Green = Champions, Red = Rest of League)For the most part, we see what we would expect to see. There are very large differences between the championship teams and the rest of the league in points per game. On average, championship teams score much more than the rest of the league (shocker, I know). It's important to note, however, the few outliers. We can see that in the 2011 and 2014 season, the difference between the two become much smaller. In fact, in 2014, the championship team scored less than the league average. 
Diving into the discrepancies into the 2014 season further, we can see that UConn won the tournament that year with an average PPG of 71, a full 2.5 points less than the rest of the league average. Most notably, this season was the first season in NCAA history where the championship game did not feature any teams in the top 3 seeds. So, we can see that the Cinderella-run of the 2017 UConn Huskies is well-reflected in the data with their relatively less-stellar offense than some other teams. 
We can see similar trends and differences between the championship team and the rest of the league in other offensive metrics like assists per game and field goal percentage below. 

Green = Champions, Red = Rest of League

Moving onto defensive metrics we see more of the same. Below is a boxplot of blocks per game split by championship teams and the rest of the league. 
We can see that the median of the championship team is higher than that of the losing team, but the losing teams had a much higher variance in their values. This suggests that while blocks might be important to a championship team, a team that accumulates many blocks is not necessarily a good team. The high block teams we see in our losing group could have been flawed in other areas such as turnover rate, or assist rate.

# Model Building
For our predictive model, I used Naive Bayes Classifier and Random Forest to attempt to classify a team as one that more closely resembled a championship team of a losing team. 
Starting with our Naive Bayes Classifier, I used our larger dataframe of all championship and losing teams and sampled around two-thirds of the data as training and one-thirds of the data as testing. To keep all of our season data intact, I chose to randomly sample by year and then use that year to decide whether or not a subsection of my data was in the training or testing set. 
Initial forays with Naive Bayes proved to be shaky at best. Here we can see a prediction of winning teams for the 2012, 2013, 2014, and 2016 seasons. I used my model to first generate a group of 68 probability vectors for each season, where the Loser probability would be the chance that the model thought the team was a losing team, and the Winning one the opposite. Then, I selected the highest Winning probability team of that specific team to attempt to see which team had the best chance of winning. Below actually are the top 2 Winning probabilities of each season, as a single point had a much tougher time at getting an accurate prediction. We can see that our initial attempt (with a handicap of 2 guesses) has an accuracy of 50 percent. Not great.
Probability of Winning vs Losing team from Naive Bayes Classifier on Testing Data (2012–2016 seasons)Moving on to our Random Forest approach. After imputing NAs in the Minutes per Game and Games Played columns, I used the RandomForest library in R to predict the championship team of each season. I transformed our testing and training data frames intro matrices, and then created an initial model. I create and plot an initial tree showing the decision making between going from Blocks to then cutoffs for Points and Free Throw Percentage. 
 To fine-tune the mtry parameter, or the number of variables selected at each tree split, I used the "TuneRF" function to run through multiple iterations of the model at different mtrys and then selecting the one with the minimum OOB error. This ended up being 3 variables. 
To gauge the accuracy of our model, I used Leave-One-Out-Cross-Validation to use a single season of data as our testing data and the rest of the seasons as our training data for each season. After looping through each of these different selections of the data, our model ended up with a prediction accuracy of 62%. Again, not amazing, but this falls in line with the 54% to 84% model predictions from the University of New Hampshire's "Predictive Analytics for College Basketball: Using Logistic Regression for Determining the Outcome of a Game."
Analyzing the variance importance of our Random Forest model, we can see that Games played, Three Points Attempted, and Field Goal Percentage take the top of our important variables. These all make sense: a team that plays more games is more experiences, one that shoots more threes will eventually outscore a team that only shoots twos given each shoot them relatively-efficiently, and efficiency is also hugely important in making the most out of each possession. 

# Conclusion
What did we discover? Reinforcements for obvious statements-good offense, and good defense are needed for any championship team. The variance and excitement of the tournament, the upsets and head-scratchers, are also reflected in our exploratory data analysis, where some seasons saw a less-than-league average offense or defense ended up winning the championship. This is indicative of the single-elimination style of March Madness, as opposed to a 7 game series like the NBA uses, which helps to rule out variance and let the better team move on. We also learned that predicting championships are hard-my initial models were returning prediction accuracies in the 20%'s. Future iterations of this project could be improved by scraping more data, and utilizing implementing game-by-game data for each season. Regardless of prediction toughness, we can see the reflections of the unique aspects of March Madness-it's unpredictability-in both our exploratory data analysis and model building parts of this project.
