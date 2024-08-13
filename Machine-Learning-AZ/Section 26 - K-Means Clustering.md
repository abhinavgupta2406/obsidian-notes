# What is Clustering? (Supervised vs Unsupervised Learning)

> Clustering is, where model gets to think itself

![[Pasted image 20240810095458.png]]

![[Pasted image 20240810095545.png]]

* Supervised Learning - this is what we were learning
	* You already have training data and answers (answers are the training data which you supply to the model)
	* We ask model to learn from the answers from training data. For example, with photo of Apple, we pass a label or annotation of text saying Apple. So ML model learns from that.
* Unsupervised Learning
	* Here we don't have answers and the model has to think for itself.
	* So we ask model to group these fruits into different categories even though we don't know or pass the categories of fruits to the model. So machine has no understanding of the data, like which one is Apple or Banana.
	* It can just see certain similarities or certain differences in the data. From that, it can make conclusions and make its own group.

> In supervised learning, you give model an opportunity to train where it has the answers. In unsupervised learning, you don't have answers to supply to the model.

Below is an example in Business sense:
![[Pasted image 20240810100250.png]]

* You don't have pre-existing categories, preexisting classes or groups of customers to group them into. So we are applying clustering.
* Now further dig deeper in business sense, spending sense.. how to best use it.

# K-Means Clustering

First step is to decide how many clusters you want.

In below screenshots, lets say we decided to have 2 clusters

Now for each cluster, you need to place randomly placed centriod on scatter plot
![[Pasted image 20240811094554.png]]

Next K-means will assign each of the data points to the closest centroid, which is easiest done by joining this equidistant line.

![[Pasted image 20240811094730.png]]

Next thing is to calculate centre of mass or centre of gravity for each of the clusters, which will give position of the centre of mass
![[Pasted image 20240811094836.png]]

![[Pasted image 20240811094907.png]]
Now we move the centre of mass to the new position, and repeat the process. We assign datapoints to the nearest centroid. 

Now we keep on repeating the process.
![[Pasted image 20240811095100.png]]

![[Pasted image 20240811095119.png]]

We keep on doing this until we get into situation where doing the process again doesn't change anything.

![[Pasted image 20240811095257.png]]
These are the clusters.

# The Elbow Method

K-means doesn't necessarily have to work in 2 dimension, it can work in many dimensions.

Here are the data points:
![[Pasted image 20240811095815.png]]

Now how do we decide, how many clusters to select as very first step. Elbow method is one of the approach to help you make this decision. Sometimes you will know how many clusters are there based on domain knowledge.

![[Pasted image 20240811095954.png]]

![[Pasted image 20240811100016.png]]

![[Pasted image 20240811100035.png]]

![[Pasted image 20240811100051.png]]

* As we can see to calculate WCSS, we need clusters to really exist.
* Every time we have to run K-means clustering algorithm and then we calculate the WCSS
* So it is kind of bit backwards, we don't first do the elbow method to find the optimal number of clusters then do K-means. Instead, we do K-means many times, find the WCSS for every single setup - whether 1 cluster, 2 clusters, etc, then we apply the elbow method

> The more clusters we have, less the value of WCSS will be. It is because distance between points and centroid will reduce for each cluster as number of clusters will increase. So we can continue increasing the number of clusters till we reach the max - which is equal to number of data points, this is where distance between points and centroid will be zero, and hence WCSS will be zero.

Chart we can built:
![[Pasted image 20240811100723.png]]

![[Pasted image 20240811100758.png]]
* Now we find out where the king or elbow of the chart, which will be the optimal number of clusters. This is when WCSS stops dropping as rapidly.
* Of course, it is judgement call and sometimes it can be unclear. There can be 2 potential candidates or more candidates for optimal number of clusters, but that you need to decide as data scientist.

# K-Means++

![[Pasted image 20240812194920.png]]
* Lets say we want to apply K-means at top.
* Now lets say we again apply K-means at bottom, and as centroid are chosen at random, they are initialised in a different way. So we got different clusters at bottom.
* This is not good. Results are different.

![[Pasted image 20240812195126.png]]
* They are only different because initialisation of the centroid was different. This is called __random initialisation trap__ because:
	* Main reason is, you want to be deterministic, that is you want result to be same. It should not be different just because centroid were initialised at different points.
	* Looking at the dataset visualisation, you can intuitively see what the cluster should look like. Looks like K-means at the top shows us the correct clusters. But at the bottom, we do not see the best clusters.

![[Pasted image 20240812195511.png]]

![[Pasted image 20240812195606.png]]
![[Pasted image 20240812195627.png]]
* Furthest one with a weight is chosen as next centroid. __Notice the thickness of the lines, which shows the weight.__

![[Pasted image 20240812195703.png]]
![[Pasted image 20240812195811.png]]
![[Pasted image 20240812195825.png]]
![[Pasted image 20240812195837.png]]
* These are the final clusters.

* K-means does not guarantee that there will not be any issue with initialisation because it is done at random, but because it is done in __weighted random fashion__, the chances of that happening are much lower. This does solve the problem we get in __random initialisation trap__.
---
# Coding

* We are going to create a __dependent variable__ which will take finite number of values, lets say 4-5 values. Each of the values will be the class of this dependent variable which we are going to create.
* To identify the patterns in the data, we have to build the dependent variable.

## Importing the Libraries

``` python
import numpy as np
import matplotlib.pyplot as plt
import pandas as pd
```

## Importing the dataset

``` python
dataset = pd.read_csv('Mall_Customers.csv')

# for each feature, there will be new dimension so it will be difficult to visulise with lots of features.
X = dataset.iloc[:, [3, 4]].values
```

## Using the elbow method to find the optimal number of clusters

```python
from sklearn.cluster import KMeans
wcss = []

# range() starts with 0, and end is excluded from the list
for i in range(1, 11):
	#random_state can be any value
	kmeans = KMeans(n_clusters = i, init = 'k-means++', random_state = 42)
	kmeans.fit(X)
	wcss.append(kmeans.inertia_)

plt.plot(range(1, 11), wcss)
plt.title('The Elbow Method')
plt.xlabel('Number of clusters')
plt.ylabel('WCSS')
plt.show()
```

## Training K-Means model on the dataset

```python
# same as above cell

# This was already imported above
# from sklearn.cluster import KMeans

# Through elbow method, we got 5 clusters
kmeans = KMeans(n_clusters = 5, init = 'k-means++', random_state = 42)
# kmeans.fit(X)

# Now we have to create a dependent variable, which will hold values 1, 2, 3, 4, 5
y_kmeans = kmeans.fit_predict(X)
```

## Visualising the clusters

```python
# s means size of dot on plot

# X have 2 columns. 0 -> Annual Income, 1 -> Spending Score
# X[y_kmeans == 2, 0] means, get all annual incomes which are in cluster 3=
# X[y_kmeans == 2, 1] means, get all spending scores which are in cluster 3
plt.scatter(X[y_kmeans == 0, 0], X[y_kmeans == 0, 1], s = 100, c = 'red', label = 'Cluster 1')
plt.scatter(X[y_kmeans == 1, 0], X[y_kmeans == 1, 1], s = 100, c = 'blue', label = 'Cluster 2')
plt.scatter(X[y_kmeans == 2, 0], X[y_kmeans == 2, 1], s = 100, c = 'green', label = 'Cluster 3')
plt.scatter(X[y_kmeans == 3, 0], X[y_kmeans == 3, 1], s = 100, c = 'cyan', label = 'Cluster 4')
plt.scatter(X[y_kmeans == 4, 0], X[y_kmeans == 4, 1], s = 100, c = 'magenta', label = 'Cluster 5')

  
# cluster_centers_ return X and Y position of all centroids of clusters.
# kmeans.cluster_centers_[:, 0] means return X positions of all centroids
plt.scatter(kmeans.cluster_centers_[:, 0], kmeans.cluster_centers_[:, 1], s = 300, c = 'yellow', label = 'Centroids')

plt.title('Clusters of customers')
plt.xlabel('Annual Income (k$)')
plt.ylabel('Spending Score (1-100)')
plt.legend()
plt.show()
```