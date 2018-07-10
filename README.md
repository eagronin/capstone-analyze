# Analysis

## Overview
This section describes the analysis performed in the final project of the Big Data Specialization by University of California San Diego on Coursera. The project focuses on developing recommendations for increasing revenue in a fictitious online game called "Catch the Pink Flamingo". 

The data for the project consists of 9 files with data on in-app purchases, ad clicks and game-specific information as well as 6 files with chat data.  The data were downloaded from the Coursera website.  All the data for the project were generated by a development team of data scientists.  The data were designed to simulate many aspects of game play and in-game user activity.  

The project entails classification analysis by fitting a decision tree and cluster analysis using k-means.  It also uses graph analytics to identify the chattiest users / teams, longest conversations and active user groups. 

The analysis in this project was performed using Python, Spark, KNIME and Neo4j.

We will discuss the classification analysis first, then we will proceed with the cluster analysis and graph analytics.

Data exploration is described in the [previous section](https://eagronin.github.io/capstone-prepare/).

The recommendations based on the analysis performed in this section are reported in the [next section](https://eagronin.github.io/capstone-report/).

## Classification Analysis
As we discussed in the [previous section](https://eagronin.github.io/capstone-prepare/), in-app purchases of expensive items generate more revenue for Eglence, Inc. than purchases of inexpensive items. Therefore, it is important to identify users who are more likely to purchase expensive items and target such items to these users.  We call the users who tend to pucharase expensive items “HighRollers” and the users who tend to purchase inexpensive items “PennyPinchers”.  Big-ticket items are those with a price of more than $5.00, and inexpensive items are those that cost $5.00 or less.  

In order to predict who is a HighRoller and who is a PennyPincher based on the known attributes, we fitted a decision tree in KNIME.  A screenshot of the resulting decision tree can be seen below:

![](https://github.com/eagronin/capstone-analyze/blob/master/decision-tree.png?raw=true)

![](https://github.com/eagronin/capstone-analyze/blob/master/decision-tree-simple.png?raw=true)

The confusion matrix calculated using the test data shows that the model correctly predicted 308 PennyPinchers.  Similarly, it correctly predicted 192 HighRollers.  The model incorrectly predicted 38 HighRollers (classifying them as PennyPinchers).  Similarly, the model incorrectly predicted 27 PennyPinchers (classifying them as HighRollers).

A screenshot of the confusion matrix can be seen below:

![](https://github.com/eagronin/capstone-analyze/blob/master/confusion-matrix.png?raw=true)

As seen in the screenshot above, the overall accuracy of the model is 88.496%.

The implications of this analysis are discussed in the [next section](https://eagronin.github.io/capstone-report/).

## Cluster Analysis
As we discussed in the [previous section](https://eagronin.github.io/capstone-prepare/), groups of users with different attributes (for example, experienced vs. inexperienced players) are likely to have differences in their tendency to make in-app purchases.  Therefore, revenue and profitability can be increased by choosing different strategies for targeting and setting fees for hosting in-app purchase items shown to users from different groups.

We performed cluster analysis in Spark to identify 3 distinct groups of users, based on 3 user attributes identified in the [previous section](https://eagronin.github.io/capstone-prepare/) (teamLevel, accuracyRate and Revenue).  The cluster centers generated by this analysis are shown below:

Cluster | #	Cluster Center array([teamLevel, accuracyRate, revenue])
--- | ---
1 | array([-0.89, -0.06, -0.31])
2 | array([0.40, 0.38, 2.68])
3 | array([0.84, -0.02, -0.25])

These clusters can be differentiated from each other as follows:

**Cluster 1** is different from the others in that it includes the least experienced players (teamLevel is 0.89 standard deviations below the mean).  These players tend to have the lowest accuracy rate (likely due to their lack of experience) and the lowest amount spent on in-app purchases.

**Cluster 2** is different from the others in that it represents players with intermediate-to-advanced level of experience (0.39 standard deviations above the mean).  These players have the highest accuracy rate and spend the most on in-app purchases.

**Cluster 3** is different from the others in that it represents the most experienced players (teamLevel is 0.83 standard deviations above the mean).  The accuracy rate of these players is higher than that of the beginners and lower than that of the intermediate-to-advanced players.  It is possible that the accuracy rate declines with teamLevel (after certain point) as the game becomes more complicated and, despite the high level of experience, it becomes harder to catch the right flamingos.  It is also possible that highly experienced users become less engaged with the game (after certain point) and, as a result, their accuracy rate declines.  The amount spent on in-app purchases by these players is low, just above the amount spent by the beginners.  As mentioned above, it is possible that the most experienced players become less engaged with the game and, as a result, do not spend substantial amount on in-app purchases.   

The implications of this analysis are discussed in the [next section](https://eagronin.github.io/capstone-report/).

## Graph Analytics
The players in the game (also referred to as users) can chat with the members of their teams.  Each player can initiate a chat session in which the team members can post comments, respond to each other’s comments and mention other players in their comments.  Players can also join and leave a chat session.  

As we discussed in the [previous section](https://eagronin.github.io/capstone-prepare/), chattier users, initiators of longer conversations and users who belowng to chattier teams and active user groups are likely to be more valuable, because of their potential to spread information to wider audiences.  As a result, Eglence, Inc. can increase its revenue by choosing the right marketing strategy to target such users, for example, showing the more expensive items to such users.  Even if these users are not going to buy these items, they may influence others in their networks to buy such items.

### Finding the longest conversation chain and its participants
The length of the longest conversation chain is 9.  The code that generated the longest conversation chain is shown below:

```Graphviz
match p=(i1:ChatItem)-[:ResponseTo*]->(i2:ChatItem)
return p, length(p) order by length(p) desc limit 1
```

There were 5 unique users involved in this conversation.  The code to calculate the number of users is shown below.  The IDs of the ChatItems used in the code correspond to the first and last ChatItems, respectively, in the conversation chain.

```GraphQL
match p=(i1:ChatItem)-[:ResponseTo*]->(i2:ChatItem)
where i1.id = 52694 and i2.id = 7803
with p
match (u:User)-[:CreateChat]->(i:ChatItem)
where i in nodes(p)
return count(distinct u)
```

### Analyzing the relationship between top 10 chattiest users and top 10 chattiest teams
The top 10 chattiest users were selected by counting the number of ChatItems created by each user and outputting the users by count in descending order.

Chattiest Users

User ID |	Number of Chats
--- | ---
394 | 115
2067 | 111
1087 | 109

The top 10 chattiest teams were selected by counting the number of ChatItems created by each team and outputting the teams by count in descending order.  The ChatItems created by each team were selected as being part of team chat session owned by the team.

Chattiest Teams

Team ID | Number of Chats
--- | ---
82 | 1324
185 | 1036
112 | 957

Only one of the top 10 chattiest users (ID: 999) belongs to one of the top 10 chattiest teams (ID: 52).  This result was obtained by outputting the team IDs of the top 10 chattiest users and comparing these team IDs with the team IDs of the top 10 chattiest teams.

### How Active Are Groups of Users?
We will answer this queastion by computing an estimate of how “dense” the neighborhood of a node is. If we can identify highly interactive neighborhoods, we can potentially target some members of the neighborhood for direct advertising.

The analysis of identifying such interactive neighborhoods was performed as follows.  First, an edge was created between any pair of users who either responded to each other’s chats or mentioned each other in a chat.  In that process, the edges of self-loops (e.g., a user responds to their own chat) were removed.  Second, a "clustering coefficient" ranking the top 10 chattiest users in terms of density of their neighborhoods was calculated for each user.  

A clustering coefficient is a score ranging from 0 (a disconnected user) to 1 (a user in a clique – where every node is connected to every other node). For example, if the number of neighbors of node is 5, then the clustering coefficient of the node is the ratio between the number of edges amongst these 5 neighbors(not counting the given node) and 5 * 4 / 2 (all the pairwise edges that could possibly exist). Thus the denominator is k * (k-1) / 2 if the number of neighbors of the node is k.

Examples of Active Users (based on Cluster Coefficients)

User ID | Coefficient
--- | ---
394 | 1.00
461 | 1.00
209 | 0.95
516 | 0.95
554 | 0.90
999 | 0.87
1087 | 0.80

The implications of this analysis are discussed in the [next section](https://eagronin.github.io/capstone-report/).

Next step: [Recommendations](https://eagronin.github.io/capstone-report/)
