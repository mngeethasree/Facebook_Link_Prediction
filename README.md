# Facebook_Link_Prediction
## Objective:

This project aims to predict the possibility of future link between all the users, given the current state of connections.
Given a new user pair (Um, Un), model should be able to predict what is the probability that Um is most likely to follow Un in future.


## Structure of Data:
In the dataset, each record consists of a pair of users who are currently connected to each other <br>

For example, <br>
a record A, B  -  indicates that A follows B (A -> Source node, B-> Destination node) <br>
while a record B, A  -  indicates that B follows A (B -> Source node, A-> Destination node) <br>

Connections can be two way. For example, if A follows B, its not necessary that B follows A. But if they do follow each other, they will be represented as two seperate records in the dataset.

Hence this data can be represented as directed graph.

<img width="330" alt="img1" src="https://github.com/mngeethasree/Facebook_Link_Prediction/assets/68059811/7f7c22ea-a80a-4034-b186-fcfff9c56410">


## Exploratory Data Analysis:
**1. Distribution & Density function of number of followers for each user**

<img width="440" alt="image" src="https://github.com/mngeethasree/Facebook_Link_Prediction/assets/68059811/77c75481-8ef8-4eb7-bca6-34bc1e01c428"> 
<img width="500" alt="image" src="https://github.com/mngeethasree/Facebook_Link_Prediction/assets/68059811/5c218db7-c7c2-4de1-b796-d0a0e38ab8d1"> <br>

<img width="178" alt="image" src="https://github.com/mngeethasree/Facebook_Link_Prediction/assets/68059811/4bce054b-c100-42e4-b70c-7378992cb943">


99% of the users have 40 or less followers. In other words, only 1% of users have follower count higher than 40.


**2. Distribution of number of followees for each user**

<img width="480" alt="image" src="https://github.com/mngeethasree/Facebook_Link_Prediction/assets/68059811/b20c937e-440c-424b-983d-e907da8bf01f"> 
<img width="508" alt="image" src="https://github.com/mngeethasree/Facebook_Link_Prediction/assets/68059811/cfab14e4-d4e2-4134-9246-fc8217160dc7"> <br>

<img width="187" alt="image" src="https://github.com/mngeethasree/Facebook_Link_Prediction/assets/68059811/9db567de-2152-4b4e-b2a6-03ccc5772e58">


99.9% of the users follow less than 123 other users.

**3. Distribution of sum of number of followers and number of followees**

<img width="425" alt="image" src="https://github.com/mngeethasree/Facebook_Link_Prediction/assets/68059811/3f5008f4-741c-4f7e-9912-725bb30bea22"> <br>

<img width="164" alt="image" src="https://github.com/mngeethasree/Facebook_Link_Prediction/assets/68059811/bf3d3af4-d845-4a73-8890-44005c94c154">

This metric represents the degree of connection of one user with others in the network.

## Data Preparation:

### Formulating as supervised classification problem - Creating missing links:
In order to translate to a classification problem we need a good set of positive and negative class points for the model to train on. 
Since our data is confined to only existing connections, model can only use these as positive class data points. Inorder to generate extra negative class points, we can randonmly sample any two users who are not currently connected in the network such that the shortest path between two users is greater than two (missing links) . Inorder to have a balanced data set with good distribution of positive and negative data points, we can generate as many missing links as there are existing links.

### Train Test Split for Model Validation:
Append the missing links with actual links in the network with appropriate target labels. Create an 80-20 split for train and test datasets.

<img width="466" alt="image" src="https://github.com/mngeethasree/Facebook_Link_Prediction/assets/68059811/4f4091c7-15cd-4202-a135-502f9b0126a8"> <br>

Almost **~7.12%** of the users that fall in the Test set are NOT present in the Training data. Hence the model will have never seen these users during the training process.

## Feature Engineering:
For each pair of users representing each record, create additional features representing similarity between users followers and followees. Following are certain features that are created for the directed graph data: <br>

**(i) Jaccard Distance for Followers:** <br>
Jaccard index measures the degree of similarity between two sets. Mathematically, it is intersection over union between two sets. Using this feature, we can measure the degree of overlap between followers of both the users. <br>

$$j = \frac{\left|X \cap Y\right|}{\left|X \cup Y\right|}$$ <br>

Refer to https://www.statisticshowto.com/jaccard-index/ for more details on Jaccard Distance <br>

**(ii) Jaccard Distance for Followees:** <br>
Similar to followers, we can create Jaccard distance for followees. Using this feature, we can measure the degree of overlap between followees of both the users represented by source and destination nodes. <br>

**(iii) Cosine Distance for Followers:** <br>
Cosine distance is a measure of dissimilarity between two sets. In this context, cosine distance between follower sets of both the source and destination nodes represents how dissimilar are followers of both the users are and hence can be used as a feature for the future link prediction. <br>

$$\text{Cosine Distance} = \frac{\left|X \cap Y\right|}{\left|X\right| \cdot \left|Y\right|}$$ <br>

**(iv) Cosine Distance for Followees:** <br>
Similar to followers, we can create Cosine distance for followees. Using this feature, we can measure the dissimilarity between followees of source and destination node users.<br>

**(v) Page Rank:** <br>
PageRank computes a ranking of the nodes in the graph G based on the structure of the incoming links. It was originally designed as an algorithm to rank web pages. 
PageRank for the directed graph is calculated using networkx library. <br>

PageRank - https://en.wikipedia.org/wiki/PageRank <br>
For more details on pagerank API from networkx - https://networkx.github.io/documentation/networkx-1.10/reference/generated/networkx.algorithms.link_analysis.pagerank_alg.pagerank.html <br>

**(vi) Shortest path between nodes in the graph:** <br>
Another important feature could be the shortest path distance between nodes in the graph. 
If the shortest path is smaller, it indicates that the two users are more closely connected compared to users with a larger shortest path distance.
But if the users are directly connected to each other, then the shortest path distance would be 0. Hence in this case, the edge is removed from the graph and the shortest path is then calculated. <br>

**(vii) Flag indicating if users belong to same community:** <br>
Creating a binary indicator that denotes whether two users belong to the same community or group within the network graph. Communities are densely connected subgraphs with sparse connections to other parts of the graph. This feature can be useful for understanding relationships in smaller communities. This binary flag represents if the users belong to same weakly connected component. <br>

**(viii) Adamic/Adar Index:** <br>
Adamic/Adar measures is defined as inverted sum of degrees of common neighbours for given two vertices/nodes. It is mathematically represented by following formula <br>
$$A(x, y) = \sum_{u \in N(x) \cap N(y)} \frac{1}{\log(|N(u)|)}$$ <br>

**(ix) Is user following back:** <br>
Is following back is a binary flag indicating if destination node user follows source node user. This can be very useful feature in predicting whether source node will follow destination node in future.

**(x) Katz centrality:** <br>
Katz centrality computes the centrality for a node based on the centrality of its neighbors. It is a generalization of the eigenvector centrality.
The Katz centrality for node i is  given by <br>
$$x_i = \alpha \sum_j A_{ij} x_j + \beta$$ <br>
where A is the adjacency matrix of the graph G with eigenvalues $\lambda$. The parameter $\beta$ controls the initial centrality and <br> 
$$\alpha < \frac{1}{\lambda_{max}}$$ <br>

Refer to - https://en.wikipedia.org/wiki/Katz_centrality <br>
https://www.geeksforgeeks.org/katz-centrality-centrality-measure/ <br>

**(xi) Hits Score:** <br>

**(xii) SVD features from Adjacency matrix:**  <br>

**(xiii) preferential attachment:** <br>


## Performance metric to validate the model:
It is important to be precise about model prediction as well as capture all actual possible links within model predictions. Since both precision and recall are important,  F1 score can be considered as a metric to select the optimal model

## Modeling
Once features are setup, train a Random Forest Classifer and observe the performance for different values of n_estimators, depth etc.

## Final Results
Final Model has an roc_auc of 0.93.

### Interpretation
1. follows back is the top feature, indicating a user is most likely to follow back any of its followees.
2. Including preferential attaching as a feature improved model performance and turns up in top 10 features.
3. svd features do not seem to be helpful in the classification decision



