![[Pasted image 20240324072547.png]]

Points other then support vectors doesn't contribute to the algorithm.

![[Pasted image 20240324072832.png]]

Generally, anything on the left is negative hyperplane and anything on the right is positive hyperplane.

![[Pasted image 20240324073346.png]]

Lets say we have to classify Apples and Oranges:
* On the left in red points are apples, and on the right in green points are oranges.
* Typically, normal apples and oranges points will come in the covered area above.

![[Pasted image 20240324073406.png]]

But SVM is special, it looks for extreme values:
* On the left, Apple is looking like orange, but it is still Apple.
* On the right, Orange is green in color which is still orange.
* So these points are support vectors as these points are extremely rare.