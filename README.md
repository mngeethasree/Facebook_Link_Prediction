# Facebook_Link_Prediction
##Objective:

To predict the possibility of future link between all the users, given the current state of connections.
Given a new user pair (Um, Un), model should be able to predict what is the probability that um is most likely to follow un in future.


##Structure of Data:
In the dataset, each record consists of a pair of users who are currently connected to each other
For example, 
A, B  -  indicates that A follows B (A -> Source node, B-> Destination node)
B, A  -  indicates that B follows A (B -> Source node, A-> Destination node)

Connections can be two way. For example, if A follows B, its not necessary that B follows A. But if they do follow each other, they will be represented as two seperate records in the dataset.

Hence this data can be represented as directed graph.

##Exploratory Data Analysis:
1. Distribution of number of followers for each user - 99% of the users have 40 or less followers
2. Distribution of number of followees for each user - 99.9% of the users follow less than 123 other users.
3. Distribution of sum of number of followers and number of followees - represents the degree of connection of one user with others in the network.

##Formulating as supervised classification problem:
In order to translate to a classification problem we need a good set of positive and negative class points for the model to train on. 
Since our data is confined to only existing connections, model can only use these as positive class data points. 



##Feature Engineering:
For each pair of users, create additional features representing characteristics of both users.
