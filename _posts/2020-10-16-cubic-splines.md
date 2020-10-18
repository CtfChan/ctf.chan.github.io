---
title: "Robotics Blog: Cubic Splines"
tags: [robotics]
style: fill
color: dark
description: Another method of path parametrization
---
<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>

## Cubic Splines
A cubic spline is constructed from a series of piecewise third degree polynomials. Given $n$ control points 
$$\{(x_0,y_0), ... , (x_{n-1}, y_{n-1}) \}$$

We can construct $n-1$ segments where the $i$th segment is computed as:

$$ f_i (x) = a_i + b_i (x - x_i) + c_i (x - x_i)^2 + d_i (x - x_i)^3$$

with $x \in [x_i, x_{i+1}]$

Typically we set the 2nd derivative of each $f_i$ to 0 at the endpoints to create a system of $n-2$ equations that we can solve for $Ax=b$.

$$
A = 
\begin{bmatrix}
1 &  &  \\ 
  & 2(h_0 + h_1) & h_1 \\ 
  & h_1 & 2 (h_1 + h_2) & h_2 \\
  &     &   h_2 &  2(h_2 + h_3) & h_3 \\
  &     &         & \ddots & \ddots & \ddots &  \\
  &     &         &     & &    &  h_{n-3}      &    2 (h_{n-3} + h_{n-2})   & & 0 \\
  &     &         &    & &     &        &   0    & & 1
\end{bmatrix}
$$


$$
x = 
\begin{bmatrix}
c_0 \\
c_1 \\
\vdots \\
c_{n-1}
\end{bmatrix}
$$

$$
b = 
\begin{bmatrix}
0 \\
\frac{3 (y_2 - y_1)}{h_1} - \frac{3 (y_1 - y_0)}{h_0} \\
\vdots \\
\frac{3 (y_{n-1} - y_{n-2})}{h_{n-2}} - \frac{3 (y_{n-2} - y_{n-3})}{h_{n-3}} \\
0
\end{bmatrix}
$$


Having solved for all $c_i$s solve for $d_i$ and $b_i$ using:

$$  b_i  = \frac{(y_{i+1} - y_{i})}{h_i} - \frac{1}{3} (2c_i + c_{i+1}) h_i $$

$$ d_i = \frac{c_{i+1} - c_{i}}{3 h_i} $$


## Derivation 

We seek to solve for $a_i$, $b_i$, $c_i$ and $d_i$ for each segment of the spline with $i = 0, ..., n-1$.

Since all $(x-x_i)$ terms go zero when $x=x_i$, we can easily get $a_i$ with $f_i(x_i) = a_i = y_i$. 

We require our function function to be continuous and twice differentiable. The zeroth, first and second derivative with respect to x for each segment is reproduced below.
$f_{i}''(x_{i+1}) = f''_{i+1}(x_{i+1})$.
$$ f_i (x) = a_i + b_i (x - x_i) + c_i (x - x_i)^2 + d_i (x - x_i)^3$$

$$ f_i' (x) = b_i+ 2 c_i (x - x_i) + 3 d_i (x - x_i) ^ 2$$

$$ f_i'' (x) = 2 c_i + 6 d_i (x - x_i)$$


Let us define $h_i = x_{i+1} - x_i$ and substitute $a_i = y_i$ from here on. 


- Continuous zeroth derivative means $ f_i(x_{i+1}) = y_{i+1} $. For $i = 0, ..., n-1$.

$$ y_i + b_i (h_i) + c_i (h_i)^2 + d_i (h_i) ^3 = y_{i+1}$$

- Continuous first derivative means $f_{i}'(x_{i+1}) = f'_{i+1}(x_{i+1})$. For $i = 0, ..., n-2$.

$$ b_i+ 2 c_i (h_i) + 3 d_i (h_i) ^ 2 = b_{i+1} $$

- Continuous second derivative means $f_{i}''(x_{i+1}) = f''_{i+1}(x_{i+1})$. For $i = 0, ..., n-2$.

$$ 2 c_i + 6 d_i (h_i) = 2 c_{i+1} $$ 


From the three equations above, we seek to express $d_i$ and $b_i$ in terms of $c_i$. 

Lets start off with expressing $d_i$ in terms of $c_i$ using $f_{i}''(x_{i+1}) = f''_{i+1}(x_{i+1})$ 

$$ 2 c_i + 6 d_i (h_i) = 2 c_{i+1} $$ 

$$ d_i = \frac{c_{i+1} - c_{i}}{3 h_i} $$

We can also express $b_i$ in terms of $c_i$ using  $ f_i(x_{i+1}) = y_{i+1} $. 


$$ y_i + b_i (h_i) + c_i (h_i)^2 + d_i (h_i) ^3 = y_{i+1}$$

$$  b_i (h_i) = (y_{i+1} - y_{i}) - c_i (h_i)^2 - d_i (h_i) ^3$$

$$  b_i  = \frac{(y_{i+1} - y_{i})}{h_i}- c_i h_i - d_i (h_i) ^2$$

Sub in $d_i$.

$$  b_i  = \frac{(y_{i+1} - y_{i})}{h_i}- c_i h_i - \frac{c_{i+1} - c_i}{3 h_i} (h_i) ^2$$

$$  b_i  = \frac{(y_{i+1} - y_{i})}{h_i}- c_i h_i - \frac{c_{i+1} - c_i}{3 } (h_i) $$

$$  b_i  = \frac{(y_{i+1} - y_{i})}{h_i} - \frac{c_{i+1} h_i}{3} - \frac{2 c_i h_i}{3} $$

$$  b_i  = \frac{(y_{i+1} - y_{i})}{h_i} - \frac{1}{3} (2c_i + c_{i+1}) h_i $$

Subbing our expressions for $b_i$ and $d_i$ into the equation formed from  $f_{i}'(x_{i+1}) = f'_{i+1}(x_{i+1})$.

$$ b_i+ 2 c_i (h_i) + 3 d_i (h_i) ^ 2 = b_{i+1} $$


$$ [\frac{(y_{i+1} - y_{i})}{h_i} - \frac{1}{3} (2c_i + c_{i+1}) h_i] + 2 c_i h_i  + 3[ \frac{c_{i+1} - c_{i}}{3 h_i}] h_i^2 =   \frac{(y_{i+2} - y_{i+1})}{h_{i+1}} - \frac{1}{3} (2c_{i+1} + c_{i+2}) h_{i+1} $$

$$ \vdots $$

$$   h_i c_i + 2(h_i + h_{i+1}) c_{i+1} +  h_{i+1} c_{i+2} = 3 \frac{y_{i+2} - y_{i+1}   }{h_{i+1}} - 3 \frac{y_{i+1} - y_{i} }{h_i}$$


Because we set the second derivative to zero at the end points of the spline, we can set $c_0 = c_{n-1} = 0$. So we only have to solve for $n-2$ $c_i$'s. Conveniently, we have $n-2$ "versions" of the equation above with $i = 0, ..., n-2$. This allows us to create a system of linear equations of the form $Ax=b$ where we can solve for all $c_i$'s. The top left and bottom right 1s in the $A$ matrix encode $c_0 = c_{n-1} = 0$, the rest of the matrix expresses encodes the $n-2$ equations for $c_i$


$$
A = 
\begin{bmatrix}
1 &  &  \\ 
  & 2(h_0 + h_1) & h_1 \\ 
  & h_1 & 2 (h_1 + h_2) & h_2 \\
  &     &   h_2 &  2(h_2 + h_3) & h_3 \\
  &     &         & \ddots & \ddots & \ddots &  \\
  &     &         &     & &    &  h_{n-3}      &    2 (h_{n-3} + h_{n-2})   & & 0 \\
  &     &         &    & &     &        &   0    & & 1
\end{bmatrix}
$$


$$
x = 
\begin{bmatrix}
c_0 \\
c_1 \\
\vdots \\
c_{n-1}
\end{bmatrix}
$$

$$
b = 
\begin{bmatrix}
0 \\
\frac{3 (y_2 - y_1)}{h_1} - \frac{3 (y_1 - y_0)}{h_0} \\
\vdots \\
\frac{3 (y_{n-1} - y_{n-2})}{h_{n-2}} - \frac{3 (y_{n-2} - y_{n-3})}{h_{n-3}} \\
0
\end{bmatrix}
$$

Having solved for all $c_i$s we can use our expressions for $d_i$ and $b_i$ in terms of $c_i$ to solve for our final coefficients.