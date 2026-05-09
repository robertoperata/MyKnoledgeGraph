---
tags:
  - math
  - machine-learning
  - linear-algebra
feature: 
type: course
author: "[[Jon Krohn]]"
source: 
---
# The Essential Learning Foundations

github.com/jonkrohn/ML-foundations
https://github.com/the-deep-learners/TensorFlow-LiveLessons/blob/master/notebooks/lenet_in_tensorflow.ipynb
https://github.com/the-deep-learners/TensorFlow-LiveLessons/blob/master/notebooks/deep_net_in_tensorflow.ipynb
## Orientation to Linear Algebra
>[!info] Algebra is arithmetic that included non-numerical entities like x:

$$
\begin{aligned}
&2x+5 = 25 \\[0.5em]
&2x = 25 - 5 \\[0.5em]
&2x = 20 \\[0.5em]
&\frac{2x}{2} = \frac{20}{2} \\[0.5em]
&x = 10
\end{aligned}
$$

Quadratic equation are in the form of:
$$
\begin{align}
ax^{2}+bx+c=0\\[0.5em]
\end{align}
$$
and is solved with:
$$
\begin{align}
x = \frac{-b\pm\sqrt{b^{2}-4ac}}{2a}
\end{align}
$$
the discriminant:
$$
\begin{align}
\Delta = b^{2}+4ac=0\\[0.5em]
\Delta > 0 \text{: two real solutions}\\[0.5em]
\Delta = 0 \text{: one real solutions}\\[0.5em]
\Delta < 0 \text{: no real solutions}\\[0.5em]
\end{align}
$$
Linear Algebra is a system of equations
- could be many equations
- could be many unknowns in each equation

$$
\begin{align}
y = a + bx_{1}+ cx_{2}+\ldots+ mx_{m}
\end{align}
$$
$$
\begin{bmatrix} 
y_{1}=& a +& bx_{1,1} +& cx_{1,2} +& \dots +& mv_{1,m}\\[0.5em]
y_{2}=& a +& bx_{2,1} +& cx_{2,2} +& \dots +& mv_{2,m}\\[0.5em]
\vdots =&  \vdots +& \vdots +& \vdots +& \ddots+&  \vdots \\[0.5em]
y_{n}=& a +& bx_{n,1} +& cx_{n,2} +& \dots +& mv_{n,m}
 \end{bmatrix}
$$

### Exercise 1:
1<sup>st</sup> April -> 1 Kj/day
1<sup>st</sup> May -> 4 Kj/day

$$
\begin{aligned}
&4(x - 30) = 1x \\[0.5em]
&4x - 120 = x \\[0.5em]
&4x - x = 120 \\[0.5em]
&3x = 120 \\[0.5em]
&x = \frac{120}{3} \\[0.5em]
& x = 40 \\[0.5em]
&\text{1. } 10^{th} \text{ May } \\
&\text{2. } 80 \\
&\text{3. undetermined} 
\end{aligned}
$$
## Data structures for algebra
### Tensors
ML generalisation of vectors and matrices to any number of dimensions.
Different dimension of tensors:
$$
\begin{aligned}
&\text{scalar: } x\\[0.5em]
&\text{vector: } [x_{1},  x_{2}, x_{3}]\\[0.5em]
&\text{matrix: } 
\begin{bmatrix} 
x_{1,1}  &x_{1,2} \\[0.5em] 
x_{2,1}  &x_{2,2} \\[0.5em]
\end{bmatrix} \\[0.5em]
&\text{3 tensor: 3 dimensional}
\end{aligned}
$$

| Dimension | Mathematical Name | Description              |
| --------- | ----------------- | ------------------------ |
| 0         | scalar            | magnitude only           |
| 1         | vector            | array                    |
| 2         | matrix            | flat table, e.g., square |
| 3         | 3-tensor          | 3d table, e.g., cube     |
| n         | *n*-tensor        | higher dimension         |
### Scalar
- no dimension
- single number
- denoted in lowercase, italics
- should be typed like all other tensors: int, float32 (best practice in NumPy, PyTorch, TensorFlow)
### Vectors and Vector Transposition
- one dimensional array of numbers
- denoted in lowercase, italic, bold e.g. $\boldsymbol{x}$
- arranged in an order, so element can be accessed by its index
	- elements are scalars so *not* bold e.g., second element of $\boldsymbol{x} \text{ is } x_{2}$
- representing a point in space:
	- vector of length two represents location in 2D matrix
	- length of three represents location in 3D cube
	- length of *n* represents location in *n*-dimensional tensor 
#### Vector Transposition
$$
\begin{bmatrix} x_1 & x_2 & x_3 \end{bmatrix}^T = \begin{bmatrix} x_1 \\ x_2 \\ x_3 \end{bmatrix}
$$


I understand that if there's a microservice with a stream application that join two or more topics and store to a ktable, the microservice reads from all partitions of all topics.
If the application scale up I think all the instances of the microservice continue to read from all partitions for all topic to find the join needed. Is it correct?
Are there best practice?