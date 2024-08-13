# Multiple Linear Regression Intuition - Step 5

![[Pasted image 20240226071237.png]]

![[Pasted image 20240226071312.png]]

# Backward Elimination

![[Pasted image 20240226071343.png]]
Above significance level is set to 5%

![[Pasted image 20240226071439.png]]

# Forward Selection
![[Pasted image 20240226071933.png]]

# Bidirectional Elimination (aka Stepwise Regression)
![[Pasted image 20240226072147.png]]

# All Possible Models
![[Pasted image 20240226072512.png]]

As models are increasing exponentially, it is more resourceful

![[Pasted image 20240226072602.png]]

> In these tutorials, we will look mainly into Backward Selection as it is the most fastest one.
---
> In multiple linear regression, there is no need of feature scaling as each independent variable (x1, x2, ...) are multiplied with coefficients (b1, b2, ...), so it doesn't matter few coefficients have higher or lower value.

![[Pasted image 20240228071457.png]]

> Do we need to check assumptions of linear regression? Answer is absolutely not. (check again in Multiple Linear Regression in Step 2b)

> We don't need to take care of dummy variable trap as it will be automatically handled by the library.
> 
> Also, library will also take care of logic of backward elimination, etc. We don't have to worry about P value. Library class will automatically identify best features.
> 
> Library is scikit learn
> 
> Implementing things like backward elimination, etc takes time.



