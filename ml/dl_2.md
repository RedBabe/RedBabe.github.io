## 1. Setting up your ML Application

### 训练，开发，测试数据集(train/dev/test sets)

训练数据集(train set)：训练算法

开发数据集(development set)：评估不同算法

传统分割比例是60/20/20，但根据数据规模的大小，后两者的比例可以非常非常小。

**重要准则：确保开发集和测试集的数据分布相同**

#### 另外

很多时候，数据集只有训练和开发，也是Ok的。此时很多人会称两个集合为训练集和测试集。

### 偏差和方差(Bias / Variance)

很少人讨论偏差方差权衡(bias-variance trade-off)。

在二维度时，可以通过画图很容易看出数据是否是欠拟合(underfitting)或过拟合(overfitting)。

比如一条直线，不能很好的把数据分为两类，是欠拟合；比如它的预测准确率(accuracy rate)大概是：训练集80%，测试集79%；此时是**高偏差**的；

比如一条极其弯曲的线，把所有训练数据无一例外的正确分为两类，是过拟合；比如它的预测准确率大概是训练集96%，测试集79%；此时是**高方差**的。

所以对于现实的数据，不可能只有2个特征的情况下，可以**观察其训练集的准确率和测试集的准确率情况**。

> 如果train_score为15%，test_score为30%，则为high bias & high variance；
>
> 如果train_score为0.5%，test_score为1%，则为low bias & low variance。

高偏差时，可以通过改善模型；

高方差时，可以通过增大数据集或正则化。

### 正则化(Regularization)

#### 逻辑回归中的正则化

最常用的是L2正则化，即：
$$
代价函数J = 
\frac1m \sum_{i=1}^{m} Loss(y^{(i)}, \hat{y}^{(i)})
+ \frac{\lambda}{2m} \parallel {w} \parallel^2 \\
其中 \parallel w \parallel ^ 2 = \sum_{j=1}^{n} w_j^2 = w^Tw
$$
L2正则化也称作岭回归(Ridge Regression)。

L1正则化(LASSO)就是去掉二次方符号，它在实际使用中并没有什么显著效果。

#### 神经网络中的正则化

它不叫L2范数，而是叫做弗罗贝尼乌斯范数(Frobenius norm)。
$$
代价函数J(W^{[1]}, b^{[1]}, .., W^{[L]}, b^{[L]}) = \\
\frac1m \sum_{i=1}^{m} Loss(y^{(i)}, \hat{y}^{(i)})
+ \frac{\lambda}{2m} \sum_{L} \parallel {w^{[L]}} \parallel^2 \\
其中 \parallel {w^{[L]}} \parallel^2 = \sum_{i=1}^{n^{[L-1]}} \sum_{j=1}^{n^L} (w_{ij}^{[L]})^2 \\
这里的w不光是一组参数，它是多组参数，所以会有i和j两个维度。
$$


这里的代价函数后半部分怎么求导呢？
$$
没看懂
$$

### Drop-out Regularization

> 神经元随机失活？大概是这么翻译。

以概率p保留部分神经元。目的是：简化神经网络的复杂度，降低过拟合。

drop-out被广泛应用于计算机视觉领域(Computer Vision)，因为其输入层向量维度非常大，因为要包含没个像素点的值。

但在其它领域，请记住，drop-out是一项正则化技术，如果没有发生过拟合，就没必要使用。

### 其它正则化方法(other regularization methods)

#### 扩大数据集(data augmentation)

比如图像识别，如果不能增加数据集，可以将图片**水平翻转**，或者**旋转、放大**。

#### 提早终止(early stopping)

绘制出训练集损失和开发集损失的图形。它们开始会一起下降，然后，训练数据集损失下降更多，而开发集损失下降变小；此时停止模型训练，选取此时的开发集误差的值。

### 数据归一化(Normalizing inputs)

$$
1. 减去均值(subtract \space mean) \\
\mu = \frac1m \sum_{i=1}^m x^{(i)} \\
x = x - \mu \\
2. 标准化方差(normalize \space variance) \\
\sigma^2 = \frac1m \sum_{i=1}^m (x^{(i)})^2 \\
x = x / \sigma^2
$$

无论数据集的特征值的范围如何，都可以使用数据归一化。

当不同特征的值有很大差异时，一定要使用数据归一化。

### 梯度消失与梯度爆炸(Vanishing/Exploding gradients)

比如W为1.1时，y_hat的值会指数级增加；W为0.9时，y_hat的值为指数级减少。同样，其梯度也会指数级增加或减少。

解决方案：让w初始值为1/n。如果是ReLU，设为2/n。

不是特别懂。

### 梯度检验(Gradient Checking)

> 用来验证反向传播是否是正确的。

用标准的求导定义，求出梯度。然后跟求解的梯度进行比较欧几里得距离。

如果小于$10^{-7}$，说明没问题；

如果在$10^{-7} ～10^{-5}$，说明可能有错误；

如果大于$10^{-3}$，一定有错误。

注意：

1. 只在debug时使用，不要在训练或预测时使用；
2. 正则化；
3. 不要和dropout一起使用。

## 2. Optimization Algorithms

### 小批量梯度下降法(Mini-batch Gradient Descent)

比如有5百万条数据，以1000为1个批量，则共有5000个批量。

> 小括号代表第i个样例，中括号表示第l层，大括号表示第t个批量。

$$
X: 5billion \\
X^{\{1\}}: 1 - 1000个数据 \\
$$

#### 极限情况

设m为样本总大小。

如果 mini-batch size 为 m，则称为批量梯度下降(batch g d)，就是以前学的那个，对整体进行梯度下降。如果m过大，每次迭代的速度会很慢。

如果 mini-batch size 为 1，则称为随机梯度下降(stochastic g d)，每次每次对一个维度进行梯度下降。它的缺点是：失去了向量计算的加速优势。

#### 选择尺寸(choosing your mini-batch size)

设m为样本总大小。

如果m小于2000，可以直接选择批量梯度下降。

通常，mini-batch size 可以选择为 $2^6 - 2^9$，即64、128、256或512。使用2的幂可以让计算机的计算速度更快。

它也是一个超参数。

### 指数加权平均(Exponentially weighted averages)

> 一种常用的序列数据处理方式。它是一种近似求平均的方法。
>
> 时间越近，权重越大，而且是指数式的，所以叫做加权平均。

$$
v_0 = b \\
v_t = \beta v_{t-1} + (1 - \beta)\theta_{t} \\
v_t是指数加权平均值，\\
v_{t-1}是上一次的指数加权平均值，\\
\theta_t是第t天的观察值，\\
\beta是权重。
$$

#### 偏差修正

$$
计算v_t后，\\
v_t(correct) = \frac{v_t}{1 - \beta^t}
$$



不是特别懂。

通常可以将$v_0$的值设置为一个大概的偏差值，而不是0。这样就可以不做偏差修正了。

### 动量梯度下降法(Gradient descent with momentum)

> 通过计算梯度的指数加权平均，代替梯度来使用，可以使得梯度下降更快。
>
> 原理是：
>
> 1. 设从左往右是梯度下降的方向；
> 2. 在梯度下降的过程中，梯度会上下震荡；使用指数加权平均，使得竖直方向的上下震荡会抵消，水平方向会叠加。

$$
v_{dw} = \beta v_{dw} + (1 - \beta) dw \\
v_{db} = \beta v_{db} + (1 - \beta) db \\
W = W - \alpha v_{dw} \\
b = b - \alpha v_{db} \\
\alpha和\beta都是超参数。\beta很稳健的值是0.9。
$$

### RMSprop(Root Mean Square prop)

> 大概叫均方根传递。
>
> 动量算法的$\beta$写作$\beta_1$，RMSprop的$\beta$写作$\beta_2$。

实现：
$$
s_{dw} = \beta s_{dw} + (1-\beta)dw^2 \\
s_{db} = \beta s_{db} + (1-\beta)db^2 \\
W = W - \alpha \frac{dw}{\sqrt{s_{dw}} + \epsilon} \\
b = b - \alpha \frac{db}{\sqrt{s_{db}} + \epsilon} \\
\epsilon = 10e-8, 它用来防止分母等于0
$$
说明：
$$
从下降曲线可以得知（虽然我没保存），db是大于dw的，所以s_{db}>s_{dw} \\
所以b的变化程度比W会小，这样就降低了b的震荡程度。
$$


### Adam(Adam optimization algorithm)

> 它是动量算法和RMSprop的结合。

实现：
$$
v_{dw} = \beta v_{dw} + (1 - \beta_1) dw \\
v_{db} = \beta v_{db} + (1 - \beta_1) db \\
s_{dw} = \beta s_{dw} + (1-\beta_2)dw^2 \\
s_{db} = \beta s_{db} + (1-\beta_2)db^2 \\
W = W - \alpha \frac{v_{dw}}{\sqrt{s_{dw}} + \epsilon} \\
b = b - \alpha \frac{v_{db}}{\sqrt{s_{db}} + \epsilon} \\
$$
它有 $\alpha, \beta_1, \beta_2, \epsilon$ 4个超参数。

通常都会设置：
$$
\beta_1 = 0.9, \beta_2 = 0.999, \epsilon = 10^{-8}
$$
然后去调试$\alpha$这个超参数。

### 学习率衰减(learning rete decay)

由于采取了 mini-batch gradient descent，而且学习率是固定值，所以梯度可以不会收敛于一点，而是在附近摆动。

学习率递减的公式有多种，没细讲，这里就不写了。

### 局部最优

在高维空间(比如20000维)中，出现局部最优的点的概率大概是$2^{-20000}$。

所以更可能出现的是鞍点(saddle point)。

## 3. Hyperparameter tuning

### 调试处理(Tuning process)

众多超参数中的重要程度排序：

1. $\alpha$(learning rate)
2. $\beta$(momentum=0.9)、#hidden units、mini-batch size
3. #layers、learning rate decay
4. $\beta_1$(momentum=0.9)、$\beta_2$(RMSprop=0.999)、$\epsilon$(error=10e-8)

#### 寻找超参数

1. 随机搜索。不要使用网格搜索
2. 区域定位，就是说，哪一个值还不错，就在它附近搜索

比如找 learning rate 这个超参数，可以从1、0.1、0.01、0.001这样的范围分别进行随机测试。

### 正则化激活函数(Normalizing activations in a network)

在逻辑回归中，我们会对X进行正则化处理：
$$
均值 \mu = \frac1m \sum_i X^{(i)} \\
方差 \sigma^2 = \frac1m \sum_i {X^{(i)}}^2 \\
X = X - \mu \\
X = X / \sigma^2
$$
在神经网络中，更常见的方式是对Z进行正则化(batch norm)、而不是A。

#### Batch Norm

$$
\gamma \tilde{d} \\
\mu = \frac1m \sum_i Z^{(i)} \\
\sigma^2 = \frac1m \sum_i {Z^{(i)}}^2 \\
\tilde{Z}^{(i)}_{norm} = \frac{Z^{(i)} - \mu}{\sqrt{\sigma^2 + \epsilon}} \\
\tilde{Z}^{(i)} = \gamma \tilde{Z}^{(i)}_{norm} + \beta, 
\gamma 和 \beta 用来使隐藏单元有可控的方差和均值。
$$

因为 batch norm 会对Z做均值处理，所以如果使用了 batch norm，则 b 这个参数可以省略。

在更新参数时：
$$
W^{[l]} = W^{[l]} - \alpha {dw}^{[l]} \\
\beta^{[l]} = \beta^{[l]} - \alpha {d\beta}^{[l]} \\
\gamma^{[l]} = \gamma^{[l]} - \alpha {d\gamma}^{[l]}
$$

### 测试时的batch norm

测试时，数据集可能只有1个，那么此时的均值和方差的计算就需要用新的方法。

将训练时的移动加权平均值记录下来，作为测试时的均值和方差。

### Softmax Regression

之前学的都是二分类问题，softmax 回归用于解决多分类问题：
$$
Z^{[L]} = W^{[L]}A^{[L-1]} + b^{[L]} \\
t^{[L]} = e^{Z^{[L]}} \\
A^{[L]} = \frac{t^{[L]}}{\sum_it^{[L]}} \\
A^{[L]}_i = \frac{t_i^{[L]}}{\sum_it^{[L]}}
$$

##### 损失函数

softmax 的损失函数是：
$$
Loss = -\sum_j y_j log \hat{y_j} \\
Cost = -\frac1m \sum_i Loss
$$

## 作业1: 初始参数的设置

### Initialization


```python
import numpy as np
import matplotlib.pyplot as plt
from init_utils import compute_loss, forward_propagation, backward_propagation
from init_utils import update_parameters, predict, predict_dec, load_dataset, plot_decision_boundary

plt.rcParams['figure.figsize'] = (7.0, 4.0)  # set default size of plots
plt.rcParams['image.interpolation'] = 'nearest'
plt.rcParams['image.cmap'] = 'gray'

# load data
train_X, train_Y, test_X, test_Y = load_dataset()

"""shape of train_X is = (2, 300), shape of train_Y is (1, 300)"""


def model(X, Y, learning_rate=0.01, num_iterations=15000, print_cost=True, initialization='he'):
    """
    Implements a three-layer neural network, LINEAR->RELU->LINEAR->RELU->LINEAR->SIGMOID

    Arguments:
    X -- input data, of shape (2, #examples)
    Y -- "true" label vector, (containing 0 for red dots, 1 for blue dots), of shape (1, m)
    learning_rate -- learning rate for gradient descent
    num_iterations -- number of iterations to run gradient descent
    print_cost -- if True, print the cost every 1000 iterations
    initialization -- flag to choose which initialization to use ("zeros", "random" or "he")

    Returns:
    parameters -- parameters learnt by the model
    """
    grads = {}
    costs = []  # to keep track of the loss
    m = X.shape[1]
    layers_dims = [X.shape[0], 10, 5, 1]

    # Initialize parameters dictionary
    if initialization == 'zeros':
        parameters = initialize_parameters_zeros(layers_dims)
    elif initialization == 'random':
        parameters = initialize_parameters_random(layers_dims)
    elif initialization == 'he':
        parameters = initialize_parameters_he(layers_dims)

    # Loop: gradient descent
    for i in range(num_iterations):
        # Forward propagation
        a3, cache = forward_propagation(X, parameters)

        # Loss
        cost = compute_loss(a3, Y)

        # Backward propagation
        grads = backward_propagation(X, Y, cache)

        # Update parameters
        parameters = update_parameters(parameters, grads, learning_rate)

        # print the loss every 1000 iterations
        if print_cost and i % 1000 == 0:
            print('Cost after iteration {}: {}'.format(i, cost))
            costs.append(cost)

    # plot the loss
    plt.plot(costs)
    plt.ylabel('cost')
    plt.xlabel('iterations (per hundreds)')
    plt.title('Learning rate = ' + str(learning_rate))
    plt.show()

    return parameters


# Zero initialization
# You'll see later that it does not work well
def initialize_parameters_zeros(layers_dims):
    """
    Arguments:
    layers_dims: Python list containing the size of each layer

    Returns:
    parameters -- Python dictionary containing parameters "W1", "b1", ..., "WL", "bL"
                Wl -- weight matrix of shape (layers_dims[l], layers_dims[l-1])
                bl -- bias vector of shape (layers_dims[l], 1)
    """
    parameters = {}
    L = len(layers_dims)

    for l in range(1, L):
        parameters['W' + str(l)] = np.zeros((layers_dims[l], layers_dims[l - 1]))
        parameters['b' + str(l)] = np.zeros((layers_dims[l], 1))

    return parameters


parameters = model(train_X, train_Y, initialization="zeros")
print("On the train set:")
predictions_train = predict(train_X, train_Y, parameters)
print("On the test set:")
predictions_test = predict(test_X, test_Y, parameters)


# Random initialization
def initialize_parameters_random(layers_dims):
    """
    Arguments:
    layers_dims: Python list containing the size of each layer

    Returns:
    parameters -- Python dictionary containing parameters "W1", "b1", ..., "WL", "bL"
                Wl -- weight matrix of shape (layers_dims[l], layers_dims[l-1])
                bl -- bias vector of shape (layers_dims[l], 1)
    """
    np.random.seed(3)
    parameters = {}
    L = len(layers_dims)

    for l in range(1, L):
        parameters['W' + str(l)] = np.random.randn(layers_dims[l], layers_dims[l - 1]) * 10
        parameters['b' + str(l)] = np.zeros((layers_dims[l], 1))

    return parameters


parameters = model(train_X, train_Y, initialization="random")
print("On the train set:")
predictions_train = predict(train_X, train_Y, parameters)
print("On the test set:")
predictions_test = predict(test_X, test_Y, parameters)

print(predictions_train)
print(predictions_test)
plt.title("Model with large random initialization")
axes = plt.gca()
axes.set_xlim([-1.5, 1.5])
axes.set_ylim([-1.5, 1.5])
plot_decision_boundary(lambda x: predict_dec(parameters, x.T), train_X, train_Y)


# He initialization
def initialize_parameters_he(layers_dims):
    """
    Arguments:
    layers_dims: Python list containing the size of each layer

    Returns:
    parameters -- Python dictionary containing parameters "W1", "b1", ..., "WL", "bL"
                Wl -- weight matrix of shape (layers_dims[l], layers_dims[l-1])
                bl -- bias vector of shape (layers_dims[l], 1)
    """
    np.random.seed(3)
    parameters = {}
    L = len(layers_dims)

    for l in range(1, L):
        parameters['W' + str(l)] = np.random.randn(layers_dims[l], layers_dims[l - 1]) * np.sqrt(2 / layers_dims[l - 1])
        parameters['b' + str(l)] = np.zeros((layers_dims[l], 1))

    return parameters


parameters = model(train_X, train_Y, initialization="he")
print("On the train set:")
predictions_train = predict(train_X, train_Y, parameters)
print("On the test set:")
predictions_test = predict(test_X, test_Y, parameters)

print(predictions_train)
print(predictions_test)
plt.title("Model with large random initialization")
axes = plt.gca()
axes.set_xlim([-1.5, 1.5])
axes.set_ylim([-1.5, 1.5])
plot_decision_boundary(lambda x: predict_dec(parameters, x.T), train_X, train_Y)

```

## 作业2: 通过L2正则化和dropout优化方差问题

```python
import numpy as np
import matplotlib.pyplot as plt
from reg_utils import sigmoid, relu, plot_decision_boundary, initialize_parameters, load_2D_dataset, predict_dec
from reg_utils import compute_cost, predict, forward_propagation, backward_propagation, update_parameters
import sklearn
import sklearn.datasets
import scipy.io
from testCases import *

train_X, train_Y, test_X, test_Y = load_2D_dataset()

def model(X, Y, learning_rate=0.3, num_iterations=30000, print_cost=True, lambd=0, keep_prob=1):
    """
    Implements a three-layer neural network: LINEAR->RELU->LINEAR->RELU->LINEAR->SIGMOID
    
    Arguments:
    X -- input data, of shape (input size, m)
    Y -- true "label" vector (0 for blue dot, 1 for red dot), of shape (output size, m)
    learning_rate -- learning rate of the optimization
    num_iterations -- number of iterations of the optimization loop
    print_cost -- print the cost every 1000 iterations
    lambd -- regularization hyperparameters, scalar
    keep_prob -- probability of keeping a neuron active during drop-out
    
    Returns:
    parameters -- parameters learned by the model. They can then be used to predict
    """
    
    grads = {}
    costs = []
    m = X.shape[1]
    layers_dims = [X.shape[0], 20, 3, 1]
    
    parameters = initialize_parameters(layers_dims)
    
    """
    正则化是对损失函数进行操作，所以它会影响到 compute_cost 和 backward
    随机失活是对神经元进行操作，所以它会影响到 forward 和 backward
    """
    # Loop for the gradient descent
    for i in range(num_iterations):
        # Forward propagation
        if keep_prob == 1:
            A3, cache = forward_propagation(X, parameters)
        elif keep_prob < 1:
            A3, cache = forward_propagation_with_dropout(X, parameters, keep_prob)
            
        # Cost functon
        if lambd == 0:
            cost = compute_cost(A3, Y)
        else:
            # L2正则化
            cost = compute_cost_with_regularization(A3, Y, parameters, lambd)
            
        # Backward propagation
        assert(lambd == 0 or keep_prob == 1)  # it's possible to use both, but not in this assignment
        
        if lambd == 0 and keep_prob == 1:
            grads = backward_propagation(X, Y, cache)
        elif lambd != 0:
            grads = backward_propagation_with_regularization(X, Y, cache, lambd)
        elif keep_prob < 1:
            grads = backward_propagation_with_dropout(X, Y, cache, keep_prob)
        
        # Update parameters
        parameters = update_parameters(parameters, grads, learning_rate)
        
        # Print cost
        if print_cost and i % 1000 == 0:
            print("Cost after iteration {}: {}".format(i, cost))
            costs.append(cost)
            
    # plot the cost
    plt.plot(costs)
    plt.ylabel('cost')
    plt.xlabel('iterations (x1,000)')
    plt.title("Learning rate =" + str(learning_rate))
    plt.show()
    
    return parameters
  
  
parameters = model(train_X, train_Y)
print("On the training set:")
predictions_train = predict(train_X, train_Y, parameters)
print("On the test set:")
predictions_test = predict(test_X, test_Y, parameters)

plt.title("Model without regularization")
axes = plt.gca()
axes.set_xlim([-0.75, 0.40])
axes.set_ylim([-0.75, 0.65])
plot_decision_boundary(lambda x: predict_dec(parameters, x.T), train_X, train_Y)

# L2 Regularization

# 正则化cost的计算：交叉熵cost + 正则化cost
# 正则化cost = 所有W^2的和 * lambd / (2 * m)
def compute_cost_with_regularization(AL, Y, parameters, lambd):
    """
    Implements the cost function with regularization.
    
    Arguments:
    AL -- post-activation, output of forward propagation, of shape (output size, m)
    Y -- "true" labels vector, of shape (output size, m)
    parameters -- Python dictionary containing parameters of the model
    lambd -- regularization hyperparameter, scalar
    
    Returns:
    cost -- value of the regularized cost function
    """
    m = Y.shape[1]
    L = len(parameters) // 2
    
    cross_entropy_cost = compute_cost(AL, Y)
    
    reg_cost = 0
    for l in range(L):
        reg_cost += np.sum(np.square(parameters['W' + str(l + 1)]))
        
    L2_regularization_cost = reg_cost * lambd / (2 * m)
    
    cost = cross_entropy_cost + L2_regularization_cost
    
    return cost
  
def backward_propagation_with_regularization(X, Y, cache, lambd):
    """
    Implements the backward propagation of our baseline model to which we added an L2 regularization.
    
    Arguments:
    X -- input dataset, of shape (input size, m)
    Y -- "true" label vector, of shape (output size, m)
    cache -- cache output from forward_propagation()
    lambd -- regularization hyperparameter, scalar
    
    Returns:
    gradients -- A dictionary with the gradients with respect to each parameter, activation and pre-activation variables
    """
    
    """
    梯度的求法：
    grads = cross_entropy_grads + reg_grads
    对最后一层进行求导：
    dAL = (1-Y) / (1-AL) - Y / AL
    dZL = d_sigmoid(dAL) = (A - A * A) * dAL = AL - Y
    dWL = dZL * X + d_reg_grads 
            = (AL - Y) * X / m + lambd / (2 * m) * (2 * WL) 
            = (AL - Y) * X / m + lambd * WL / m
    """
    grads = {}
    L = len(cache)
    (Z1, A1, W1, b1, Z2, A2, W2, b2, Z3, A3, W3, b3) = cache
    m = Y.shape[1]
    
    
    dZ3 = A3 - Y  # shape of (n3, m)
    # A2's shape: (n2, m)
    dW3 = np.dot(dZ3, A2.T) / m + lambd * W3 / m  # shape of (n3, n2)
    db3 = np.sum(dZ3, axis=1, keepdims=True) / m  # shape of (n3, 1)
    
    dA2 = np.dot(W3.T, dZ3)  # shape of (n2, m)
    dZ2 = np.multiply(dA2, np.int64(A2 > 0))
    dW2 = 1. / m * np.dot(dZ2, A1.T) + (lambd * W2) / m
    db2 = 1. / m * np.sum(dZ2, axis=1, keepdims=True)
    
    dA1 = np.dot(W2.T, dZ2)
    dZ1 = np.multiply(dA1, np.int64(A1 > 0))
    dW1 = 1. / m * np.dot(dZ1, X.T) + (lambd * W1) / m
    db1 = 1. / m * np.sum(dZ1, axis=1, keepdims=True)
    
    gradients = {"dZ3": dZ3, "dW3": dW3, "db3": db3, "dA2": dA2,
                 "dZ2": dZ2, "dW2": dW2, "db2": db2, "dA1": dA1, 
                 "dZ1": dZ1, "dW1": dW1, "db1": db1}
    
    return gradients
  
parameters = model(train_X, train_Y, lambd=0.7)
print("On the train set:")
predictions_train = predict(train_X, train_Y, parameters)
print("On the test set:")
predictions_test = predict(test_X, test_Y, parameters)

plt.title("Model with L2-regularization")
axes = plt.gca()
axes.set_xlim([-0.75,0.40])
axes.set_ylim([-0.75,0.65])
plot_decision_boundary(lambda x: predict_dec(parameters, x.T), train_X, train_Y)

# Dropout

def forward_propagation_with_dropout(X, parameters, keep_prob=0.5):
    """
    Implements the forward propagation: LINEAR -> RELU + DROPOUT -> LINEAR -> RELU + DROPOUT -> LINEAR -> SIGMOID.
    
    Arguments:
    X -- input dataset, of shape (2, m)
    parameters -- Python dictionary containing your parameters "W1", "b1", "W2", "b2", "W3", "b3"
    keep_prob -- probability of keeping a neuron active during drop-out, scalar
    
    Returns:
    A3 -- last activation value, output of the forward propagation, of shape (1, 1)
    cache -- tuple, information stored for computing the backward propagation
    """
    np.random.seed(1)
    
    # retrieve parameters
    W1 = parameters["W1"]
    b1 = parameters["b1"]
    W2 = parameters["W2"]
    b2 = parameters["b2"]
    W3 = parameters["W3"]
    b3 = parameters["b3"]
    
    Z1 = np.dot(W1, X) + b1
    A1 = relu(Z1)
    D1 = np.random.rand(A1.shape[0], A1.shape[1])  # step 1: initialize matrix D1
    D1 = D1 < keep_prob  # step 2: convert entries of D1 to 0 or 1
    A1 = A1 * D1  # step 3: shut down some neurons of A1
    A1 = A1 / keep_prob  # step 4: scale the value of neurons that haven't been shut down
    
    Z2 = np.dot(W2, A1) + b2
    A2 = relu(Z2)
    D2 = np.random.rand(A2.shape[0], A2.shape[1])
    D2 = D2 < keep_prob
    A2 = A2 * D2
    A2 = A2 / keep_prob
    
    Z3 = np.dot(W3, A2) + b3
    A3 = sigmoid(Z3)
    
    cache = (Z1, D1, A1, W1, b1, Z2, D2, A2, W2, b2, Z3, A3, W3, b3)
    return A3, cache
  
def backward_propagation_with_dropout(X, Y, cache, keep_prob):
    """
    Implements the backward propagation of our baseline model to which we added dropout.
    
    Returns:
    gradients -- Python dictionary with the gradients with respect to each parameter, activation and pre-activation variables
    """
    
    m = Y.shape[1]
    (Z1, D1, A1, W1, b1, Z2, D2, A2, W2, b2, Z3, A3, W3, b3) = cache
    
    dZ3 = A3 - Y
    # W'shape is (nl, nl-1), Z'shape is (nl, m), A-1'shape is (nl-1, m)
    dW3 = np.dot(dZ3, A2.T) / m
    db3 = np.sum(dZ3, axis=1, keepdims=True) / m
    
    dA2 = np.dot(W3.T, dZ3)
    dA2 = dA2 * D2  # step 1: Apply mask D2 to shut down the same neurons as during the forward propagation
    dA2 = dA2 / keep_prob  # step 2: Scale the value of neurons that haven't been shut down
    dZ2 = np.multiply(dA2, np.int64(A2 > 0))  # np.int64(A2>0) 等价于 np.array(A2>0, dtype=int)
    dW2 = 1. / m * np.dot(dZ2, A1.T)
    db2 = 1. / m * np.sum(dZ2, axis=1, keepdims=True)
    
    dA1 = np.dot(W2.T, dZ2)
    dA1 = dA1 * D1
    dA1 = dA1 / keep_prob
    dZ1 = np.multiply(dA1, np.int64(A1 > 0))
    dW1 = 1. / m * np.dot(dZ1, X.T)
    db1 = 1. / m * np.sum(dZ1, axis=1, keepdims=True)
    
    gradients = {"dZ3": dZ3, "dW3": dW3, "db3": db3,"dA2": dA2,
                 "dZ2": dZ2, "dW2": dW2, "db2": db2, "dA1": dA1, 
                 "dZ1": dZ1, "dW1": dW1, "db1": db1}
    
    return gradients
  
parameters = model(train_X, train_Y, keep_prob=0.86, learning_rate=0.3)

print("On the train set:")
predictions_train = predict(train_X, train_Y, parameters)
print("On the test set:")
predictions_test = predict(test_X, test_Y, parameters)

plt.title("Model with dropout")
axes = plt.gca()
axes.set_xlim([-0.75, 0.40])
axes.set_ylim([-0.75, 0.65])
plot_decision_boundary(lambda x: predict_dec(parameters, x.T), train_X, train_Y)
```

## 作业3: 梯度验证

```python
import numpy as np
from testCases import *
from gc_utils import sigmoid, relu, dictionary_to_vector, vector_to_dictionary, gradients_to_vector

# 1-dimensional gradient checking

# 现在计算的是线性回归：J(theta) = theta * X，就是损失和X是线性关系
"""
J = wx + b - y
"""

def forward_propagation(x, theta):
    """
    Implements the linear forward propagation (compute J) presented J(theta) = theta * x
    
    Arguments:
    x -- a real-valued input
    theta -- our parameter, a real number as well
    
    Returns:
    J -- the value of function J, computed using the formula J(theta) = theta * x
    """
    J = np.dot(theta, x)
    return J
  
def backward_propagation(x, theta):
    dtheta = x
    return dtheta
  
# np.linalg.norm(x, ord=2) 用来计算明可夫斯基距离。ord默认值是2，代表着欧拉距离
def gradient_check(x, theta, epsilon=1e-7):
    """
    Implement the backward propagation presented in Figure 1.
    
    Arguments:
    x -- a real-valued input
    theta -- our parameter, a real number as well
    epsilon -- tiny shift to the input to compute approximated gradient with formula(1)
    
    Returns:
    difference -- difference (2) between the approximated gradient and the backward propagation gradient
    """
    theta_plus = theta + epsilon
    theta_minus = theta - epsilon
    J_plus = forward_propagation(x, theta_plus)
    J_minus = forward_propagation(x, theta_minus)
    grad_approx = (J_plus - J_minus) / (theta_plus - theta_minus)
    
    grad = backward_propagation(x, theta)
    
    numerator = np.linalg.norm(grad - grad_approx)
    denominator = np.linalg.norm(grad) + np.linalg.norm(grad_approx)
    difference = numerator / denominator
    
    if difference < 1e-7:
        print('The gradient is correct!')
    else:
        print('The gradient is wrong!')
    
    return difference
  
x, theta = 2, 4
difference = gradient_check(x, theta)
print("difference = " + str(difference))

# N-dimensional gradient checking

def forward_propagation_n(X, Y, parameters):
    """
    Implements the forward propagation (and computes the cost) presented in Figure 3.
    
    Arguments:
    X -- training set for m examples
    Y -- labels for m examples 
    parameters -- python dictionary containing your parameters "W1", "b1", "W2", "b2", "W3", "b3":
                    W1 -- weight matrix of shape (5, 4)
                    b1 -- bias vector of shape (5, 1)
                    W2 -- weight matrix of shape (3, 5)
                    b2 -- bias vector of shape (3, 1)
                    W3 -- weight matrix of shape (1, 3)
                    b3 -- bias vector of shape (1, 1)
    
    Returns:
    cost -- the cost function (logistic cost for one example)
    """
    
    # retrieve parameters
    m = X.shape[1]
    W1 = parameters["W1"]
    b1 = parameters["b1"]
    W2 = parameters["W2"]
    b2 = parameters["b2"]
    W3 = parameters["W3"]
    b3 = parameters["b3"]

    # LINEAR -> RELU -> LINEAR -> RELU -> LINEAR -> SIGMOID
    Z1 = np.dot(W1, X) + b1
    A1 = relu(Z1)
    Z2 = np.dot(W2, A1) + b2
    A2 = relu(Z2)
    Z3 = np.dot(W3, A2) + b3
    A3 = sigmoid(Z3)

    # Cost
    logprobs = np.multiply(-np.log(A3), Y) + np.multiply(-np.log(1 - A3), 1 - Y)
    cost = 1. / m * np.sum(logprobs)
    
    cache = (Z1, A1, W1, b1, Z2, A2, W2, b2, Z3, A3, W3, b3)
    
    return cost, cache
  
def backward_propagation_n(X, Y, cache):
    """
    Implement the backward propagation presented in figure 2.
    
    Arguments:
    X -- input datapoint, of shape (input size, 1)
    Y -- true "label"
    cache -- cache output from forward_propagation_n()
    
    Returns:
    gradients -- A dictionary with the gradients of the cost with respect to each parameter, activation and pre-activation variables.
    """
    
    m = X.shape[1]
    (Z1, A1, W1, b1, Z2, A2, W2, b2, Z3, A3, W3, b3) = cache
    
    dZ3 = A3 - Y
    dW3 = 1. / m * np.dot(dZ3, A2.T)
    db3 = 1. / m * np.sum(dZ3, axis=1, keepdims=True)
    
    dA2 = np.dot(W3.T, dZ3)
    dZ2 = np.multiply(dA2, np.int64(A2 > 0))
    dW2 = 1. / m * np.dot(dZ2, A1.T) * 2  # Should not multiply by 2
    db2 = 1. / m * np.sum(dZ2, axis=1, keepdims=True)
    
    dA1 = np.dot(W2.T, dZ2)
    dZ1 = np.multiply(dA1, np.int64(A1 > 0))
    dW1 = 1. / m * np.dot(dZ1, X.T)
    db1 = 4. / m * np.sum(dZ1, axis=1, keepdims=True) # Should not multiply by 4
    
    gradients = {"dZ3": dZ3, "dW3": dW3, "db3": db3,
                 "dA2": dA2, "dZ2": dZ2, "dW2": dW2, "db2": db2,
                 "dA1": dA1, "dZ1": dZ1, "dW1": dW1, "db1": db1}
    
    return gradients
  
def gradient_check_n(parameters, gradients, X, Y, epsilon=1e-7):
    """
    Checks if backward_propagation_n computes correctly the gradient of the cost output by forward_propagation_n
    
    Arguments:
    parameters -- python dictionary containing your parameters "W1", "b1", "W2", "b2", "W3", "b3":
    grad -- output of backward_propagation_n, contains gradients of the cost with respect to the parameters. 
    x -- input datapoint, of shape (input size, 1)
    y -- true "label"
    epsilon -- tiny shift to the input to compute approximated gradient with formula(1)
    
    Returns:
    difference -- difference (2) between the approximated gradient and the backward propagation gradient
    """
    
    # Set-up variables
    parameters_values, _ = dictionary_to_vector(parameters)
    grad = gradients_to_vector(gradients)
    num_parameters = parameters_values.shape[0]  # 参数的总数量，就是W1的总数+b1的总数+..
    J_plus = np.zeros((num_parameters, 1))
    J_minus = np.zeros((num_parameters, 1))
    gradapprox = np.zeros((num_parameters, 1))
    
    # Compute gradapprox
    for i in range(num_parameters):
        
        # Compute J_plus[i]. Inputs: "parameters_values, epsilon". Output = "J_plus[i]".
        # "_" is used because the function you have to outputs two parameters but we only care about the first one
        ### START CODE HERE ### (approx. 3 lines)
        thetaplus =  np.copy(parameters_values)                                       # Step 1
        thetaplus[i][0] = thetaplus[i][0] + epsilon                                   # Step 2
        J_plus[i], _ =  forward_propagation_n(X, Y, vector_to_dictionary(thetaplus))  # Step 3
        ### END CODE HERE ###
        
        # Compute J_minus[i]. Inputs: "parameters_values, epsilon". Output = "J_minus[i]".
        ### START CODE HERE ### (approx. 3 lines)
        thetaminus = np.copy(parameters_values)                                       # Step 1
        thetaminus[i][0] = thetaminus[i][0] - epsilon                                 # Step 2        
        J_minus[i], _ = forward_propagation_n(X, Y, vector_to_dictionary(thetaminus)) # Step 3
        ### END CODE HERE ###
        
        # Compute gradapprox[i]
        ### START CODE HERE ### (approx. 1 line)
        gradapprox[i] = (J_plus[i] - J_minus[i]) / (2 * epsilon)
        ### END CODE HERE ###
    
    # Compare gradapprox to backward propagation gradients by computing difference.
    ### START CODE HERE ### (approx. 1 line)
    numerator = np.linalg.norm(grad - gradapprox)                                     # Step 1'
    denominator = np.linalg.norm(grad) + np.linalg.norm(gradapprox)                   # Step 2'
    difference = numerator / denominator                                              # Step 3'
    ### END CODE HERE ###

    if difference > 1e-7:
        print("\033[93m" + "There is a mistake in the backward propagation! difference = " + str(difference) + "\033[0m")
    else:
        print("\033[92m" + "Your backward propagation works perfectly fine! difference = " + str(difference) + "\033[0m")
    
    return difference
  
X, Y, parameters = gradient_check_n_test_case()

cost, cache = forward_propagation_n(X, Y, parameters)
gradients = backward_propagation_n(X, Y, cache)
difference = gradient_check_n(parameters, gradients, X, Y)
```

