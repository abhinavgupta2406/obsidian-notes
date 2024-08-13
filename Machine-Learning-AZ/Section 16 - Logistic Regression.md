![[Pasted image 20240312081913.png]]
* Logistic regression predicts the categorical data instead of continuous data (in regression). For example, predicting yes or no is Logistic Regression
* In above screenshot, there is only one dependent variable (age)
* On the right side, we have plotted the graph.
* The curve in yellow is called logistic regression curve or sigmoid curve
* In the formula on left bottom, `p` is probability.
* In graph, curve shows the probability of the person purchasing the health insurance.
* For example, person with age 35 has probability of 42%, while with age 45 has probability of 81%.
* Generally, if the probability is >= 50% that means yes, if < 50% that means no

![[Pasted image 20240312082423.png]]
* If more variables are added, this is how the equation will look like.

# Maximum Likelihood

Identify which curve will be better

![[Pasted image 20240312082927.png]]
* To identify we multiply probability (from the curve) will all the input data we have. This will help identify the best curve - with maximum likelihood.
* On right graph, remember that curve is to show people who said yes to purchase insurance. So it should be 1 - (probability of saying no)

![[Pasted image 20240312083153.png]]