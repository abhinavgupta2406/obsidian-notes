![[Pasted image 20240618073052.png]]
Consider we are identifying whether a personal who have received an offer in email will take the offer or not. So 1 represents person took the offer, while 0 represents person did not take the offer.

`y` represents actual DV (dependent variable) value or the actual values, and `y^` represents predicted value.

* Lets start with actual values. We are mapping the actual values on the graph.
* `Red plus` on 0 and 1 are these actual values mapped, and `Blue Plus` are the projection on the curve.
* `Grey plus` on 0 and 1 are the predicted values, and `Blue Plus` again is projection on curve.

* We can observe that #1 and #4 values are giving proper answers. #1 shows that person did not take the offer and curve also shows that. #4 shows that person took the offer and curve projection also shows that.
* But now coming to #2 and #3. For #2, `Red plus` (or actual value) shows that person took the offer, but curve shows (`Blue Plus`) that it did not take the offer (because < 0.5).
* For #3, `Red plus` (or actual value) shows that person did not took the offer, but curve shows (`Blue Plus`) that it did take the offer (because > 0.5).
* So there is a false positive and false negative.

* Now how to remember false positive is Type 1 error, and false negative is Type 2 error.
* `Yellow triangle with !` is a warning, and we know that false positive will still show positive, but it will false, which is not end of the world.
* `Red triangle with !` is an alert, and we know that false negative will not show anything positive, but it can be positive, which is risky.

![[Pasted image 20240618074306.png]]

![[Pasted image 20240618074325.png]]

---
# Accuracy Paradox

![[Pasted image 20240619212518.png]]
Suppose that we have a model and these are the results. And we identified accuracy rate to be 98%

Now consider we have completely abandoned the ML model, and it is not giving any results:
![[Pasted image 20240619212635.png]]

![[Pasted image 20240619212706.png]]
Now we can see that our accuracy rate has increased to 98.5%.
So even though we have increase in accuracy rate (even when we completely stopped using the model), that is why you should not base your judgement just on the accuracy rate.

---
# CAP Curve

Cumulative Accuracy Profile

Lets say, you are data scientist, and say you have total 100,000 customers and you send them offers. And only 10% of them responds to your offer communication. (This is a random sample)

![[Pasted image 20240804100807.png]]

Instead of sending offers randomly to customers, how about we pick and choose the customers we send the offers to. How do we pick and choose? Lets create a model.

Purchased is binary variable - yes or no. Plus take many other factors, like whether they are using mobile or computer, etc.

After building this model, select the customers to send offer communication. For example, male customers in certain age range who uses mobile, etc.

After building the new model, we get the new curve:
![[Pasted image 20240804101143.png]]

Red coloured one is when we used ML model to predict the customers.

> Red coloured line is CAP of your model.
> The better your model, the larger will be area between red and blue line.

Now convert axis values to percentage
![[Pasted image 20240804101353.png]]

Now lets say we ran another regression model which has less independent variables or we don't have access to these variables or many other reasons:
![[Pasted image 20240804101518.png]]
Now we know it will be worst (because of less independent variables). This new green CAP curve is from new model

> Here we can see how much gain we get between each model. Also called GAIN CHART.
> Additional gain we are getting by switching between each model.

![[Pasted image 20240804101840.png]]
* Grey coloured line is ideal model (or when we had a crystal ball)

![[Pasted image 20240804102036.png]]
* As we initially mentioned, 10% of the customers contacted responds to offers. In ideal scenario, all 10% of the customers responds to the offer. This is where curve is there in crystal ball.

If there is anything below Random line, that model is really bad and is disservice for your business.

![[Pasted image 20240804102241.png]]

---
# CAP Curve Analysis

![[Pasted image 20240804103004.png]]

What insights can we derive from above? If your curve is near to perfect model it is good. 

But how can we quantify our derivation? There is a standard approach, accuracy ratio.

![[Pasted image 20240804103244.png]]
aP is area under perfect model which is coloured in grey.

![[Pasted image 20240804103201.png]]
aR is area under the red line.

AR will always be between 0 and 1. If close to 1, it is good. Else if close 0, model is worst.

However, it can be quite complicated to get area under the curve. Statistical tools can do that for you. But how can you assess the CAP curve by just looking at it visually. It is not easy to get quantifiable matrix by just looking at the curve.

Now look at another approach:
![[Pasted image 20240804103630.png]]

* Look at 50% line on horizontal line on vertical axis, and look where it crosses your model, and where it crosses your vertical line.
* So we are getting outcome if we consider 50% of the population.

Rule of Thumb:
![[Pasted image 20240804103832.png]]

* When between 90-100%, it is too good but be very careful about over fitting:
	* This means that one of your independent variable is post factum variable (meaning that it is looking for a data which is looking into future). Person who applied model, forgot to take out this forward looking parameter.
	* Overfitting: You could be overfitting your model. Your model is so well fit to that specific dataset and heavily relying on the anomalies in that data set. And when you fit a new dataset where it is not trained, it will crash or not perform well.

---
# Conclusion of Classification
![[Pasted image 20240804104835.png]]