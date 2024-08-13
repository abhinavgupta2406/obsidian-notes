CART - Classification and Regression Trees
![[Pasted image 20240303083125.png]]

> Regression trees are more complex than classification trees

![[Pasted image 20240303083421.png]]
In above image:
* There are two independent variable - x1, x2
* There is a dependent variable y which is not shown in the diagram as it is 3rd dimension coming out of the screen (example is shown on right top)
* We don't need to worry about y at this moment as we will make our decision tree first and then predict for y.

![[Pasted image 20240303083733.png]]
In above image:
* How and where splits are conducted is detected by the algorithm and it involves something like __information entropy__
* So when the split happens, is it adding some value the way we group our points
* Algorithm stops when there is certain minimum information needs to be added. Once it cannot add any more value to out setup by splitting the leaf
* Each split is called leaf
* Final leafs are called terminal leafs

![[Pasted image 20240303084348.png]]
* For above splits, this is our decision tree

Now we have to predict value for y

![[Pasted image 20240303084545.png]]
* We take average of y for all the splits or terminal leafs
* So once we have value of x1 and x2 to make prediction, through that we can predict value of y by checking in which split those x1 and x2 are there. The average value for that split will be new predicted value of y.

![[Pasted image 20240303084846.png]]

> Decision tree model is not good for single feature dataset (or 2 dimensional dataset), they are more adapted to datasets with many features or high dimensional dataset.