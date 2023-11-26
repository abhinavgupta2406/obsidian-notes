![[Pasted image 20231126202244.png]]

# Training Set & Test Set
> Generally we split like this - Test as 20%, Train as 80%

Training set is used to build the model, and then apply the model on the test set. Then we compare the predicted values with the actual values (on test data actual values will be known)

![[Pasted image 20231126203355.png]]

# Feature Scaling
> Feature scaling is always applied to columns, it is never applied across columns so we will never apply feature scaling to the data inside the row.

![[Pasted image 20231126204235.png]]

## Normalization

Values will be between 0 and 1

## Standardization

Subtracted from average and divided by standard deviation. Values will be between -3 and 3

> In practical examples, we will use standardization. For initial example, we will use normalization for simplicity.

# Why Normalization?
![[Pasted image 20231126204821.png]]

Statement is, who is near to purple coloured person?

Normalization is required as we can see the difference of values for income and age is quite different. For example, 10,000 value is very high compared to the age difference of 1. So it is not very easy to find out which person is near to purple coloured person. In fact, values could be presented in different ways, for example age can be represented as minutes - for which the difference is quite high.

![[Pasted image 20231126205207.png]]
After normalization, it can be seen that purple person is close to blue person.