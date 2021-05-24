README

This repository is my Capstone project, undertaken as part of General Assembly’s Data Science Immersive course.

Executive Summary

The aim of the project is to understand and identify what makes a player win a game of Player Unknown’s Battlegrounds, and thus accurately predict who the winner of each game might be.

Contents:



1. What is Player Unknown’s Battlegrounds?
2. Data collection and cleaning
3. EDA
4. Modeling
5. Findings
6. Limitations and further work

1. What is Player Unknown’s Battlegrounds?

Player Unknown’s Battlegrounds (PUBG) is an online multiplayer battle royale game with millions of monthly players and a dedicated pro league with millions in prize money.

100 players drop onto an island and must battle it out to be the last man standing. **Players start with no equipment** and must find it themselves by looting buildings. **Every few minutes, the playable area of the map begins to shrink down towards a random location**, pushing players together until only one survives.

2. Data collection and cleaning

PUBG match data is available through the [PUBG developer API](https://developer.pubg.com/). The data was retrieved in a 3 step process:



1. Call a single Player ID to return match IDs from the past 2 weeks, creating a list of match IDs
2. Call a Match ID to return an overview of the match (telemetry URL, match type, region, number of players, time, etc.), a brief synopsis of each player’s performance (time survived, placement position, number of kills, etc.) and a list of all Player IDs involved in the match.
3. Pull the Telemetry URL, which gives a full match breakdown, in depth info for each kill, movement logs of players and much more.

Steps 1 & 2 were repeated until sufficient (18,000) SOLO matches were collected.

From the match overview, 9 players were identified; top 3 placing, 2 mid placing, 10th from last placing, last placing and 3 players with the most kills. These ‘key’ players were identified to understand player behaviours across the board.

The telemetry data contains every action which happens throughout a game. For each match, the following information was pulled:



*   First and Last record of initial flight path
*   Center point of each play area (gamestate)
*   Care package drop locations
*   Players alive at each gamestate
*   Every death (Victim/killer location, damage type, weapon used, gamestate, timestamp)
*   Landing location and location at each gamestate for the 9 key players identified before

The match data was saved as a JSON, and the telemetry data for each match was saved as a CSV file. These were then compiled into a dataframe and joined together. (link joining notebook)

With this information, it is then possible to effectively recreate an overview of the match, as shown below. The dotted line is the initial flight path, the circles are each gamestate play area, and the triangles are the location of each death, coloured by the gamestate (green-red-blue).

Having obtained a dataframe where each row was a different match (and 2000+ columns) a new, more focused ‘death dataframe’ was made detailing each kill. It contained:



*   Distance between killer and victim
*   Gamestate
*   Killer’s health on victim’s death
*   Type of weapon used (groups derived from unique weapons)
*   Damage reason (Headshot, leg shot, torso shot, etc.)
*   Death # (were 0 is the last death, and depending on how many players start the match, ~99 is the first death)

A second, ‘ranking dataframe’ was created; for each player in each match, it contained:



*   Final placement rank
*   Kills
*   Heals used
*   Damaged dealt
*   Distance travelled
*   Time survived
*   Console played on (Xbox, PlayStation, Stadia)

This ranking dataframe was further reduced to only include the last 2 players alive, and placement rank was replaced with “did_win”, which would later be the target. If a player won the match, one kill was removed (95% of the time, the winner would kill the second place player) to reduce data leakage.

3. EDA

The first steps taken in the EDA was to examine the behaviours of the ‘key players’ identified before. 

Here we can see the time survived; first and second place should always be the same, as the game finishes when the second place player dies. Third, mid placing and bottom placing players each survive for less time, which makes sense. There is more range in the killer’s (right) time survived; you can achieve a lot of kills in a short space of time, but generally speaking more game time means more time to kill players.

Above we can see the distribution of kills. Crucially, achieving a lot of kills isn’t necessary to win the game, but top placing players average more kills than second place.

We also created heatmaps to show common landing locations for players at the start of the match. Here we can see the first and second place preferred landing spots; there doesn’t appear to be much difference.

When we look at some mid placing and bottom placing players we can see a change in strategy. Compared to top players, the middle placing players are more distributed away from the really popular areas in the middle of the map and instead group in smaller clusters around them. The map for the last placing player (first player to die) is distinctive and highlights an important game mechanic; if a player quits a game before it starts, they will drop to the ground at the end of the flight path and either drown over water or stand unprotected and defenseless on the perimeter of the map. There is still a strong signal from the center of the map - it is likely if multiple players are vying for the same location, one of them will quickly die and place last.

We can explore the deaths further by looking at the death dataframe. Below we can see graphs showing the reasons for each death. Damage from a gun is the most common cause, followed by simulated AI deaths (ingame simulated players being removed/despawned) and bluezone damage (incremental damage from not being inside the current play area).

We can look at how the cause of death changes over the course of a match by comparing death count by reason v death number, where early deaths are on the left and late on the right.

There are some clear trends on display here; gun deaths are fairly consistent throughout the match, aside for a lul in the middle explained by the simulated AI deaths. Bluezone deaths spike around 7th place (this makes sense as the bluezone does more damage near the end of the match). Conversely, drownings happen a lot in the earliest stages of a match.

We can explore the gun deaths in greater detail by looking at specific weapons, below we can see a sample weapon from each category (shotguns, sniper rifle, special crate weapon, assault rifles and submachine guns).

We can see that weapon use changes over the course of the match; players have time to find their preferred weapons, or the playstyle and viability of weapons changes over the match. We can examine that by looking at how the distance between killer and victim changes over the course of the match, as below.

The distance of kills changes significantly over the course of the game. At the start, most kills are very close range; players are usually within buildings or close quarters whilst they search for weapons and equipment. Compare this to mid game, where players have more area to roam and more established, the distance between killer and victim increases significantly. The distance reduces towards the end as the player area shrinks, pushing players together.



4. Modeling

A selection of models were used to predict two different things; first, what type of gun was used for a kill, second, predicting who would win the game.

<span style="text-decoration:underline;">Predicting gun type used</span>

Looking at the death dataframe, each weapon was categorized, and then this was used as a target for the model. The categories and their normalized presence is shown to the right. Assault_rifles_auto is the most prevalent, so **the baseline accuracy is 0.41.**

Using the predictors shown above, a range of models were run; the scores of which you can see below. For a more comprehensive look at the models page. An overview of the models performance can be seen below. 



<p id="gdcalert1" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image1.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert2">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image1.png "image_tooltip")


Looking at the coefficients of the Logistic Regression model, we can see distance, death number and finisher health are all good indicators when predicting the weapon type used. Intuitively, this makes sense; some weapons aren’t appropriate for long/close range damage. Equally, the close quarters nature and the lack of weapon options at the start of the game also have a large bearing, shown by the importance of the death number.



<p id="gdcalert2" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image2.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert3">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image2.png "image_tooltip")


Here we have a confusion matrix for the model predictions. We can see that we are often predicting assault rifle auto for the majority of deaths. Sniper_dmr is able to be predicted well; with distance likely being a strong indicator. Submachine gun deaths are often misclassified as assault rifles. 



<p id="gdcalert3" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image3.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert4">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image3.png "image_tooltip")


<span style="text-decoration:underline;">Predicting the winner</span>

Having reduced the placement dataframe to just the last 2 players alive of each game, we then tried to predict, based on the features seen below, if a player had won the game or not. 

The results are summarised below.



<p id="gdcalert4" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image4.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert5">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image4.png "image_tooltip")


The last model, which was a bagging classifier ensemble model, returned the highest r² score, was run through a number of grid searches to optimise it.

Looking at the coefficients from the Logistic Regression model, we can get an idea of what factors indicate a potential winner, and thus characteristics of winning players. 

By far the largest coefficients are damage dealt, kill place and kills; and whilst there is likely some significant multicollinearity between these features, it is evident that having more kills and damage is indicative of a match winner.



5. Findings

The EDA and modeling suggest that when predicting what type of weapon caused a kill, the number of players remaining and the distance of the kill are key factors. When predicting a winner, more active players with more kills are expected to win.

To translate this into play styles and strategies for that game; whilst it is possible to play passively and avoid fights to become of the last two players surviving, this player is unlikely to win the game. In Player Unknown’s Battlegrounds, surviving isn’t enough - you need to be able to kill.



6. Limitations and further work

Limitations

When considering the weapon type predictor model, its usefulness is limited as players are able to carry two weapons, and are likely to use whichever is most appropriate for the situation. Also, there are more situational nuances to consider with weapon selection, such as availability of cover or preparedness.

When considering the winner prediction model, the main limitations are the lack of scope (only predicts from the last two players alive) and failing to account for a players situational advantages, for example proximity to play area, superior cover/area, or a players individual skill.

Further work

With such in-depth data available for each match, there is a lot of scope for further work. One restrictive element is the nature of the play area for each match; reducing at a uniform rate but to a random location within the current play area. This makes it difficult to compare like for like games - this could be negated to some degree with a far larger data set.

Including skill based data into the death dataframe would also add more depth to the quality of the dataset. Season and lifetime performance data is available through the API; requesting that and joining it with existing death dataframe could offer further insight.

Creating a tool for PUBG players to use to predict the probability of them winning the final showdown could offer value to players, although could be distracting.

With a larger data set, a model could be made to predict optimal locations to play for wins (or to maximise kills) based on past players performance.
