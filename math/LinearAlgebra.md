
## 向量

+ [加减法](#加减法)
+ [数乘（标量乘法）](#数乘（标量乘法）)
+ [点乘（内积、数量积）](#点乘（内积、数量积）)

### 加减法

代数公式

$$
设 \quad \vec{a} = \begin{bmatrix}
1 \\ 
2 \\
\end{bmatrix}, \quad \vec{b} = \begin{bmatrix}
3 \\ 
1 \\
\end{bmatrix}
$$
$$
则 \quad \vec{a} + \vec{b} = 
\begin{bmatrix}
1 \\ 
2 \\
\end{bmatrix}
 +
\begin{bmatrix}
3 \\ 
1 \\
\end{bmatrix}
= 
\begin{bmatrix}
1 + 3 \\ 
2 + 1 \\
\end{bmatrix}
=
\begin{bmatrix}
4 \\ 
3 \\
\end{bmatrix}
$$
$$
则 \quad \vec{a} - \vec{b} = 
\begin{bmatrix}
1 \\ 
2 \\
\end{bmatrix}
 -
\begin{bmatrix}
3 \\ 
1 \\
\end{bmatrix}
= 
\begin{bmatrix}
1-3 \\ 
2-1 \\
\end{bmatrix}
=
\begin{bmatrix}
-2 \\ 
1 \\
\end{bmatrix}
$$
$$
即 \quad \vec{a} \pm \vec{b} = 
\begin{bmatrix}
a_1 \\ 
a_2 \\
\dots
\end{bmatrix}
\pm
\begin{bmatrix}
b_1 \\ 
b_2 \\
\dots
\end{bmatrix}
$$

几何意义

![Diagram.svg](../Diagram.svg)


### 数乘（标量乘法）

代数公式

$$
k\vec{v} = k
\begin{bmatrix}
v_1 \\ 
v_2 \\
\dots \\
v_n
\end{bmatrix}
=
\begin{bmatrix}
kv_1 \\ 
kv_2 \\
\dots \\
kv_n
\end{bmatrix}
,\quad k \in R
$$

几何意义

![Diagram1.svg](../Diagram1.svg)

###  点乘（内积、数量积、点积）

代数公式

**向量点乘** 等价于 *1行n列* 与 *n行1列*  两个**矩阵相乘**

$$
\vec{a} \bullet \vec{b} =
\begin{bmatrix}
a_1 \\ 
a_2 \\
\dots \\
b_n
\end{bmatrix}
\bullet
\begin{bmatrix}
b_1 \\ 
b_2 \\
\dots \\
b_n
\end{bmatrix}
=
\sum_{i=1}^{n}a_ib_i
\qquad
\Longleftrightarrow
\qquad
\begin{bmatrix}
a_1 &
a_2 &
\dots
a_n
\end{bmatrix}
\begin{bmatrix}
b_1 \\ 
b_2 \\
\dots \\
b_n
\end{bmatrix}
=
\sum_{i=1}^{n}a_ib_i
$$

几何意义

![Diagram2.svg](../dot_product.svg)

### 叉乘（向量积、外积、叉积）

代数公式1




