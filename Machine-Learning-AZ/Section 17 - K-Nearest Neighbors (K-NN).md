![[Pasted image 20240319082111.png]]
* Lets say we have 2 categories - category 1 and category 2
* Category 1 is represented as red, while category 2 is represented as green.
* Now we have a new datapoint, but we have to identify, in which category this data point belongs to.
* This is where K-NN is used for classification.

![[Pasted image 20240319082255.png]]
* In step 2, it can be any method to get the distance of two data points. For example, here we are going to use Euclidean distance, otherwise it can be Manhattan distance or so on.

![[Pasted image 20240319082426.png]]

![[Pasted image 20240319082444.png]]

![[Pasted image 20240319082516.png]]
* Circled ones are the closest ones

![[Pasted image 20240319082552.png]]

![[Pasted image 20240319082614.png]]
* As category 1 had 3 neighbours so new data point will belong to category 1