![[Pasted image 20240302081833.png]]

* The points in left and right are same
* We are going to talk about left one
	* Ordinary Least Squares (OLS)
	* y with ^ is the point on the line.
	* Basically we are trying to reduce the errors by trying to find the minimum distance from the prediction (linear line) so that answers are accurate.

![[Pasted image 20240302082047.png]]

* In the right
	* E is Epsilon
	* So in the yellow tube, we are going to ignore the points for errors.
	* But the ones which are outside the tube are the points where we are interested.

![[Pasted image 20240302082214.png]]

Why is it called vector? It is because there are vector lines to the points which are outside yellow tube.
The ones which are highlighted in red, they are support vectors, because they are dictating or supporting the structure formation of the tube, that's why they are called support vectors.

---
# Non Linear SVR

![[Pasted image 20240302083158.png]]

---
> In SVR, we have to apply feature scaling as there are no coefficients (b0, b1, b2,...) to multiply with