CART - Classification and Regression Trees

![[Pasted image 20240616191737.png]]
- Classification trees are used to classify your data, so they will need categorical variables such as:
	- Male or Female
	- Apple or Orange
	- Different types of colors and variables
- Regression trees are designed to help predict outcome which can be real numbers, so:
	- Salary of person
	- Temperate which is going to be there outside
- In this section, we are going to work on Classification Trees


![[Pasted image 20240616193030.png]]
* How decision tree works, it cuts up points into slices into several iterations.
* Algorithm is going to find optimal splits that are going to maximise number of different points in each pockets or leafs.

![[Pasted image 20240616193744.png]]

![[Pasted image 20240616193801.png]]
* It is not necessary that while evaluating any point, it will follow the whole path from root.
* For example, if there is some point in green between Split 1 and Split 4, it may directly start with condition X2 < 20. It may NOT follow the whole path from X2 < 60

![[Pasted image 20240616194232.png]]

