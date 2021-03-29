## 1. Introduction to Deep Learning

### 什么是神经网络(What is Neural Network)

比如，通过一个房屋“尺寸-价格”的线性函数，来预测一个新的数据的价格：
$$
size \rightarrow \bigcirc \rightarrow price
$$
这个函数就是一个神经元(neuron)。

### 用神经网络进行监督学习(Supervised Learning with Neural Networks)

监督学习：一组输入对应一组输出

#### 结构化数据和非结构化数据

结构化数据(Structured Data): 具有类似数据库结构的数据，比如房屋信息，用户信息

非结构化数据(Unstructured Data): 类似于音频、图片、文本这样的原始数据。

### 为什么深度学习会兴起(Why are the deep learning only just now taking off)

1. 更多的数据(Data)

2. 更快的计算(Fast Computation)能力

3. 更好的算法(Algorithms)

   比如$Sigmoid \rightarrow ReLU$：Sigmoid的问题是它的梯度极难下降到0，造成了很大的性能损失 

## 2. Basics of Neural Network Programming

### 二分分类(Binary Classification)

在神经网络中，我们使用与机器学习相反的矩阵排列(n * m)：
$$
\left[
\begin{matrix}
X^{(1)}_0 & X^{(2)}_0 & .. & X^{(m)}_0 \\
X^{(1)}_1 & X^{(2)}_1 & .. & X^{(m)}_1 \\
.. \\
X^{(1)}_n & X^{(2)}_n & .. & X^{(m)}_n 
\end{matrix}
\right]
$$
就是将每一个数据放在列上。

M代表样本总数，N代表特征总数。

### 逻辑回归(Logistic Regression)

在神经网络中，常使用下面的方式计算逻辑回归：
$$
z = w^T x + b \\
\hat{y} = \sigma z = \sigma (w^T x + b)
$$
而不是把w和b合并为$\theta$。

### 代价函数(Cost Function)

> 也叫做损失函数。
>
> 准确来说，损失函数用于描述单一样本，代价函数用于描述整个数据集。

目标是使平方误差尽可能小(To be Square Error as small as possible)：
$$
\frac12 (\hat{y} - y)^2
$$
所以对每一个样本作用下面的公式：
$$
Loss = -y log \hat{y} - (1-y) log(1-\hat{y}) \\
y为1时，L = -log \hat{y}，\\
如果\hat{y}为1，L为0；如果\hat{y}为0，L无限大 \\
y为0时，L = - log(1-\hat{y})，\\
如果\hat{y}为1，L为无限大；如果\hat{y}为0，L为0
$$
所以损失函数是：
$$
Cost = - \frac1m \sum_{i=1}^m [y^{(i)} log \hat{y}^{(i)} + (1 - y^{(i)}) log (1 - \hat{y}^{(i)})]
$$

### 梯度下降法(Gradient Descent)

概述(Recap)：
$$
\hat{y} = \sigma (w^T x + b), \sigma(z) = \frac1{1 + e^{-z}} \\
J(w, b) = \frac1m \sum_{i=1}^m Loss(\hat{y}^{(i)}, y^{(i)}) \\
= - \frac1m \sum_{i=1}^m [y^{(i)} log \hat{y}^{(i)} + (1 - y^{(i)}) log (1 - \hat{y}^{(i)})]
$$
有最小值的函数称为凸函数(convex function)，它不会有很多不同的局部最优。

w就是梯度，即多个维度的斜率(slope)。梯度下降：
$$
w = w - \alpha \cdot dw
$$
其中$w$是预测函数的梯度，是预测函数的系数列表；$\alpha$是学习率；$dw$是$J$对$w$的求导，就是损失函数的当前斜率。

我们的目的是让损失函数达到最小值，即这个凸函数的$dw$趋近于0。

$dw$趋近于0的表现是：两次$w$的值趋近于相等。

所以梯度下降的代码大概就是：

```python
last_w = w 
w = w - alpha * dw 
b = b - alpha * db 
if last_w - w < 0.0001:
		break  # 0.0001是随便写的值
```

> $dw表示对w求导；\partial{w}表示对w求偏导，此时函数应当有多个变量。$

### 导数(Derivatives)

$$
f(x) = 3x，函数的导数是：\\
\frac{df(x)}{dx} = 3
$$

导数的含义是：当x增加一个无限小的值时，f(x)会增加3倍这个无限小的值。

### 计算图(Computation Graph)

比如J = 3(a + bc)，计算图表示就是：
$$
u = bc \\
v = a + u \\
J = 3v
$$

### 计算图的导数(Derivatives with a Computation Graph)

主要是说明导数的链式法则(Chain Rule)。
$$
\frac{dJ}{da} = 3 \\
\frac{dJ}{db} = 3c \\
\frac{dJ}{dc} = 3b
$$

### 逻辑回归的梯度下降(Implement Gradient Descent for Logistic Regression)

> 上面已经写出了公式，求出dw就行了

计算图：
$$
w, x , b \longrightarrow z=wx+b 
\longrightarrow \hat{y}=a=\sigma(z) 
\longrightarrow L(a, y)
$$
L对a(即y_hat, sigmoid)求导：
$$
Loss = -y log \hat{y} - (1-y) log(1-\hat{y}), a=\hat{y} \\
\frac{dL}{da} = - \frac{y}{a} + \frac{1-y}{1-a} 
= \frac{a-y}{a(1-a)} \\
$$

a对z求导：
$$
a = \sigma(z) = \frac1{1+e^{-t}} = (1+e^{-t})^{-1} \\

\frac{da}{dz} = -(1+e^{-t})^{-2} \cdot e^{-t} \cdot -1 = (1+e^{-t})^{-2} e^{-t} \\
= a^2 (\frac{1}{a} - 1) = a-a^2 \\
$$
z对w求导：
$$
z = wx + b \\
\frac{dz}{dw} = x
$$
所以，L对w求导：
$$
\frac{dL}{dw} = \frac{dL}{da} \cdot \frac{da}{dz} \cdot \frac{dz}{dw} 
= (a-y)x
$$
所以，**代价函数的导数，称为dw，它的求解是：**
$$
dw = \frac{dC}{dw} = \frac{1}{m} \sum_{i=1}^m (a^{(i)} - y^{(i)}) \cdot x^{(i)}
$$


### 向量化(Vectorization)

> Whenever possible, avoid explicit for-loops.

在numpy中，可以直接用向量化计算来代替for循环的计算，计算速度提升非常大。

如计算$z = w^T x + b$：（其中w和x都是列向量）

```python
z = np.dot(w, x) + b 
```

#### 举个例子

$$
v = 
\left[
\begin{matrix}
v1 \\
v2 \\
.. \\
vn
\end{matrix}
\right] \\
求得： \\
u = 
\left[
\begin{matrix}
e^{v1} \\
e^{v2} \\
.. \\
e^{vn}
\end{matrix}
\right]
$$

写成代码就是：

```python
u = np.zeros((n, 1))  # n行1列的0
for i in range(n):
  u[i] = math.exp(v[i])
```

但使用向量化的写法就是：

```python
import numpy as np
u = np.exp(v)
```

### 向量化的逻辑回归(Vectorizing Logistic Regression)

n表示n个特征，m表示m组数据。
$$
w.shape 是 (n, 1) \\
X.shape 是 (n, m) \\
b.shape 是 (1, 1)，它是个实数
$$
在numpy中，向量和实数进行相加时，会自动把实数扩展为向量。这个操作称为广播(broadcasting)。

求z = wx + b的代码：

```python
z = np.dot(w.T, X) + b  # shape是(m, 1)
a = sigmoid(z)  # a就是y_hat
```

求dw的代码：

```python
dw = np.sum(np.dot((a - y), x)) / m
```

### 广播(Broadcasting in Python)

比如，用m * n的矩阵，和1 * n的矩阵相加，会自动把1 * n的矩阵扩展为m * n的矩阵。

示例：

```python
import numpy as np

# Apples Beef Eggs Potatoes
X = np.array([
    [56.0, 0.0, 4.4, 68.0],  # Carb 碳水化合物
    [1.2, 104.0, 52.0, 8.0],  # Protein 蛋白质
    [1.8, 135.0, 99.0, 0.9]  # Fat 热量
])
X.shape  # (3, 4)

cal = X.sum(axis=0)
cal.shape  # (4,)
print(X / cal)

cal = cal.reshape(1, 4)  # 和上面的cal是一样的
cal.shape  # (1, 4)
print(X / cal)

"""
[[0.94915254 0.         0.02831403 0.88426528]
 [0.02033898 0.43514644 0.33462033 0.10403121]
 [0.03050847 0.56485356 0.63706564 0.01170351]]
 """
```

为不产生奇怪的bug，在使用矩阵时，确保其的秩(rank)不为1（就是shape不能是 (n,) 这样的形式）。比如创建一个列向量：

```python
# a column vector
np.random.randn(5, 1)  # 不要写：np.random.randn(5)
```

为保证矩阵的shape是正确的，可以使用：

```python
assert(a.shape == (5, 1))
```

## 3. One hidden layer Neural Network

### 神经网络表示(Neural Network Representation)

有输入层(Input Layer of the Neural Network)、隐藏层(Hidden Layer)和输出层(Output Layer)。

**输入层的不同节点对应的是不同的特征。**

表示：
$$
X输入层：写作a^{[0]} \space a 表示 activate \\
隐藏层：写作a^{[1]} \\
输出层：写作a^{[2]} \\
该神经网络一共两层
$$
不清楚的话看图片哦。

在隐藏层有多个节点，它用来计算当前层的任务：
$$
z^{[1]}_1 = w^{[1]T}_1x + b^{[1]}_1, a^{[1]}_1 = \sigma(z^{[1]}_1) \\
z^{[1]}_2 = w^{[1]T}_2x + b^{[1]}_2, a^{[1]}_2 = \sigma(z^{[1]}_2) \\
z^{[1]}_3 = w^{[1]T}_3x + b^{[1]}_3, a^{[1]}_3 = \sigma(z^{[1]}_3) \\
z^{[1]}_4 = w^{[1]T}_4x + b^{[1]}_4, a^{[1]}_4 = \sigma(z^{[1]}_4) \\
$$
所以Hidden Layer的输出是：
$$
z^{[1]} = W^{[1]}x + b^{[1]} \\
a^{[1]} = \sigma(z^{[1]})
$$
Output Layer的输出是：
$$
z^{[2]} = W^{[2]}a^{[1]} + b^{[2]} \\
\hat{y} = a^{[2]} = \sigma(z^{[2]})
$$

### 多个例子的向量化(Vectorizing across multiple examples)

$$
a^{[2](i)} 表示第i个训练实例的预测值，a^{[2]}就是\hat{y}。
$$

#### 分析参数是啥

首先，对于Input Layer来说，$x_1, x_2$，它们是x的n个特征。我们暂时不去管这些特征。

现在我们有一批数据：
$$
X = 
\left[
\begin{matrix}
X^{(1)} & X^{(2)} & .. & X^{(m)}
\end{matrix}
\right]
$$
在Hidden Layer有多组w和b，所以：
$$
z^{[1](1)} = W^{[1]}X^{(1)} + b^{[1]}
$$
这里的W是若干组系数，注意每一行是一组：
$$
W^{[1]} = 
\left[
\begin{matrix}
w_1 & w_2 & .. & w_n \\
w_1 & w_2 & .. & w_n \\
.. \\
w_1 & w_2 & .. & w_n
\end{matrix}
\right]
$$


这里的X是一条数据，有n个特征：
$$
X^{(1)} = 
\left[
\begin{matrix}
x_1 \\
x_2 \\
.. \\
x_n
\end{matrix}
\right]
$$
这里的b也是若干组系数，注意每一行是一组：
$$
b^{[1]} = 
\left[
\begin{matrix}
b \\
b \\
.. \\
b
\end{matrix}
\right]
$$

### 激活函数(Activation Functions)

目前我们在Hidden Layer和Output Layer都使用了Sigmoid函数作为激活函数。

把Hidden Layer的激活函数，换成tanh将是更优的选择。它使得数据分布在0的左右，其值是更有意义的。
$$
tanh = \frac{e^z - e^{-z}}{e^z + e^{-z}}
$$
但是tanh和Sigmoid共同的缺点是，当传入的值很大或很小时，tanh和Sigmoid的结果会很小，梯度下降的就会很慢。

所以更好的方法是：使用ReLU：
$$
a = max(0, z)
$$
使用ReLU的学习速度会比Sigmoid快很多。

结论：对于二分类工作，使用Relu作为Hidden Layer的激活函数，Sigmoid作为Output Layer的激活函数。对于非二分类工作，不要使用Sigmoid函数。

> leaky ReLU: a = max(0.01z, z)

### 为什么要使用非线性激活函数(Why does a neural network need a non-linear activation function)

因为两个线性函数的结合仍是线性的。

比如对于上面的模型：
$$
z^{[1]} = W^{[1]}x + b^{[1]} \\
a^{[1]} = \sigma(z^{[1]}) \\
z^{[2]} = W^{[2]}a^{[1]} + b^{[2]} \\
\hat{y} = a^{[2]} = \sigma(z^{[2]})
$$
我们把Sigmoid换成线性的，如y=x：
$$
z^{[1]} = W^{[1]}x + b^{[1]} \\
a^{[1]} = z^{[1]} \\
z^{[2]} = W^{[2]}a^{[1]} + b^{[2]} \\
\hat{y} = a^{[2]} = z^{[2]} \\
求得： \\
a^{[2]} = W^{[2]}a^{[1]} + b^{[2]} = W^{[2]}(W^{[1]}x + b^{[1]}) + b^{[2]} \\
所以：a^{[2]} = kx + l
$$
这样一来，再多的隐藏层都是无意义的。它们其实只是计算了一个线性的值。除非是在计算回归问题（如房价预测）。通常，只能在Hidden Layer使用非线性激活函数。

### 激活函数的导数(Derivatives of activation functions)

Sigmoid：
$$
g(z) = \frac1{1 + e^{-z}} \\
g\prime(z) = - \frac1{(1 + e^{-z})^2} \cdot - e^{-z} 
= g(z)^2 \cdot (\frac1{g(z)} - 1)
= g(z) (1 - g(z))
$$
tanh：
$$
g(z) = \frac{e^z - e^{-z}}{e^z + e^{-z}} \\
g\prime(z) = 
$$
ReLU：
$$
g(z) = max(0, z) \\
g\prime(z) = 
\left\{
\begin{matrix}
0, if (z < 0) \\
1, if (z > 0) \\
undefined, if (z = 0) \space 编程把这个归结到上面任意一个里面去
\end{matrix}
\right\}
$$
leaky ReLU：
$$
g(z) = max(0.01z, z) \\
g\prime(z) = 
\left\{
\begin{matrix}
0.01, if (z < 0) \\
1, if (z > 0) \\
undefined, if (z = 0) \space 编程把这个归结到上面任意一个里面去
\end{matrix}
\right\}
$$

### 神经网络的梯度下降法(Gradient Descent for Neural Networks)

现在呢，Input Layer输出$n^{[0]}$个特征，Hidden Layer输出$n^{[1]}$个特征，Output Layer输出$n^{[2]}$个特征。

所以对于w和b：
$$
w^{[1]}是：(n^{[1]}, n^{[0]}) \\
b^{[1]}是：(n^{[1]}, 1) \\
可以让n^{[1]}为1，体会一下： \\
w就是1行n^{[0]}列，表示n^{[0]}个特征对应的n^{[0]}个系数；\\
b就是1行，表示n^{[0]}个特征对应1个b； \\
n^{[1]}这个参数是共有n^{[1]}组w和b。
$$
继续：
$$
w^{[1]}是：(n^{[1]}, n^{[0]}) \\
b^{[1]}是：(n^{[1]}, 1) \\
w^{[2]}是：(n^{[2]}, n^{[1]}) \\
b^{[2]}是：(n^{[2]}, 1) 
$$
所以Cost Function是：
$$
J(w^{[1]}, b^{[1]}, w^{[2]}, b^{[2]}) =  - \frac1m \sum_{i=1}^m [y^{(i)} log \hat{y}^{(i)} + (1 - y^{(i)}) log (1 - \hat{y}^{(i)})] 
$$

J对$w^{[2]}$的导数（别忘了a是y_hat）：

> J是损失函数，a是y_hat、在这里是激活函数(Sigmoid)，z是激活函数的变量，w是要求的变量。a指的是$a^{[2]}$。

$$
dw^{[2]} = 
\frac{\partial{J}}{\partial{a}} \cdot
\frac{\partial{a}}{\partial{z}} \cdot
\frac{\partial{z}}{\partial{w^{[2]}}} \\
= \frac{a-y}{a(1-a)} \cdot 
a(1 - a) \cdot
a^{[1]} \\
= (a^{[2]} - y) \cdot a^{[1]}
$$



直接写老师计算的结果：
$$
dz^{[2]} = a^{[2]} - y \\
dw^{[2]} = dz^{[2]} a^{[1]T} \\
db^{[2]} = dz^{[2]} \\
dz^{[1]} = W^{[2]T}dz^{[2]} * g^{[1]}\prime(z^{[1]}) \\
dW^{[1]} = dz^{[1]} xT \\
db^{[1]} = dz^{[1]}
$$


放到numpy中：
$$
dZ^{[2]} = A^{[2]} - Y \\
dW^{[2]} = \frac1m dZ^{[2]} A^{[1]T} \\
db^{[2]} = \frac1m np.sum(dZ^{[2]}, axis=1, keepdims=True) \\
dZ^{[1]} = W^{[2]T}dZ^{[2]} * g^{[1]}\prime(Z^{[1]}) \\
dw^{[1]} = \frac1m dZ^{[1]}X^T \\
db^{[1]} = \frac1m np.sum(dZ^{[1]}, axis=1, keepdims=True) \\
$$


### 随机初始化

对于每一层的节点，应该使用随机初始化的权重参数。如

```python
w1 = np.random.randn(2, 2) * 0.01  # 因为z = w1 * x + b，z作用于sigmoid，值应该小，否则梯度下降很慢
b1 = np.zeros((2, 1))
```

## 4. Deep L-Layer Neural Network

### 个人复习逻辑回归

y_hat = a = sigmoid(z), z = wX + b

Coss = J = - 1/m * (yloga  + (1-y)log(1-a))

dw = dJ / dw = (a - y) * X / m

db = dJ / db = (a - y) / m

求最合适的w和b，使得Coss最小，即Coss(y_hat_last, y_hat) -> 0时

已有X和y，设w、b和alpha(学习率)

w_last = w

b_last = b

w = w - alpha * dw

b = b - alpha * db

计算y_hat和y_hat_last

Cost(y, y_hat) - Cost(y, y_hat_last) -> 0，欧了

此时的w和b就是我们要的最优w和b



预测：

y_predict = sigmoid(wX_test + b)

y_predict_int = np.array(y_predict > 0.5, dtype=int)  # 预测结果

accuracy = np.sum(y_predict_int == y_true) / y_true.shape[1]

### 符号表示(Deep neural network notation)

L: 层数(#layers)。注意层数的计算包括Output Layer(Ln)，不包括Input Layer(L0)

n: 单元数(#units)。如$n^{[1]}$，表示第1层的神经元数量

a: 激活函数(activation)，也是每一层的输出。如Input Layer(L0)的a等于x，Output Layer(Ln)的a等于$\hat{y}$



### 前向传播(Forward propagation in a deep network)

对单层进行for循环：
$$
Z^{[i]} = w^{[i]}A^{[i - 1]} + b^{[i]} \\
A^{[i]} = g^{[i]}(Z^{[i]})
$$
假设5层的神经网络，其神经元的数量分别是：`[2(Input), 3, 5, 4, 2, 1(Output)]`。

计算维度(Dimensions)：
$$
w^{[l]}: 维度是(n^{[l]}, n^{[l-1]}) \\
b^{[l]}: 维度是(n^{[l]}, 1) \\
X^{[l]}: 维度是(n^{[l-1]}, m) \\
Z^{[l]}: 维度是(n^{[l]}, m)
$$


### 搭建深层神经网络块(Building blocks of deep neural networks)

见图片。所有的blocks连接起来，就是一次梯度下降。它包括前向传播(forward propagation)和反向传播(backward propagation)。

见图片。

### 超参数(Hyperparameters)

> 生成参数的过程中，需要添加的辅助参数，如学习率(alpha)、迭代次数(number of iterations)、隐藏层数量(#hidden layers)、神经元数量(#hidden units)等。

## 作业2: 逻辑回归实现猫图片识别

### 代码

```python
from lr_utils import load_dataset
from matplotlib import pyplot as plt
import numpy as np

X_train_origin, y_train_origin, X_test_origin, y_test_origin, classes = load_dataset()

# 查看图片
plt.imshow(X_train_origin[12])
plt.show()

# 把 X_train 的 shape 由 (209, 64, 64, 3) 转为 (12288, 209)
X_train = X_train_origin.reshape((X_train_origin.shape[0], -1)).T
# 把 X_test 的 shape 由 (50, 64, 64, 3) 转为 (12288, 50)
X_test = X_test_origin.reshape((X_test_origin.shape[0], -1)).T

# y_train 和 y_test 的 shape 在 load_dataset 函数中处理过了
y_train = y_train_origin
y_test = y_test_origin

# 特征数量
n_dim = X_test.shape[0]

# 定义w和b
w = np.zeros((n_dim, 1))
b = 0

# 对数据进行标准化处理：将值投影到0-1之间
X_train = X_train / 255
X_test = X_test / 255


# Sigmoid
def sigmoid(z):
    return 1 / (1 + np.exp(-z))


# y_hat 即 a
def y_hat(w, X, b):
    assert X.shape[0] == w.shape[0]
    return sigmoid(w.T.dot(X) + b)


# 代价函数
def J(y, a):
    assert y.shape == a.shape
    m = y.shape[1]
    return -1 / m * np.sum(y * np.log(a) + (1 - y) * np.log(1 - a))


# 代价函数对w的求导
def dw(X, y, a):
    assert y.shape == a.shape
    m = y.shape[1]
    return 1 / m * X.dot((a - y).T)


# 代价函数对b的求导
def db(y, a):
    assert y.shape == a.shape
    m = y.shape[1]
    return 1 / m * np.sum(a - y)


# 前向传播
def propagate(w, b, X, y):
    assert w.shape[0] == X.shape[0] & X.shape[1] == y.shape[1]
    z = np.dot(w.T, X) + b
    a = sigmoid(z)
    return a


# 实现梯度下降法求最优W和b
def gradient_descent(X, y, w_init, b_init, n_iters=3000, eta=0.01, epsilon=1e-8):
    """
    梯度下降：m是样本个数，n是特征数量
    :param X: X训练集，shape为(n, m)
    :param y: y训练集，shape为(1, m)
    :param w_init: 起始的w值，shape为(n, 1)
    :param b_init: 起始的b值，int
    :param n_iters: 最大循环次数
    :param eta: 学习率
    :param epsilon: 认为是0的值
    :return:
    """
    assert X.shape[1] == y.shape[1] and X.shape[0] == w_init.shape[0]

    w = w_init
    b = b_init

    i = 0

    while i < n_iters:
        last_w = w
        last_b = b
        w = w - eta * dw(X, y, y_hat(w, X, b))
        b = b - eta * db(y, y_hat(w, X, b))
        i += 1
        v = J(y, y_hat(w, X, b)) - J(y, y_hat(last_w, X, last_b))
        if i % 100 == 0:
            print('第 i 次', i)
            print('epsilon', v)
        if abs(v) < epsilon:
            break

    return w, b


w_w, b_b = gradient_descent(X_train, y_train, w, b)

def predict(X, y_true):
    y_predict = y_hat(w_w, X, b_b)
    return np.sum(np.array(y_predict > 0.5, dtype=int) == y_true) / y_true.shape[1]

print('成功率约是' ,predict(X_test, y_test))

img_url = '/Users/goyave/Desktop/cat2.jpg'
img = plt.imread(img_url)

"""预测新图片"""

# 尺寸变换：由 (500, 500, 3) 变成 (64, 64, 3)
from skimage import transform
new_img = transform.resize(img, (64, 64, 3))
plt.imshow(new_img)
plt.show()
new_img = new_img.reshape(-1, 1)
print('shape', new_img.shape)
print(y_hat(w_w, new_img, b_b))
```

## 作业3：One Hidden Layer实现二分类

### 代码

```python
# coding: utf-8

# In[2]:


import numpy as np
import matplotlib.pyplot as plt
import sklearn
import sklearn.datasets
import sklearn.linear_model
from testCases import *
from planar_utils import load_planar_dataset, load_extra_datasets, sigmoid, plot_decision_boundary

np.random.seed(1)


# In[6]:


X, Y = load_planar_dataset()

# Visualize the data:
plt.scatter(X[0, :], X[1, :], c=Y, s=40, cmap=plt.cm.Spectral);


# In[8]:


# X.shape is (2, 400), Y.shape is (1, 400)
# the number of training dataset is 400, the number of features is 2
# means m = 400, n^[0] = 2


# In[15]:


# first see how logistic regression performs on this problem: 
clf = sklearn.linear_model.LogisticRegressionCV()
clf.fit(X.T, Y.T.reshape(-1))
print(clf.score(X.T, Y.T))  # 0.47

# Plot the decision boundary for logistic regression
plot_decision_boundary(lambda x: clf.predict(x), X, Y)
plt.title("Logistic Regression")


# In[16]:


# define the size of units
def layer_sizes(X, Y):
    n_x = X.shape[0]  # size of the Input Layer
    n_h = 4  # size of the Hidden Layer
    n_y = Y.shape[0]  # size of the Output Layer
    return (n_x, n_h, n_y)


# In[21]:


# initialize the model's parameters
def initialize_parameters(n_x, n_h, n_y):
    """
    RETURNS:
    params -- python dictionary containing your parameters:
        GIVEN X1 -- (n_x, m)
        W1 -- weight matrix of shape (n_h, n_x)
        b1 -- bias vector of shape (n_h, 1) 
        
        GIVEN X2 -- (n_h, m)
        W2 -- weight matrix of shape (n_y, n_h)
        b2 -- bias vector of shape (n_y, 1)
    """
    np.random.seed(2)
    
    # 生成0为均值、1为标准差的正态分布，再乘以0.01
    W1 = np.random.randn(n_h, n_x) * 0.01
    b1 = np.zeros(shape=(n_h, 1))
    W2 = np.random.randn(n_y, n_h) * 0.01
    b2 = np.zeros(shape=(n_y, 1))
    
    params = {
        'W1': W1,
        'b1': b1,
        'W2': W2,
        'b2': b2
    }
    return params


# In[24]:


# one loop forward propagation
def forward_propagation(X, parameters):
    """
    Arguments:
    X -- input data of size (n_x, m)
    parameters -- python dictionary containing your parameters (output of initialization function)
    
    Returns:
    A2 -- The sigmoid output of the second activation
    cache -- a dictionary containing "Z1", "A1", "Z2" and "A2"
    """
    # retrieve each parameter from the dictionary "parameters"
    W1 = parameters['W1']
    b1 = parameters['b1']
    W2 = parameters['W2']
    b2 = parameters['b2']
    
    # Implement forward propagation to calculate A2
    A0 = X
    Z1 = np.dot(W1, A0) + b1  # shape (n_h, m)
    A1 = np.tanh(Z1)  # shape (n_h, m)
    Z2 = np.dot(W2, A1) + b2  # shape (n_y, m)
    A2 = sigmoid(Z2)  # shape (n_y, m), means (1, m)
    
    assert(A2.shape == (1, X.shape[1]))
    
    cache = {
        'Z1': Z1,
        'A1': A1,
        'Z2': Z2,
        'A2': A2
    }
    return A2, cache


# In[36]:


# cost function
def compute_cost(A2, Y, parameters):
    """
    Arguments:
    A2 -- the sigmoid output of the second activation, of shape (1, #examples)
    Y -- "true" labels vector of shape (1, #examples)
    parameters -- python dictionary containing your parameters W1, b1, W2 and b2

    Returns:
    cost -- cross-entropy cost given equation
    """
    m = Y.shape[1]
    
    # retrieve W1 and b1
    W1 = parameters['W1']
    b1 = parameters['b1']
    
    # compute the cross-entropy cost
    cost = -np.sum(Y * (np.log(A2)) + (1 - Y) * (np.log(1 - A2))) / m
    
    cost = np.squeeze(cost)  # makes sure "cost" is the dimension we expect
    assert(isinstance(cost, float))
    
    return cost


# In[48]:


# one loop backward propagation
def backward_propagation(parameters, cache, X, Y):
    """
    Arguments:
    parameters -- python dictionary containing "W1", "b1", "W2" and "b2"
    cache -- python dictionary containing "Z1", "A1", "Z2" and "A2"
    X -- input data of shape (n_x, #examples)
    Y -- "true" labels vector of shape (n_y, #examples)
    
    Returns:
    grads -- python dictionary containing gradients with respect to different parameters
    """
    m = X.shape[1]
    
    W1 = parameters['W1']
    W2 = parameters['W2']
    A1 = cache['A1']
    A2 = cache['A2']
    
    # backward propagation: calculate dw1, db1, dw2, db2
    
    # 这里说明一下：dx 表示 Loss 对 x 求导
    # note: 为什么求dw_n时要除以m，因为代价函数前面有一个1/m，就是MSE前面那个1/m、求导时自动延续下来了
    # -- dA2 = (A2 - Y) / ((1 - A2) * A2)
    # -- (A2对Z2求导) = a - a^2
    # -- dZ2 = dA2 * (A2对Z2求导) = A2 - Y
    
    dZ2 = A2 - Y
    # shape of dZ2 is (n_y, m), shape of (Z2对W2求导) 即A1 is (n_h, m)
    # shape of dW2 is (n_y, n_h)
    dW2 = np.dot(dZ2, A1.T) / m  # dZ2 * (Z2对W2求导)
    # shape of db2 is (n_y, 1)
    db2 = np.sum(dZ2, axis=1, keepdims=True) / m
    
    # dZ1是难点：
    # np.power 不会改变维度
    # dZ2 = A2 - Y, which of shape (n_y, m)
    # Z2对A1求导 = W2, which of shape (n_y, n_h)
    # A1对Z1求导 = 1 - A1^2, which of shape (n_h, m)
    # dZ1 = dZ2 * (Z2对A1求导) * (A1对Z1求导), shape (n_h, m)
    dZ1 = np.dot(W2.T, dZ2) * (1 - np.power(A1, 2))
    # shape of dW1 is (n_h, n_x), shape of X is (n_x, m)
    dW1 = np.dot(dZ1, X.T) / m
    db1 = np.sum(dZ1, axis=1, keepdims=True) / m
    
    grads = {
        'dW1': dW1,
        'db1': db1,
        'dW2': dW2,
        'db2': db2
    }
    return grads


# In[53]:


# gradient descent
def update_parameters(parameters, grads, learning_rate=1.2):
    """
    Arguments:
    parameters -- python dictionary containing parameters
    grads -- python dictionary containing gradients
    
    Returns:
    parameters -- python dictionary containing your updated parameters
    """
    W1 = parameters['W1']
    b1 = parameters['b1']
    W2 = parameters['W2']
    b2 = parameters['b2']
    
    dW1 = grads['dW1']
    db1 = grads['db1']
    dW2 = grads['dW2']
    db2 = grads['db2']
    
    W1 = W1 - learning_rate * dW1
    b1 = b1 - learning_rate * db1
    W2 = W2 - learning_rate * dW2
    b2 = b2 - learning_rate * db2
    
    parameters = {
        'W1': W1,
        'b1': b1,
        'W2': W2,
        'b2': b2
    }
    return parameters


# In[61]:


# integrate parts
def nn_model(X, Y, n_h, num_iterations, print_cost=False):
    """
    Arguments:
    X -- dataset of shape (n_x, #examples)
    Y -- labels of shape (n_y, #examples)
    n_h -- size of hidden layer
    num_iterations -- Number of iterations in gradient descent loop
    print_cost -- if True, print the cost every 1000 iterations
    
    Returns:
    parameters -- parameters learnt by the model. They can then be used by predict.
    """
    np.random.seed(3)
    n_x = layer_sizes(X, Y)[0]
    n_y = layer_sizes(X, Y)[2]
    
    # initialize parameters
    parameters = initialize_parameters(n_x, n_h, n_y)
    W1 = parameters['W1']
    b1 = parameters['b1']
    W2 = parameters['W2']
    b2 = parameters['b2']
    
    # Loop (gradient descent)
    for i in range(0, num_iterations):
        # Forward Propagation. Inputs: "X, parameters". Outputs: "A2, cache".
        A2, cache = forward_propagation(X, parameters)
        
        # Cost function. Inputs: "A2, Y, parameters". Outputs: "cost".
        cost = compute_cost(A2, Y, parameters)
        
        # Backward Propagation. Inputs: "parameters, cache, X, Y". Outputs: "grads".
        grads = backward_propagation(parameters, cache, X, Y)
        
        # Gradient Descent parameter update. Inputs: "parameters, grads". Outputs: "parameters".
        parameters = update_parameters(parameters, grads)
        
        if (print_cost and i % 1000 == 0):
            print("Cost after iteration %i: %f" % (i, cost))  # i(int), f(float)
    
    return parameters


# In[86]:


# Predictions
def predict(parameters, X):
    A2, cache = forward_propagation(X, parameters)
    predictions = np.array(A2 > 0.5, dtype=int)  # 或者用np.round(A2)，它返回float类型
    return predictions


# In[93]:


# Build a model with a n_h-dimensional hidden layer
parameters = nn_model(X, Y, n_h = 4, num_iterations=10000, print_cost=True)

# Plot the decision boundary
plot_decision_boundary(lambda x: predict(parameters, x.T), X, Y)
plt.title("Decision Boundary for hidden layer size " + str(4))


# In[97]:


predictions = predict(parameters, X)
print(np.sum(np.array(predictions == Y, dtype=int)) / Y.shape[1])

# 老师是这么写的：
print ('Accuracy: %d' % float((np.dot(Y, predictions.T) + np.dot(1 - Y, 1 - predictions.T)) / float(Y.size) * 100) + '%')
# 计算的是：(都为1时的数量 + 都为0时的数量) / len


# In[100]:


# Tuning hidden layer size to observe different behaviors of the model for various hidden layer sizes

plt.figure(figsize=(16, 32))
hidden_layer_sizes = [1, 2, 3, 4, 5, 20, 50]
for i, n_h in enumerate(hidden_layer_sizes):
    plt.subplot(5, 2, i + 1)
    plt.title('Hidden Layer of size %d' % n_h)
    parameters = nn_model(X, Y, n_h, num_iterations=5000)
    plot_decision_boundary(lambda x: predict(parameters, x.T), X, Y)
    predictions = predict(parameters, X)
    accuracy = float((np.dot(Y, predictions.T) + np.dot(1 - Y, 1 - predictions.T)) / float(Y.size) * 100)
    print ("Accuracy for {} hidden units: {} %".format(n_h, accuracy))


# In[ ]:



```

## 作业4: Building a deep neural network

### 代码

```python
import numpy as np
import matplotlib.pyplot as plt

np.random.seed(1)


def initialize_parameters(layer_dims):
    """
    Random Initialize Parameters.
    :param layer_dims: Python list containing the dimensions of each layer, including the Input Layer
                e.g. [3, 4, 5, 1], means 2 hidden layers
    :return: parameters -- Python dictionary containing parameters:
                W1 -- weight matrix of shape (n1, n0)
                b1 -- bias vector of shape (n1, 1)
                ..
                WL -- weight matrix of shape (nl, nl-1)
                bL -- bias vector of shape (nl, 1)
    """
    parameters = {}
    L_all = len(layer_dims)

    for l in range(1, L_all):
        parameters['W' + str(l)] = np.random.randn(
            layer_dims[l], layer_dims[l - 1]) / np.sqrt(
            layer_dims[l - 1])  # randn() returns the "standard normal" distribution
        parameters['b' + str(l)] = np.zeros(shape=(layer_dims[l], 1))

    return parameters


def forward_propagation(X, parameters):
    """
    Implement forward propagation for the [LINEAR->RELU]*(L-1) -> LINEAR -> SIGMOID computation.
    :param X: feature data, numpy array of shape (n0, m), n0 is size of input layer
    :param parameters: Python list containing all Ws and bs
    :return: AL, caches:
                AL -- last post-activation value, that is y_hat
                caches -- Python list of caches containing:
                            every cache of ReLU forward (indexed from 1 to L-1)
                            the cache of Sigmoid forward (indexed L)
                            cache[0] is empty Python tuple
    """
    caches = []
    L = len(parameters) // 2

    Al_prev = X

    def relu(Z):
        A = np.maximum(0, Z)
        return A

    def sigmoid(Z):
        A = 1 / (1 + np.exp(-Z))
        return A

    # cache of Input Layer is empty Python tuple
    caches.append(())

    # Wl of shape (nl, n(l-1)), A(l-1) of shape (n(l-1), m), bl of shape (nl, 1), Zl of shape (nl, m)
    for l in range(1, L + 1):
        Wl = parameters['W' + str(l)]
        bl = parameters['b' + str(l)]
        Zl = np.dot(Wl, Al_prev) + bl
        linear_cache = (Al_prev, Wl, bl)
        activation_cache = Zl
        cache = (linear_cache, activation_cache)
        caches.append(cache)
        # layer L, that is last layer if l != L
        Al_prev = relu(Zl) if l != L else sigmoid(Zl)

    AL = Al_prev

    m = X.shape[1]
    assert AL.shape == (1, m)

    return AL, caches


def compute_cost(AL, Y):
    """
    Compute cost.
    :param AL: probability vector corresponding to label predictions, of shape (1, m)
    :param Y: "true" label vector, of shape (1, m)
                e.g. containing 0 if non-cat, 1 if cat
    :return: cross-entropy cost. Python float
    """
    assert AL.shape == Y.shape
    m = AL.shape[1]

    AL[AL == 0] = 0.00000001
    AL[AL == 1] = 0.99999999

    # cost = -np.sum(Y * np.log(AL) + (1 - Y) * np.log(1 - AL)) / m
    cost = (-1 / m) * np.sum(np.multiply(Y, np.log(AL)) + np.multiply(1 - Y, np.log(1 - AL)))
    return cost


def backward_propagation(AL, Y, caches):
    """
    Implement the backward propagation for the [LINEAR->RELU] * (L-1) -> LINEAR -> SIGMOID group.
    :param AL: probability vector corresponding to label predictions, of shape (1, m)
    :param Y: "true" label vector, of shape (1, m)
    :param caches: Python list of caches containing:
                            every cache of ReLU forward (indexed from 1 to L-1)
                            the cache of Sigmoid forward (indexed L)
    :return: gradients -- Python dictionary with the gradients
    """
    gradients = {}
    dAL = (1 - Y) / (1 - AL) - Y / AL
    L = len(caches) - 1
    m = AL.shape[1]

    def d_relu(dA, Z):
        dZ = np.array(dA, copy=True)
        dZ[Z <= 0] = 0
        return dZ

    def d_sigmoid(dA, Z):
        A = 1 / (1 + np.exp(-Z))
        dZ = (A - A * A) * dA
        return dZ

    dAl = dAL
    # Wl of shape (nl, n(l-1)), A(l-1) of shape (n(l-1), m), bl of shape (nl, 1), Zl of shape (nl, m)
    for l in range(L, 0, -1):
        linear_cache, activation_cache = caches[l]
        Al_prev, Wl, bl = linear_cache
        Zl = activation_cache
        dZl = d_relu(dAl, Zl) if l != L else d_sigmoid(dAl, Zl)

        gradients['dW' + str(l)] = np.dot(dZl, Al_prev.T) / m
        gradients['db' + str(l)] = np.sum(dZl, axis=1, keepdims=True) / m

        dAl = np.dot(Wl.T, dZl)

    return gradients


def update_parameters(parameters, gradients, learning_rate):
    """
    Update parameters using gradient descent.
    :param parameters: Python list containing all Ws and bs
    :param gradients: Python dictionary with the gradients
    :param learning_rate:
    :return: updated parameters
    """
    L = len(parameters) // 2

    for l in range(1, L + 1):
        parameters['W' + str(l)] = parameters['W' + str(l)] - gradients['dW' + str(l)] * learning_rate
        parameters['b' + str(l)] = parameters['b' + str(l)] - gradients['db' + str(l)] * learning_rate

    return parameters


def L_layer_model(X, Y, layer_dims, learning_rate=0.0075, num_iterations=3000, print_cost=False):
    """
    L layers neural network.
    :param X:
    :param Y:
    :param layer_dims:
    :param learning_rate:
    :param num_iterations:
    :param print_cost:
    :return: parameters -- parameters learnt by the model
    """
    costs = []
    parameters = initialize_parameters(layer_dims)

    for i in range(num_iterations):
        AL, caches = forward_propagation(X, parameters)
        cost = compute_cost(AL, Y)
        grads = backward_propagation(AL, Y, caches)
        parameters = update_parameters(parameters, grads, learning_rate=learning_rate)
        if print_cost and i % 100 == 0:
            print("Cost after iteration %i: %f" % (i, cost))
            costs.append(cost)

    # plot the cost
    plt.plot(np.squeeze(costs))
    plt.ylabel('cost')
    plt.xlabel('iterations (per tens)')
    plt.title("Learning rate =" + str(learning_rate))
    plt.show()

    return parameters


def score(X, Y, parameters):
    AL, caches = forward_propagation(X, parameters)
    AL_trans = np.array(AL > 0.5, dtype=int)
    return np.sum(AL_trans == Y) * 1.0 / Y.shape[1]


from dnn_app_utils_v2_2 import load_data

# train_x_orig of shape (209, 64, 64, 3), train_y of shape (1, 209)
train_x_orig, train_y, test_x_orig, test_y, classes = load_data()

# Reshape the training and test examples
train_x_flatten = train_x_orig.reshape(train_x_orig.shape[0],
                                       -1).T  # The "-1" makes reshape flatten the remaining dimensions
test_x_flatten = test_x_orig.reshape(test_x_orig.shape[0], -1).T

# Standardize data to have feature values between 0 and 1.
train_X = train_x_flatten / 255
test_X = test_x_flatten / 255

m_train = train_x_orig.shape[0]
num_px = train_x_orig.shape[1]
m_test = test_x_orig.shape[0]

train_x_flatten = train_x_orig.reshape(train_x_orig.shape[0],
                                       -1).T  # The "-1" makes reshape flatten the remaining dimensions
test_x_flatten = test_x_orig.reshape(test_x_orig.shape[0], -1).T

# Standardize data to have feature values between 0 and 1.
train_x = train_x_flatten / 255.
test_x = test_x_flatten / 255.

layers_dims = [12288, 20, 7, 5, 1]  # 5-layer model

params = L_layer_model(train_x, train_y, layers_dims, num_iterations=3000, print_cost=True)
print(score(train_x, train_y, params))
print(score(test_X, test_y, params))

```

