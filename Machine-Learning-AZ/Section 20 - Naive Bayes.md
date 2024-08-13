# Bayes Theorem

![[Pasted image 20240509084833.png]]

For example, there are 2 machines in factory which is creating spanners. Upper ones are created by machine 1 (m1) and bottom ones are created by machine 2 (m2)

![[Pasted image 20240509084947.png]]

Now we have all pile of it. Workers goal is to take out defective spanners (marked as black)

![[Pasted image 20240509085031.png]]

What is the probability that machine 2 produced defective spanners?

![[Pasted image 20240509085059.png]]

> Wrenches and spanners are same thing.

![[Pasted image 20240509085442.png]]

![[Pasted image 20240509085828.png]]

> Vertical line (|) means given

![[Pasted image 20240509085946.png]]
We don't need probability of machine 1 is not required - to keep things clean

![[Pasted image 20240509090133.png]]

![[Pasted image 20240509090202.png]]

 ![[Pasted image 20240509090302.png]]

![[Pasted image 20240609094757.png]]

![[Pasted image 20240609094856.png]]
We are basically doing same thing in the formula. But counting is very time consuming, or sometimes we don't have access to information/numbers. So therefore Bayes Theorm is useful.

# Naive Bayes Intuition

![[Pasted image 20240612214653.png]]

![[Pasted image 20240612214708.png]]

![[Pasted image 20240612214731.png]]
Lets take an example. People who walk to office or drives to office.

![[Pasted image 20240612214826.png]]

![[Pasted image 20240612214903.png]]
We will calculate these values one by one in these steps.

![[Pasted image 20240612214952.png]]

![[Pasted image 20240612215008.png]]

## Lets calculate

![[Pasted image 20240612215032.png]]

![[Pasted image 20240612215100.png]]

![[Pasted image 20240612215140.png]]
 * To calculate Marginal Likelihood, select a radius to draw around observation.
 * We can decide this radius on our own, you need to decide for your algorithm. This will be input variable - you can select less or more.
 * Then we will look into all the points inside the circle.
 * So radius will have big say in the way our algorithm works.

![[Pasted image 20240612215702.png]]
Number of similar observations means number of points inside the circle.

![[Pasted image 20240612215815.png]]

![[Pasted image 20240612215855.png]]

![[Pasted image 20240612215932.png]]

![[Pasted image 20240612220008.png]]

![[Pasted image 20240612220031.png]]
Here we are going to find posterior probability for those who drives.

![[Pasted image 20240612220110.png]]

![[Pasted image 20240612220123.png]]

![[Pasted image 20240612220139.png]]

![[Pasted image 20240612220153.png]]

![[Pasted image 20240612220227.png]]

---
* 