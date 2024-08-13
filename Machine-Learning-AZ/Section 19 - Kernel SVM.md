 ![[Pasted image 20240328082719.png]]

On the left, SVM can separate these points as they are linearly separable. But there is no way to separate on the right linearly.

So on the right side data, we can add extra dimensions and make our data linearly separable.

# Mapping to Higher Dimension

## A Higher Dimensional Space

Single dimension space which is not linearly separable, which means this is non linearly separable dataset:
![[Pasted image 20240328083027.png]]

So we will map to higher dimension to make linearly separable
![[Pasted image 20240328083720.png]]

Now we have got linearly separable data, and then we will project the separable data in 1 dimensional space.

Now we are going to have 2D space
![[Pasted image 20240328083916.png]]

![[Pasted image 20240328084006.png]]

![[Pasted image 20240328084040.png]]

# The Kernel Trick

K stands for Kernel
l stands for landmark

![[Pasted image 20240328084216.png]]

![[Pasted image 20240328084603.png]]

Red spot in above image is the landmark, which is the projection of highest point

![[Pasted image 20240328084827.png]]

![[Pasted image 20240403071917.png]]

![[Pasted image 20240403071937.png]]

In above image, K value of red is zero or very near to zero, while for green ones, values is greater then 0

![[Pasted image 20240403072029.png]]

![[Pasted image 20240403072044.png]]

So above points will do classification

Computations are not happening in higher dimensional space, it is happening in lower dimension space. Our visualisation is in higher dimensional space.

![[Pasted image 20240403072447.png]]

Above image is more complex example compared to last ones. Here we are adding 2 Kernel functions.

# Types of Kernel Functions

![[Pasted image 20240403073148.png]]

Video was not very detailed. There was small explanation about Sigmoid Kernel. In Sigmoid Kernel, points on the right which have higher values are classified from the points on left which are lower values.

# Non-Linear SVR

Suppose we have below data points:
![[Pasted image 20240427102151.png]]

We will see this in 3rd dimension:
![[Pasted image 20240427102221.png]]

![[Pasted image 20240427102340.png]]

* Points on blue plane is very close to zero
* Points which doesn't have arrow is not visible, this is to avoid confusion. Imagine that they are also plotted.
* Now we are on 3D model, so we can run linear model, which will look something like below:

![[Pasted image 20240427102525.png]]

Now we have to identify where it is intersecting - yellow line:
![[Pasted image 20240427102601.png]]

Now if we project on 2D
![[Pasted image 20240427102638.png]]

![[Pasted image 20240427102655.png]]

![[Pasted image 20240427102709.png]]

Where are support vectors?
![[Pasted image 20240427102833.png]]

So we want to minimise gap between these hyperplanes to reduce errors

![[Pasted image 20240427102925.png]]
In above image, red colored circle points are support vectors.

Everything here is for illustrative purpose, in reality we don't have to go to 3rd dimension and find our hyperplane, it will be too computationally expensive (we are going to use Kernel trick)

