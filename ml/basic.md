## KNN

###  KNN - K近邻算法

> K-Nearest Neighbors

在训练数据集时，我们把模型的特征叫做`X_train`，把标签(就是结果)叫做`y_train`。

机器学习的算法会生成模型。训练模型的过程，叫做`fit`拟合。

输出结果时，叫做`predict`预测。 

kNN是一个不需要训练过程的算法。就是说，它没有模型。

为和其它算法统一，对kNN来说，训练集本身就是模型。

#### 欧拉距离

$$
设点a为
\left[
\begin {matrix}
a_1 \\
a_2 \\
...
\end {matrix}
\right]
，点b为
\left[
\begin {matrix}
b_1 \\
b_2 \\
...
\end {matrix}
\right]， \\
则两点距离是
\sqrt{\sum_{i=1}^n{(a_i - b_i)^2}}
$$

求一下：

```python
# 已知raw_data raw_category (都是numpy类型数组) target 
# 求所有点与target的距离

from math import sqrt

# raw_point是每一个点，它是一个数组，每一项是其对应维度的坐标值
distances = [sqrt(np.sum((raw_point - target.x) ** 2)) for raw_point in raw_data]

# 排序：返回的是最近元素的索引
nearest = np.argsort(distances)

# 设置k的值
k = 6

# 求前k个最近点的category
topk_category = [raw_category[i] for i in nearest[:k]]

# 完事，只需统计一下结果
from collections import Counter

# 返回 Counter({0: 1, 1: 3, 2: 5}) 就是几个0 几个1 几个2
votes = Counter(topk_category) 

# 找出票数最多的3个元素
votes.most_common(3)
```

#### 使用scikit-learn中的KNN

```python
import numpy as np
from sklearn.neighbors import KNeighborsClassifier

X_train = np.array(
    [
        [3.39, 2.33],
        [3.11, 1.78],
        [1.34, 3.36],
        [3.58, 4.67],
        [2.28, 2.86],
        [7.42, 4.69],
        [5.74, 3.53],
        [9.17, 2.51],
        [7.79, 3.42],
        [7.93, 0.79]
    ]
)

y_train = np.array(
    [0, 0, 0, 0, 0, 1, 1, 1, 1, 1]
)

X_predict = np.array(
    [
        [8.8, 4.4],
        [1.2, 3.4]
    ]
)

k = 6

knn_classifier = KNeighborsClassifier(n_neighbors=k)
knn_classifier.fit(X_train, y_train)
y_predict = knn_classifier.predict(X_predict)
print(y_predict)  # [1, 0]
```

#### 自己实现

```python
import numpy as np
from math import sqrt
from collections import Counter


class KNNClassifier:
    def __init__(self, k):
        assert k >= 1, 'k must be valid.'
        self._k = k
        self._X_train = None
        self._y_train = None

    def fit(self, X_train, y_train):
        assert X_train.shape[0] == y_train.shape[0], \
            'The size of X_train must be equal to size of y_train.'
        assert self._k <= X_train.shape[0], \
            'The size of X_train must be at least k.'

        self._X_train = X_train
        self._y_train = y_train
        return self

    def predict(self, X_predict):
        """给定待预测数据集X_predict，返回表示X_predict的结果向量"""
        assert self._X_train is not None and self._y_train is not None, \
            'Must fit before predict.'
        assert self._X_train.shape[1] == X_predict.shape[1], \
            'The feature number of X_predict must be equal to X_train.'

        y_predict = [self._predict(x) for x in X_predict]
        return np.array(y_predict)

    def _predict(self, x):
        """给定单个待预测数据x，返回x的预测结果值"""
        distances = [sqrt(np.sum((x - x0) ** 2)) for x0 in self._X_train]
        nearest = np.argsort(distances)

        topK_y = [self._y_train[i] for i in nearest[:self._k]]  # 类似于 [0, 1, 1, 0, 0]
        votes = Counter(topK_y)  # 类似于 (1: 2, 0: 3)
        return votes.most_common(1)[0][0]  # 类似于 3

    def __repr__(self):
        return 'KNN(k=%d)' % self._k


X_train = np.array(
    [
        [3.39, 2.33],
        [3.11, 1.78],
        [1.34, 3.36],
        [3.58, 4.67],
        [2.28, 2.86],
        [7.42, 4.69],
        [5.74, 3.53],
        [9.17, 2.51],
        [7.79, 3.42],
        [7.93, 0.79]
    ]
)

y_train = np.array(
    [0, 0, 0, 0, 0, 1, 1, 1, 1, 1]
)

X_predict = np.array(
    [
        [8.8, 4.4],
        [1.2, 3.4]
    ]
)

k = 6

knn = KNNClassifier(k=6)
knn.fit(X_train, y_train)
y_predict = knn.predict(X_predict)
print(y_predict)  # [1 0]
```

### 训练数据集

为测试算法效果，需要将训练和测试分离(train test split)。

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn import datasets

iris = datasets.load_iris()
X = iris.data
y = iris.target
X.shape  # (150, 4)

# train test split: 把X和y分离为训练数据和测试数据
shuffled_indexes = np.random.permutation(len(X))  # 先乱序：返回一个打乱的索引list
test_ratio = 0.2  # 测试比例
test_size = int(len(X) * test_ratio)

test_indexes = shuffled_indexes[:test_size]
train_indexes = shuffled_indexes[test_size:]

X_train = X[train_indexes]
y_train = y[train_indexes]
X_test = X[test_indexes]
y_test = y[test_indexes]
```

现在封装一下（在model_selection.py中）供使用：

```python
import numpy as np

def train_test_split(X, y, test_ratio=0.2, seed=None):
    """将数据X和y按照test_ratio分割成X_train, y_train, X_test, y_test"""

    assert X.shape[0] == y.shape[0], \
        'The size of X must be equal to size of y.'
    assert 0.0 <= test_ratio <= 1.0, \
        'test_ratio must be valid.'

    if seed:
        np.random.seed(seed)

    shuffled_indexes = np.random.permutation(len(X))

    test_size = int(len(X) * test_ratio)
    test_indexes = shuffled_indexes[:test_size]
    train_indexes = shuffled_indexes[test_size:]

    X_train = X[train_indexes]
    y_train = y[train_indexes]

    X_test = X[test_indexes]
    y_test = y[test_indexes]

    return X_train, y_train, X_test, y_test
```

测试：

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn import datasets
from sklearn.neighbors import KNeighborsClassifier
from KNN.model_selection import train_test_split

iris = datasets.load_iris()
X = iris.data
y = iris.target

X_tr, y_tr, X_te, y_te = train_test_split(X, y)

knn = KNeighborsClassifier(n_neighbors=6)
knn.fit(X_tr, y_tr)

y_predict = knn.predict(X_te)

# 求预测准确率
sum(y_predict == y_te) / len(y_te)  # 0.9
```

**使用sklearn中的train_test_split**：

```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=666)
```

### 分类准确度

自己封装一个确定准确度的方法：

```python
"""metrics.py"""

def accuracy_score(y_true, y_predict):
    assert y_true.shape[0] == y_predict.shape[0], \
        'the size of y_true must be equal to the size of y_predict'

    return sum(y_predict == y_true) / len(y_predict)
```

在KNN中添加score方法：

```python
from .metrics import accuracy_score

# ...

def score(self, X_test, y_test):
    """根据测试数据集 X_test 和 y_test 确定当前模型的准确度"""
    y_predict = self._predict(X_test)
    return accuracy_score(y_test, y_predict)
```

使用`sklearn`中的`accuracy_score`和`score`，来一个KNN示例：

```python
from sklearn import datasets
from sklearn.model_selection import train_test_split
from sklearn.neighbors import KNeighborsClassifier
from sklearn.metrics import accuracy_score

digits = datasets.load_digits()
X = digits.data
y = digits.target

X_train, X_test, y_train, y_test = train_test_split(X, y, train_size=0.2, random_state=666)

neighbors = 3
knn = KNeighborsClassifier(n_neighbors=neighbors)
knn.fit(X_train, y_train)

# 直接使用KNN的score()
score1 = knn.score(X_test, y_test)

# 使用accuracy_score
y_predict = knn.predict(X_test)
score2 = accuracy_score(y_test, y_predict)

print(score1)  # 0.9673157162726008
print(score2)  # 0.9673157162726008
```

### 超参数(hyperparameter)

> 超参数：算法运行前需要决定的参数。kNN的k就是超参数。
>
> 模型参数：算法过程中学习的参数。kNN没有模型参数。

kNN还有一个重要的超参数`距离`。比如，测试数据test最近的3个点为：1个*0类型*和2个*1类型*。

但是这个*0类型*的距离很近，所以其实它的权重更大。

所以，**把距离的倒数变成超参数**。距离越小，倒数越大，权重越大。

> 这里说的距离是欧拉距离。
>
> 曼哈顿距离（点在每个维度上距离差的和）：
> $$
> \sqrt{\sum_{i=1}^n{|a_i - b_i|}}
> $$
> 明可夫斯基(Minkowski)距离：
> $$
> (\sum_{i=1}^n{(a_i - b_i)^p})^{\frac{1}{p}}
> $$
> **这个p又成为了一个超参数**。p为1时是曼哈顿距离，p为2时是欧拉距离。

寻找一下最好的超参数：

```python
from sklearn import datasets
from sklearn.model_selection import train_test_split
from sklearn.neighbors import KNeighborsClassifier

digits = datasets.load_digits()
X = digits.data
y = digits.target

X_train, X_test, y_train, y_test = train_test_split(X, y, train_size=0.2, random_state=666)

best_k = 0
best_score = 0.0
best_p = 0

for k in range(1, 5):
    for p in range(1, 6):
        knn = KNeighborsClassifier(n_neighbors=k, weights='distance', p=p)
        knn.fit(X_train, y_train)

        score = knn.score(X_test, y_test)
        if score > best_score:
            best_k = k
            best_score = score
            best_p = p

print(best_k)  # 4
print(best_p)  # 4
print(best_score)  # 0.9770514603616134
```

### 网格搜索(Grid Search)

使用f的`GridSearchCV`

```python
from sklearn import datasets
from sklearn.model_selection import train_test_split
from sklearn.neighbors import KNeighborsClassifier
from sklearn.model_selection import GridSearchCV

digits = datasets.load_digits()
X = digits.data
y = digits.target

X_train, X_test, y_train, y_test = train_test_split(X, y, train_size=0.2, random_state=666)

param_grid = [
    {
        'weights': ['uniform'],
        'n_neighbors': [k for k in range(1, 6)]
    },
    {
        'weights': ['distance'],
        'n_neighbors': [k for k in range(1, 6)],
        'p': [p for p in range(1, 6)]
    }
]

knn_clf = KNeighborsClassifier()
# n_jobs是传几个核，-1为最大；verbose为输出信息，常用值为2
grid_search = GridSearchCV(knn_clf, param_grid, n_jobs=-1, verbose=2)

grid_search.fit(X, y)

# output: Fitting 5 folds(交叉验证) for each of 30 candidates(组合), totalling 150 fits(5 * 30 拟合)
# ...
print(grid_search.best_estimator_)  # KNeighborsClassifier(n_neighbors=3, p=3, weights='distance')
print(grid_search.best_score_)  # 0.9677313525224388
```

### 更多的距离定义

向量空间余弦相似度(Cosine Similarity)

调整余弦相似度(Ajdusted Cosine Similarity)

皮尔森相关系数(Pearson Correlation Coefficient)

Jaccard相似系数(Jaccard Coefficient)

*KNeighborsClassifier 默认的 metric 参数值是 minkowski，可以修改为别的距离定义。*

### 数据归一化(Feature Scaling)

> 通过一个Scaler(换算器)对数据统一进行归一化。

比如，

| -     | 肿瘤大小(cm) | 发现时间(天) |
| ----- | ------------ | ------------ |
| 样本1 | 1            | 200          |
| 样本2 | 5            | 100          |

可以看到，**样本间的距离被发现时间所主导**。

解决方案有

#### 最值归一化(normalization)

> 适用于分布有明显边界的情况，比如考试分数。
>
> 不适用于有离群数据(outlier)，比如大家挣80，你挣20000。
>
> 注意：y是标签，不用归一

把所有数据映射到0-1之间。
$$
x_{scale} = \frac{x-x_{min}}{x_{max}-x_{min}}
$$
代码：

```python
import numpy as np

# 对向量
x = np.random.randint(0, 100, size=50)

normalized_x = (x - np.min(x)) / (np.max(x) - np.min(x))
print(normalized_x[:10])
print('-------')

# 对矩阵
X = np.random.randint(0, 100, (50, 2))
X = np.array(X, dtype=float)

for i in range(X.shape[1]):
    X[:, i] = (X[:, i] - np.min(X[:, i])) / (np.max(X[:, i]) - np.min(X[:, i]))

print(X[:10])
```

#### 均值方差归一化(standardization)

> 适用于分布没有明显边界，就是可能存在极端数据值。

把所有数据映射为：均值为0，方差为1这样的范围。
$$
x_{scale} = \frac{x-x_{mean}}{S}
$$
代码：

```python
import numpy as np
from matplotlib import pyplot as plt

# 对向量
x = np.random.randint(0, 100, size=50)

normalized_x = (x - np.mean(x)) / np.std(x)
print(normalized_x[:10])
print('-------')

# 对矩阵
X = np.random.randint(0, 100, (50, 2))
X = np.array(X, dtype=float)

for i in range(X.shape[1]):
    X[:, i] = (X[:, i] - np.mean(X[:, i])) / np.std(X[:, i])

print(X[:10])
plt.scatter(X[:, 0], X[:, 1])
```

### 对测试数据集如何归一化

> 比如使用均值方差归一化，可以使用训练数据的mean_train和std_train。
>
> 对数据的归一化也是算法的一部分。

使用sklearn的数据归一化方法：

```python
from sklearn import datasets
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.neighbors import KNeighborsClassifier

digits = datasets.load_digits()

X = digits.data
y = digits.target

X_train, X_test, y_train, y_test = train_test_split(X, y, train_size=0.2, random_state=666)

# 数据归一化：它会保存mean和std
ss = StandardScaler()
ss.fit(X_train)

X_train = ss.transform(X_train)
X_test = ss.transform(X_test)

knc = KNeighborsClassifier(n_neighbors=4, weights='distance', p=4)
knc.fit(X_train, y_train)
score = knc.score(X_test, y_test)

# 分数很低，说明这个数据集不适合用均值方差归一化
print(score)  # 0.9193324061196105
```

自己实现简单的数据归一化方法：

```python
class StandardScaler:
    def __init__(self):
        self.mean_ = None;
        self.scale_ = None;

    def fit(self, X):
        """根据训练数据集X获得数据的均值和方差"""
        assert X.ndim == 2, \
            'The dimension of X must be 2'

        self.mean_ = np.array([np.mean(X[:, i]) for i in range(X.shape[1])])
        self.scale_ = np.array([np.std(X[:, i]) for i in range(X.shape[1])])
        return self

    def transform(self, X):
        """将X这个StandardScaler进行均值方差归一化处理"""
        assert self.mean_ is not None and self.scale_ is not None, \
            'must fit before transform'

        resX = np.empty(shape=X.shape, dtype=float)
        for col in range(X.shape[1]):
            resX[:, col] = (X[:, col] - self.mean_[col]) / self.scale_[col]
        return resX
```

### 总结

KNN用来解决分类问题，同时可以解决多分类问题。

KNN也可以解决回归问题，在`sklearn`中封装了`KNeightorsRegressor`。

**缺点1：效率低下**。如果训练集有m个样本、n个特征，则预测1个新的数据，需要O(m*n)。

可以使用数结构优化如KD-Tree、Ball-Tree。

**缺点2：高度数据相关**。比如k为3时，如果有两个数据是错的，那么就错了。

**缺点3：预测结果不具有可解释性**。

**缺点4：维数灾难**。随着维数的增加，看似相近的两个点之间的距离越来越大。比如 (0, 0, .., 0) 和 (1, 1, ..., 1) 的距离(10000维)是100。

## 线性回归法(Linear Regression)

### 简单线性回归

假设最佳拟合的直线方程为：$y=ax+b$，则对于每个样本点$x_i$，其预测值为$\hat{y}_i = ax_i + b$。

我们希望所有的$\hat{y}_i$和真值$y_i$差距尽可能小。

所以有：
$$
\sum_{i=1}^{m}(y_i - \hat{y}_i)^2
尽可能小 \\
即
\sum_{i=1}^{m}(y_i - ax_i - b)^2 \\
x_i, y_i已知，所以要找到a和b \\
a = \frac
{\sum_{i=1}^{m}(x_i - \bar{x})(y_i - \bar{y})}
{\sum_{i=1}^{m}(x_i - \bar{x})^2} 
,\space
b=\bar{y} - a\bar{x}
$$

> 说明：
>
> 这种测量损失程度的函数，称为损失函数(loss function)。
>
> 对应相反的函数称为效用函数(utility function)。
>
> 通过最优化损失函数或者效用函数，获得机器学习的模型。近乎所有参数学习算法都是这样的套路。
>
> 这个学科叫*最优化原理*。

### 求解a, b

$$
求\sum_{i=1}^{m}(y_i - ax_i - b)^2的极小值 \\
所以要分别对a, b求导。\\
$$

先对a求导：
$$
\sum_{i=1}^{m}2(y_i - ax_i - b)(-1) = 0 \\
\sum_{i=1}^{m}y_i - a\sum_{i=1}^{m}x_i - \sum_{i=1}^{m}b = 0 \\
mb = \sum_{i=1}^{m}y_i - a\sum_{i=1}^{m}x_i \\
两边同时除以m：\\
b = \bar{y} - a\bar{x} \space\space (式1)
$$
再对b求导：
$$
\sum_{i=1}^{m}2(y_i - ax_i - b)(-x_i) = 0 \\
\sum_{i=1}^{m}(y_i - ax_i - b)x_i = 0 \\
把式1代进去：\\
\sum_{i=1}^{m}(y_i - ax_i - \bar{y} + a\bar{x})x_i = 0 \\
\sum_{i=1}^{m}(y_ix_i-\bar{y}x_i) = \sum_{i=1}^{m}(ax_i^2 - a\bar{x}x_i) \\
求得：\\
a = \frac
{\sum_{i=1}^{m}(y_ix_i-\bar{y}x_i)}
{\sum_{i=1}^{m}(x_i^2 -  \bar{x}x_i)} \space\space (式2)
$$
因为$\sum_{i=1}^{m}x_i\bar{y}=\sum_{i=1}^{m}\bar{x}y_i=\sum_{i=1}^{m}\bar{x}\bar{y}$，

式2可以变换为：
$$
a=
\frac
{\sum_{i=1}^{m}(x_iy_i - x_i\bar{y} - \bar{x}y_i + \bar{x}\bar{y})}
{\sum_{i=1}^{m}(x_i^2 - \bar{x}x_i - \bar{x}x_i + \bar{x}^2)}
\\
=
\frac
{\sum_{i=1}^{m}(x_i - \bar{x})(y_i - \bar{y})}
{\sum_{i=1}^{m}(x_i - \bar{x})^2}
$$

### 实现简单线性回归

在Jupyter Notebook试一下：

```python
import numpy as np
import matplotlib.pyplot as plt

x = np.array([1.0, 2, 3, 4, 5])
y = np.array([1.0, 3, 2, 3, 5])

x_mean = np.mean(x)
y_mean = np.mean(y)

deno = 0.0 # 分母
nume = 0.0 # 分子

# 计算a和b
for x_i, y_i in zip(x, y):
    nume += (x_i - x_mean) * (y_i - y_mean)
    deno += (x_i - x_mean) ** 2
    
a = nume / deno
b = y_mean - a * x_mean

# 拟合方程
y_hat = a * x + b

# 画图
plt.scatter(x, y)
plt.plot(x, y_hat)
```

自己封装一下：

```python
import numpy as np

class SimpleLinearRegression1:
    # y_hat = a * x + b
    def __init__(self):
        self.a_ = None
        self.b_ = None

    def fit(self, x_train, y_train):
        assert x_train.ndim == 1, \
            'simple linear regression can only solve simple feature training data'
        assert x_train.shape[0] == y_train.shape[0], \
            'the size of x_train must be equal to the size of y_train'

        x_mean = np.mean(x_train)
        y_mean = np.mean(y_train)

        nume = 0.0  # 分子
        deno = 0.0  # 分母

        for x_i, y_i in zip(x_train, y_train):
            nume += (x_i - x_mean) * (y_i - y_mean)
            deno += (x_i - x_mean) ** 2

        self.a_ = nume / deno
        self.b_ = y_mean - self.a_ * x_mean

        return self

    def predict(self, x_predict):
        """给定待处理数据集x_predict，返回表示x_predict的预测向量"""
        assert self.a_ is not None and self.b_ is not None, \
            'must fit before predict'
        assert x_predict.ndim == 1, \
            'simple linear regression can only solve simple feature training data'

        return np.array([self._predict(x) for x in x_predict])

    def _predict(self, x):
        return self.a_ * x + self.b_

    # 打印类实例时显示的信息
    def __repr__(self):
        return 'single linear regression'
```

### 对上面的方法优化

核心是，将for循环中的数值运算替代为向量化运算。
$$
a=
\frac
{\sum_{i=1}^{m}(x_i - \bar{x})(y_i - \bar{y})}
{\sum_{i=1}^{m}(x_i - \bar{x})^2} \\
设
\vec{w} = \sum_{i=1}^{m}(x_i - \bar{x}),
\vec{v} = \sum_{i=1}^{m}(y_i - \bar{y}),\\
则a=\frac
{\vec{w} \dot{} \vec{v}}
{\vec{w} \dot{} \vec{w}}
$$

代码：

```python
class SimpleLinearRegression2:
    # y_hat = a * x + b
    def __init__(self):
        self.a_ = None
        self.b_ = None

    def fit(self, x_train, y_train):
        assert x_train.ndim == 1, \
            'simple linear regression can only solve simple feature training data'
        assert x_train.shape[0] == y_train.shape[0], \
            'the size of x_train must be equal to the size of y_train'

        x_mean = np.mean(x_train)
        y_mean = np.mean(y_train)

        nume = (x_train - x_mean).dot(y_train - y_mean)  # 分子
        deno = (x_train - x_mean).dot(x_train - x_mean)  # 分母

        self.a_ = nume / deno
        self.b_ = y_mean - self.a_ * x_mean

        return self

    def predict(self, x_predict):
        """给定待处理数据集x_predict，返回表示x_predict的预测向量"""
        assert self.a_ is not None and self.b_ is not None, \
            'must fit before predict'
        assert x_predict.ndim == 1, \
            'simple linear regression can only solve simple feature training data'

        return np.array([self._predict(x) for x in x_predict])

    def _predict(self, x):
        return self.a_ * x + self.b_

    # 打印类实例时显示的信息
    def __repr__(self):
        return 'single linear regression'
```

测试：

```python
import numpy as np
from SLR import SimpleLinearRegression1, SimpleLinearRegression2
m = 1000000
x = np.random.random(size=m)
y = x * 2.0 + 3 + np.random.normal(size=m)

s1 = SimpleLinearRegression1()
s2 = SimpleLinearRegression2()

%timeit s1.fit(x, y)  # 956 ms ± 71.5 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
%timeit s2.fit(x, y)  # 7.49 ms ± 760 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)
```

### 评估精确度

方法1：`均方根误差(Root Mean Squared Error)`。它的值可以认为是预测的平均误差，但是略微放大了。

> $\sqrt{\frac{1}{m}\sum_{i=1}^{m}(y_i-\hat{y_i})^2}$，其中$y_i$是测试真值，$\hat{y_i}$是预测值。

方法2：`平均绝对误差(Mean Absolute Error)`。它的值可以认为是预测结果和真实结果的平均误差。

> $\frac{1}{m}\sum_{i=1}^{m}(y_i-\hat{y_i})$，其中$y_i$是测试真值，$\hat{y_i}$是预测值。

代码：

```python
def mean_squared_error(y_true, y_predict):
  	assert y_true.shape[0] == y_predict.shape[0], \
  			'the size of y_true must be equal to the size of y_predict'
    return np.sum((y_true - y_predict) ** 2) / len(y_true) 

# 略
```

#### scikit-learn中的MSE和MAE

> sklearn中没有RMSE，可以手动开方一下。

```python
from sklearn.metrics import mean_squared_error, mean_absolute_error

mean_squared_error(y_true, y_predict)
mean_absolute_error(y_true, y_predict)
```

### 更好的评估方法R Squared

$$
R^2 = 1 - \frac
{\sum_i(\hat{y}_i-y_i)^2}
{\sum_i(\bar{y}_i-y_i)^2}
\\
= 1 - \frac
{\sum_i(\hat{y}_i-y_i)^2 / m}
{\sum_i(\bar{y}_i-y_i)^2 / m}
= 1 - \frac{MSE}{Variance}
$$



性质：

1. R^2 <= 1
2. R\^2越大越好。如果预测模型不犯任何错误，R\^2得到最大值1
3. 如果R^2 <= 0，说明学习的模型还不如基准模型。此时，很可能数据不存在任何线性关系

#### 在sklearn中的使用

```python
from sklearn.metrics import r2_score
r2_score(y_test, y_predict)
```

> `sklearn`封装的`LinearRegression`跟`KNN`一样，也有`score`方法。其`score`就是用的`r2`。

### 多元线性回归

最佳拟合的线性方程为：$y = \theta_0 + \theta_1x_1 + \theta_2x_2 + .. + \theta_nx_n$。对于每个样本点$X^{(i)}$，其预测值为$\hat{y^{(i)}} = \theta_0 + \theta_1X^{(i)}_1 + \theta_2X^{(i)}_2 + .. + \theta_nX^{(i)}_n$。

捋一下：
$$
\vec{\theta} = 
\left[
\begin{matrix}
\theta_0, \theta_1, \theta_2, ...
\end{matrix}
\right] \\
\vec{X} = 
\left[
\begin{matrix} 

\left[
\begin{matrix}
X^{(1)}_1, X^{(1)}_2, X^{(1)}_3, ..
\end{matrix}
\right]
\\
\left[
\begin{matrix}
X^{(2)}_1, X^{(2)}_2, X^{(2)}_3, ..
\end{matrix}
\right]
\\
\left[
\begin{matrix}
X^{(3)}_1, X^{(3)}_2, X^{(3)}_3, ..
\end{matrix}
\right]
\\ ...

\end{matrix}
\right]
$$


计算一下：
$$
给\theta_0也添加一个X：\\
\hat{y^{(i)}} = \theta_0X^{(i)}_0 + \theta_1X^{(i)}_1 + \theta_2X^{(i)}_2 + .. + \theta_nX^{(i)}_n \\
= \vec{\theta} \dot{} \vec{X^{(i)}} \\
其中，每个变量都是一个数值，X^{(i)}_0是1
$$
所以有：
$$
\sum_{i=1}^m(y^{(i)}-\hat{y}^{(i)})^2 尽可能小，\\
即
\sum_{i=1}^m(y^{(i)}-\vec{\theta} \dot{} \vec{X^{(i)}})^2 \\
$$
迷糊了..

多元线性回归的正规方程解(Normal Equation):
$$
能算得：
\theta = (X^T_bX_b)^{-1}X^T_b{y} \\
X_b是X^{(0)}和\vec{X}的结合，
X^T是矩阵转置
$$
缺点：时间复杂度是O(n^3)

优点：不需要对数据做归一化处理

### 实现多元线性回归

代码：

```python
class LinearRegression:
    def __init__(self):
        self.intercept_ = None;  # 截距 theta 0
        self.coefficient_ = None;  # 系数 theta 1-n
        self._theta = 0;

    # 使用正规方程解(Normal Equation)进行拟合
    def fit(self, X_train, y_train):
        assert X_train.shape[0] == y_train.shape[0], \
            'the size of X_train must be equal to the size of y_train'

        # hstack: y方向上进行结合
        X_b = np.hstack((np.ones((len(X_train), 1)), X_train))
        self._theta = np.linalg.inv(X_b.T.dot(X_b)).dot(X_b.T).dot(y_train)
        self.intercept_ = self._theta[0]
        self.coefficient_ = self._theta[1:]
        return self

    def predict(self, X_predict):
        assert self.intercept_ is not None and self.coefficient_ is not None, \
            'must fit before predict'
        assert len(self.coefficient_) == X_predict.shape[1], \
            'the feature of X_predict must be equal to X_train'

        X_b = np.hstack((np.ones((len(X_predict), 1)), X_predict))
        return X_b.dot(self._theta)
```

测试：

```python
import numpy as np
from sklearn import datasets
from matplotlib import pyplot as plt
from KNN.model_selection import LinearRegression
from sklearn.model_selection import  train_test_split

boston = datasets.load_boston()
X = boston.data
y = boston.target

X = X[y<50]
y = y[y<50]

X_train, X_test, y_train, y_test = train_test_split(X, y)
lr = LinearRegression()
lr.fit(X_train, y_train)

y_predict = lr.predict(X_test)
```

### sklearn中的回归

```python
from sklearn import datasets
from sklearn.linear_model import LinearRegression
from sklearn.model_selection import train_test_split

boston = datasets.load_boston()
X = boston.data
y = boston.target
X = X[y < 50]
y = y[y < 50]
X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=666)

lr = LinearRegression()
lr.fit(X_train, y_train)

s = lr.score(X_test, y_test)
print(s)  # 0.8009390227581041
```

#### kNN Regressor

> k近邻来求解线性回归问题。

使用`sklearn`中的`KNeighborsRegressor`，同时使用网格搜索最好的超参数：

```python
from sklearn import datasets
from sklearn.model_selection import train_test_split, GridSearchCV
from sklearn.neighbors import KNeighborsRegressor

boston = datasets.load_boston()
X = boston.data
y = boston.target
X = X[y < 50]
y = y[y < 50]
X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=666)

param_grid = [
    {
        'weights': ['uniform'],
        'n_neighbors': [i for i in range(1, 11)]
    },
    {
        'weights': ['distance'],
        'n_neighbors': [i for i in range(1, 11)],
        'p': [i for i in range(1, 6)]
    }
]

knr = KNeighborsRegressor(n_neighbors=3, weights='distance', p=4)
g_search = GridSearchCV(knr, param_grid, n_jobs=-1, verbose=2)
g_search.fit(X_train, y_train)

r = g_search.best_params_
s = g_search.best_score_  # 这个值使用了交叉计算
real_s = g_search.best_estimator_.score(X_test, y_test)  # 真实分数
print(r)  # {'n_neighbors': 6, 'p': 1, 'weights': 'distance'}
print(s)  # 0.6243135119018297
print(real_s)  # 0.7353138117643773
```

### 线性回归的可解释性

> 即便线性回归算法不能得到很好的预测结果，通过观察参数列表推出特征的相关性，它也可能有很高的实用意义。
>
> 优点：对数据具有强解释性。

波士顿房价各个特征的相关性：

```python
from sklearn import datasets
from sklearn.linear_model import LinearRegression
import numpy as np

boston = datasets.load_boston()
X = boston.data
y = boston.target

lr = LinearRegression()
lr.fit(X, y)

# 参数的意义：负数为负相关，正数为正相关；绝对值越大，相关性越强
args = np.argsort(lr.coef_)  # 参数进行索引排序

# from min to max
print(np.sort(lr.coef_))
"""
[-1.77666112e+01 -1.47556685e+00 -9.52747232e-01 -5.24758378e-01
 -1.08011358e-01 -1.23345939e-02  6.92224640e-04  9.31168327e-03
  2.05586264e-02  4.64204584e-02  3.06049479e-01  2.68673382e+00
  3.80986521e+00]
"""
sorted_features = boston.feature_names[args]  # ['NOX' 'DIS' 'PTRATIO' 'LSTAT' 'CRIM' 'TAX' 'AGE' 'B' 'INDUS' 'ZN' 'RAD' 'CHAS' 'RM']
```

## 梯度下降法(Gradient Descent)

### 简介

它不是机器学习算法，是一种基于搜索的最优化方法。

> 梯度下降法：作用是最小化一个损失函数(Loss Function)
>
> 梯度上升法：最大化一个效用函数(Utility Function)

**步长$\eta$是一个超参数**。如果太小，学习速度很慢；如果太大，损失函数J不收敛。

#### 注意

有时候，一个函数有多个极小值，即多个导数为0的情况，所以求得的极值点可能是局部极值。

解决方案：多次运行，随机化初始点。

所以**初始点的位置也是一个超参数**。

### 模拟一个梯度函数

```python
import numpy as np
import matplotlib.pyplot as plt

# [-1, 6]共141个均匀的点
x = np.linspace(-1, 6, 141)
y = (x - 2.5) ** 2 - 1

plt.plot(x, y)


# 损失函数
def J(theta):
    try:
        return (theta - 2.5) ** 2 - 1
    except:
        return float('inf')


# 导数derivative
def deriv_J(theta):
    return 2 * (theta - 2.5)


theta = 0  # 起始theta
eta = 0.1  # 步长
epsilon = 1e-8  # 最小值范围
theta_history = []

n_iters = 1e4  # 最大循环次数
i = 0
while i < n_iters:
    i += 1
    gradient = deriv_J(theta)
    last_theta = theta
    theta -= eta * gradient
    theta_history.append(theta)
    # 判断是否达到了最小值0
    if abs(J(last_theta) - J(theta)) <= epsilon:
        print(theta)
        print(J(theta))
        break

# 绘制梯度图
# 找到最小点，即损失函数最小化
plt.plot(x, y)  # 2.499891109642585 -0.99999998814289
plt.plot(np.array(theta_history), J(np.array(theta_history)), color='r', marker='*')
```

### 实现线性回归中的梯度下降法

$$
目标仍是使损失函数：\\
J(\theta) = \frac{1}{m}\sum_{i=1}^m(y^{(i)} - \theta_0 - \theta_1X^{(i)}_1 - \theta_2X^{(i)}_2 - ... - \theta_nX^{(i)}_n)^2 \\
= \frac{1}{m}\sum_{i=1}^m(y^{(i)} - X^{(i)}_b\theta)^2 \\
尽可能小。其中X_b是 \space hstack(X_0,X) \\
损失函数的梯度（梯度就是多个一元导数的向量）：\\
\nabla{J}(\theta) = 
\left[
\begin{matrix}
\frac{\partial{J}}{\partial{\theta_0}} \\
\frac{\partial{J}}{\partial{\theta_1}} \\
\frac{\partial{J}}{\partial{\theta_2}} \\
.. \\
\frac{\partial{J}}{\partial{\theta_n}}
\end{matrix}
\right]
=
\frac{2}{m}
\left[
\begin{matrix}
\sum_{i=1}^m(X^{(i)}_b\theta - y^{(i)}) \\
\sum_{i=1}^m(X^{(i)}_b\theta - y^{(i)}) \cdot X^{(i)}_1 \\
\sum_{i=1}^m(X^{(i)}_b\theta - y^{(i)}) \cdot X^{(i)}_2 \\
.. \\
\sum_{i=1}^m(X^{(i)}_b\theta - y^{(i)}) \cdot X^{(i)}_n
\end{matrix}
\right]
$$

> J是损失函数，梯度下降是对J这个损失函数进行梯度下降；
>
> 导数可以代表方向，负值说明在减小，正值说明在增加；梯度是多个维度的导数；
>
> theta是损失函数的参数；
>
> 我们的目的是：找到损失函数值最低的那个点，然后求得对应theta。

代码：

```python
import numpy as np
import matplotlib.pyplot as plt


# 计算损失函数的值： (y_hat - y_real) ** 2 / m
def J(theta, X_b, y):
    try:
        return np.sum(y - X_b.dot(theta)) ** 2 / len(X_b)
    except:
        return float('int')


# 计算当前梯度： 基于损失函数，对每个维度进行求导的导数值，合并称为的向量
def deriv_J(theta, X_b, y):
    ret = np.empty(len(theta))
    ret[0] = np.sum(X_b.dot(theta) - y)
    for j in range(1, len(theta)):
        ret[j] = (X_b.dot(theta) - y).dot(X_b[:, j])
    return ret * 2 / len(X_b)


# 梯度下降法： 求出导数为0时的theta
# eta步长
def gradient_descent(X_b, y, initial_theta, n_iters=1e4, epsilon=1e-8, eta=0.01):
    theta = initial_theta  #
    i_iter = 0  # 循环次数
    while i_iter < n_iters:
        gradient = deriv_J(theta, X_b, y)
        last_theta = theta
        theta = theta - eta * gradient
        i_iter += 1

        if abs(J(theta, X_b, y) - J(last_theta, X_b, y)) < epsilon:
            break
    return theta


np.random.seed(666)
x = 2 * np.random.random(size=100)
y = x * 3. + 4 + np.random.normal(size=100)
X_b = np.hstack((np.ones((len(x), 1)), x.reshape(-1, 1)))
init_theta = np.zeros(X_b.shape[1])

theta = gradient_descent(X_b, y, initial_theta=init_theta)
print(theta)  # [4.02298606 3.00577383]
```

### 梯度下降法优化：向量计算代替数值计算

把矩阵写一下：
$$
X_b = 
\left[
\begin{matrix}
X^{(1)}_0 & X^{(1)}_1 & .. & X^{(1)}_n \\
X^{(2)}_0 & X^{(2)}_1 & .. & X^{(2)}_n \\
.. \\
X^{(m)}_0 & X^{(m)}_1 & .. & X^{(m)}_n
\end{matrix}
\right] \\

\theta = 
\left[
\begin{matrix}
\theta_0 \\
\theta_1 \\
.. \\
\theta_n
\end{matrix}
\right] \\

y = 
\left[
\begin{matrix}
y_1 \\
y_2 \\
.. \\
y_m
\end{matrix}
\right]
$$
损失函数的梯度是：
$$
\nabla{J}(\theta) = 
\frac{2}{m}
\left[
\begin{matrix}
\sum_{i=1}^m(X^{(i)}_b\theta - y^{(i)}) \cdot X^{(i)}_0 \\
\sum_{i=1}^m(X^{(i)}_b\theta - y^{(i)}) \cdot X^{(i)}_1 \\
\sum_{i=1}^m(X^{(i)}_b\theta - y^{(i)}) \cdot X^{(i)}_2 \\
.. \\
\sum_{i=1}^m(X^{(i)}_b\theta - y^{(i)}) \cdot X^{(i)}_n
\end{matrix}
\right] \\
=
\frac{2}{m} \cdot (X_b\theta - y)^T \cdot X_b 再转置 \\
= \frac{2}{m} \cdot X^T_b \cdot (X_b\theta - y)
$$

#### 数据归一化

因为有`eta`步长这个属性应用到多个特征上，所以需要对数据进行归一化。

### 随机梯度下降法(Stochastic Gradient Descent)

前面的梯度下降法中，每次循环都要对所有的数据进行计算；

随机梯度下降法中，每次循环只对随机的一组数据进行计算。

随机梯度下降法循环的结束条件，只有`n_iters`，因为`epsilon`不具有确定准确性。

**`n_iters`越大，值越准确。**

> 对于损失函数梯度的值：$\frac{2}{m} \cdot X^{(i)}_b \cdot (X^{(i)}_b\theta - y^{(i)})$
>
> 对于eta的值：`eta = t0 / (count + t1)`，其中t0, t1是超参数，比如5, 50

代码：

```python
def dJ_sgd(theta, X_b_i, y_i):
    return X_b_i * (X_b_i.dot(theta) - y_i) * 2.0


# 随机梯度下降法
# n_iters指的是几圈
def sgd(X_b, y, initial_theta, n_iters=5, t0=5, t1=50):
    theta = initial_theta
    len_X = len(X_b)
    for i_iter in range(n_iters):
        indexes = np.random.permutation(len_X)  # 获取随机打乱的索引数组
        X_b_shuffled = X_b[indexes]
        y_shuffled = y[indexes]
        for t in range(len_X):
            eta = t0 / (t1 + i_iter * len_X + t)
            gradient = dJ_sgd(theta, X_b_shuffled[t], y_shuffled[t])
            theta = theta - eta * gradient
    return theta
```

### sklearn中的随机梯度下降法

```python
from sklearn import datasets
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import SGDRegressor

boston = datasets.load_boston()
X = boston.data
y = boston.target
X = X[y < 50]
y = y[y < 50]
ss = StandardScaler()
ss.fit(X)
X = ss.transform(X)
X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=666)


sgd = SGDRegressor(n_iter_no_change=500)
sgd.fit(X_train, y_train)

sc = sgd.score(X_test, y_test)
print(sc)  # 0.8008291106557693
```

### 梯度下降法的debug

> 它本质上就是标准求导的方法，只不过速度比较慢。所以可以用它来求出测试数据的标准答案，来验证自己写的求梯度的方法，是否是正确的。


在`结果点`($\theta, J$)的上面取一个点，下面取一个点，它们与`结果点`的x轴距离都是$\epsilon$。

如果`结果点`是最小值，则有
$$
\frac{dJ}{d\theta} = 
\frac{J(\theta + \epsilon) - J(\theta - \epsilon)} {2\epsilon}
$$
如果是高维，即$\theta$是一个向量：
$$
\theta = (\theta_0, \theta_1, .., \theta_n) \\

\theta_+ = 
(
\theta_0^+, \theta_1^+, .., \theta_n^+
)
= \\
[
(\theta_0 + \epsilon, \theta_1, .., \theta_n) \\
(\theta_0, \theta_1 + \epsilon, .., \theta_n) \\
.. \\
(\theta_0, \theta_1, .., \theta_n + \epsilon)
] \\

\frac{\partial{J}}{\partial{\theta}} = 
(
\frac{\partial{J}}{\partial{\theta_0}},
\frac{\partial{J}}{\partial{\theta_1}}, 
.., 
\frac{\partial{J}}{\partial{\theta_n}}
) \\
=
(
\frac{J(\theta_0^+) - J(\theta_0^-)}{2\epsilon},
\frac{J(\theta_1^+) - J(\theta_1^-)}{2\epsilon},
..,
\frac{J(\theta_n^+) - J(\theta_n^-)}{2\epsilon}
)
$$
代码：

```python
def dJ_debug(theta, X_b, y, epsilon=0.01):
  res = np.empty(len(theta))
  for i in range(len(theta)):
    theta_1 = theta.copy()
    theta_1[i] += epsilon
    theta_2 = theta.copy()
    theta_2[i] -= epsilon
    res[i] = J(theta_1, X_b, y) - J(theta_2, X_b, y) / (2 * epsilon)
  return res
```



### 小批量梯度下降法(Mini-Batch Gradient Descent)

> 每次看k个样本。

## PCA和梯度上升法(Gradient Ascent)

### 主成分分析(Principal Component Analysis)

> 非监督机器学习算法。主要用于数据降维。

#### 数据降维

找到一个轴，使得样本空间的所有点映射到这个轴后，使得方差最大。

方差最大就是：普遍的点与点之间的距离最大，这样降维的效果是最好的。

1. demean：将样本的均值归为0；均值是每一项的和的平均值，所以demean的对象是axis=0的数据
2. 求一个轴的方向w供样本映射；二维时是$(w_1, w_2)$
3. 使 $var(X_{project}) = \frac{1}{m}\sum_{i=1}^m(X^{(i)}_{project} - \bar{X}_{project})^2$ 最大

捋一下：
$$
\vec{w}是方向向量，长度为1 \\
X^{(i)}是第i个点 \\
X^{(i)}_{project}是投影到映射轴上的第i个点 \\
X^{(i)} \cdot \vec{w} 表示X^{(i)}在\vec{w}反向上的投影 \\
X^{(i)} \cdot \vec{w} = |X^{(i)}| \cdot |\vec{w}| \cdot cos\theta =  |X^{(i)}| \cdot cos\theta = |X^{(i)}_{project}|
$$
所以，我们要求的最大值等于：
$$
所以，我们求Var(X_{project}) = \frac{1}{m}\sum_{i=1}^{m}(X^{(i)}_{project} - \bar{X}_{project})^2 的最大值， \\
因为均值被做了demean处理，所以， \\
即 Var = \frac{1}{m}\sum_{i=1}^{m}(X^{(i)}_{project})^2 \\
= \frac{1}{m}\sum_{i=1}^{m}(X^{(i)} \cdot \vec{w})^2（对点乘进行展开） \\
= \frac{1}{m}\sum_{i=1}^{m}(X^{(i)}_1w_1 + X^{(i)}_2w_2 + .. + X^{(i)}_nw_n)^2
$$

### 使用梯度上升法求最大值

$$
求w，使得 f(X) = \frac{1}{m}\sum_{i=1}^{m}(\vec{X^{(i)}} \cdot \vec{w})^2 \\
= \frac{1}{m}\sum_{i=1}^{m}(X^{(i)}_1w_1 + X^{(i)}_2w_2 + .. + X^{(i)}_nw_n)^2 最大 \\
\nabla{f} = 
\left[
\begin{matrix}
\frac{\partial{f}}{\partial{w_1}} \\
\frac{\partial{f}}{\partial{w_2}} \\
... \\
\frac{\partial{f}}{\partial{w_n}} \\
\end{matrix}
\right] \\
=
\frac2m
\left[
\begin{matrix}
\sum^m_{i=1}(X^{(i)}_1w_1 + X^{(i)}_2w_2 + .. + X^{(i)}_nw_n)X^{(i)}_1 \\
\sum^m_{i=1}(X^{(i)}_1w_1 + X^{(i)}_2w_2 + .. + X^{(i)}_nw_n)X^{(i)}_2 \\
... \\
\sum^m_{i=1}(X^{(i)}_1w_1 + X^{(i)}_2w_2 + .. + X^{(i)}_nw_n)X^{(i)}_n \\
\end{matrix}
\right]
\\
=
\frac2m
\left[
\begin{matrix}
\sum^m_{i=1}(\vec{X^{(i)}}\vec{w})X^{(i)}_1 \\
\sum^m_{i=1}(\vec{X^{(i)}}\vec{w})X^{(i)}_2 \\
... \\
\sum^m_{i=1}(\vec{X^{(i)}}\vec{w})X^{(i)}_n \\
\end{matrix}
\right] \\
=
\frac2m
\cdot
(
\left[
\begin{matrix}
\vec{X^{(1)}}\vec{w} & \vec{X^{(2)}}\vec{w} & .. & \vec{X^{(m)}}\vec{w}
\end{matrix}
\right] 
\cdot
\left[
\begin{matrix}
X^{(1)}_1 & X^{(1)}_2 & .. & X^{(1)}_n \\
X^{(2)}_1 & X^{(2)}_2 & .. & X^{(2)}_n \\
.. \\
X^{(m)}_1 & X^{(m)}_2 & .. & X^{(m)}_n \\
\end{matrix}
\right]
)^ T

\\
= \frac2m \cdot 
( \space
(\vec{X}\vec{w})^T \cdot \vec{X}
\space
)^T \\
= \frac2m \cdot (\vec{X})^T(\vec{X}\vec{w})
$$

反推有点难，但还是推完啦。

代码：

```c++
import numpy as np
from matplotlib import pyplot as plt

# 生成数据集
X = np.empty((100, 2))  # 生成 100行2列 的矩阵，每项数据为0
X[:, 0] = np.random.uniform(0.0, 100.0, size=100)  # 给X的第一列随机赋值 0-100
X[:, 1] = 0.75 * X[:, 0] + 3 + np.random.normal(0, 10.0, size=100)  # 给X的第二列按照第一列进行规律赋值

plt.scatter(X[:, 0], X[:, 1])


def demean(X):
    """
    对矩阵进行demean

    理解一下这个函数
    >> X = np.array([[1, 2, 3, 4], [1, 2, 3, 5]])
    >> print(np.mean(X, axis=0))
    # outputs: [1.  2.  3.  4.5]
    # 所以 axis=0 表示：在列上求均值

    >> print(X - np.mean(X, axis=0))
    # [[ 0.   0.   0.  -0.5], [ 0.   0.   0.   0.5]]
    >> X - np.mean(X, axis=0)) 表示：对每一项进行列方向的demean
    """
    return X - np.mean(X, axis=0)


# 每个维度的均值都为0
X_demean = demean(X)

"""梯度上升法"""


def f(w, X):
    """求y值"""
    return np.sum(X.dot(w) ** 2) / X.shape[0]


def df_math(w, X):
    """求y的梯度，即对y的求导"""
    return X.T.dot(X.dot(w)) * 2 / X.shape[0]


def df_debug(w, X, epsilon=0.00001):
    """对求梯度的debug"""
    len = w.shape[0]
    res = np.empty(len)
    for i in range(len):
        w_1 = w.copy()
        w_1[i] += epsilon
        w_2 = w.copy()
        w_2[i] -= epsilon
        res[i] = (f(w_1, X) - f(w_2, X)) / (2 * epsilon)
    return res


def direction(w):
    """
    求向量的方向向量：vec(w) / |w|
    :param w: 向量
    :return: 方向向量
    """
    return w / np.linalg.norm(w)


def gradient_ascent(df, X, initial_w, eta, n_iters=1e4, epsilon=1e-8):
    """
    梯度上升法
    :param df: 求导函数
    :param X: 数据集
    :param initial_w: 起始的进行测试的w。w是要找的参数变量
    :param eta: 步长
    :param n_iters: 最大循环次数
    :param epsilon: 误差最小值
    :return: 参数变量w
    """
    cur_iter = 0
    w = direction(initial_w)
    while cur_iter < n_iters:
        gradient = df(w, X)  # 当前梯度
        last_w = w  # 记录上一次w
        w = direction(w + eta * gradient)  # 计算本次w的值
        if abs(f(w, X) - f(last_w, X)) < epsilon:  # 如果两次的y值差趋近于0
            break
        cur_iter += 1
    return w


initial_w = np.random.random(X.shape[1])  # 随机起始参数
eta = 0.001
w = gradient_ascent(df_debug, X_demean, initial_w, eta)

print(gradient_ascent(df_debug, X_demean, initial_w, eta))  # [0.77739904 0.62900773]
print(gradient_ascent(df_math, X_demean, initial_w, eta))  # [0.77739904 0.62900773]

# 绘制图形
plt.scatter(X_demean[:, 0], X_demean[:, 1])
plt.plot([0, w[0] * 30], [0, w[1] * 30], color='r')
```

> 需要注意的是：
>
> 1. 在对参数变量 w 进行计算时，要把它变为方向向量
> 2. 起始点不能是0向量，因为0向量对应的y值是0，它是一个最小值，所以导数也是0
> 3. 数据归一化时，不能使用 StandardScaler (均值方差归一化，使均值为0，方差为1) 标准化数据，因为我们的目的就是求方差最大

### 求数据的前n个主成分

在上一个主成分数据的前提下，剔除掉数据在上一个主成分上的分量。

比如：
$$
已求得 \space \vec{w} \space 为样本空间的第一主成分 \\
那么，X^{(i)} - X^{(i)}_{project} 就是，去掉数据在该主成分上的分量 \\
其中，\\
X^{(i)} \cdot w = ||X^{(i)}_{pr}||, \\
X^{(i)}_{pr} = ||X^{(i)}_{pr}|| \cdot w, \\
X'^{(i)} = X^{(i)} - X^{(i)}_{pr} = X^{(i)} \cdot w * w， \\
X'^{(i)} 就可以用来求第二主成分
$$
代码：

```python
# 将数据去掉第一主成分
# 这里X2和X3的结果一样，只不过X3比较难懂，但X2比较慢
X2 = np.empty(X.shape)
for i in range(len(X)):
    X2[i] = X[i] - X[i].dot(w) * w
    
X3 = X - X.dot(w).reshape(-1, 1) * w
```

封装一下：

```python
def first_n_components(n, X, eta=0.01, n_iters=1e4, epsilon=1e-6):
    """
    求前n个主成分
    :param n: 要求主成分的数量
    :param X: 数据集
    :param eta: 步长
    :param n_iters: 最大循环次数
    :param epsilon: 误差最小值，就是小于epsilon时就认为等于0
    :return: 一个numpy数组，每一项都是一个主成分w
    """
    X_pca = demean(X.copy())
    ret = []
    for i in range(n):
        initial_w = np.random.random(X.shape[1])  # 随机起始参数
        w = gradient_ascent(df_math, X_pca, initial_w, eta)
        ret.append(w)
        X_pca = X_pca - X_pca.dot(w).reshape(-1, 1) * w
    return ret
```

对于二维数据来说，去掉第一主成分后，就只剩第二主成分了。两个主成分是垂直的。

### 高维数据向低维数据映射

现在我们有了前k个主成分。
$$
X = 
\left[
\begin{matrix}
X^{(1)}_1 & X^{(1)}_2 & .. & X^{(1)}_n \\
X^{(2)}_1 & X^{(2)}_2 & .. & X^{(2)}_n \\
.. \\
X^{(m)}_1 & X^{(m)}_2 & .. & X^{(m)}_n
\end{matrix}
\right] \\

W_k = 
\left[
\begin{matrix}
W^{(1)}_1 & W^{(1)}_2 & .. & W^{(1)}_n \\
W^{(2)}_1 & W^{(2)}_2 & .. & W^{(2)}_n \\
.. \\
W^{(k)}_1 & W^{(k)}_2 & .. & W^{(k)}_n 
\end{matrix}
\right] \\
$$
我们让$X \cdot W_k$，就可以得到X数据集在每个主成分上的映射：
$$
X_k = X \cdot W_k^T \\
$$
此时$X_k$变成一个m * k的矩阵，它就是低维数据。

如果要反映到原来的体系中，可以再reverse一下：
$$
X_k(reverse) = X \cdot X_k
$$
代码：

```python
import numpy as np


class PCA:
    """
    n_components: n个主成分
    components: w(k)，n个主成分构成的矩阵
    """

    def __init__(self, n_components):
        """初始化"""
        assert n_components >= 1, 'n_components must be valid'
        self.n_components = n_components
        self.components_ = None

    def fit(self, X, eta=0.01, n_iters=1e4):
        """获得数据集前n个主成分"""
        assert self.n_components <= X.shape[1], \
            'n_components must not be greater than the feature number of X'

        def demean(X):
            return X - np.mean(X, axis=0)

        def f(w, X):
            return np.sum((X.dot(w) ** 2)) / X.shape[0]

        def df(w, X):
            return X.T.dot(X.dot(w)) * 2.0 / X.shape[0]

        def direction(w):
            return w / np.linalg.norm(w)

        def first_component(X, initial_w, eta=0.01, n_iters=1e4, epsilon=1e-8):
            w = direction(initial_w)
            cur_iter = 0
            while cur_iter < n_iters:
                gradient = df(w, X)  # 当前梯度
                last_w = w  # 记录上一次w
                w = direction(w + eta * gradient)  # 计算本次w的值
                if abs(f(w, X) - f(last_w, X)) < epsilon:  # 如果两次的y值差趋近于0
                    break
                cur_iter += 1
            return w

        X_pca = demean(X)
        self.components_ = np.empty(shape=(self.n_components, X.shape[1]))
        for i in range(self.n_components):
            initial_w = np.random.random(X.shape[1])  # 随机起始参数
            w = first_component(X_pca, initial_w, eta)
            self.components_[i, :] = w
            X_pca = X_pca - X_pca.dot(w).reshape(-1, 1) * w

        return self

    def transform(self, X):
        """将给定的X，映射到各个主成分分量中"""
        assert X.shape[1] == self.components_.shape[1]
        return X.dot(self.components_.T)

    def inverse_transform(self, X):
        """将给定的X，反向映射回原来的特征空间"""
        assert X.shape[1] == self.components_.shape[0]
        return X.dot(self.components_)

    def __repr__(self):
        return 'PCA(n_components = %d)' % self.n_components
```

测试：

```python
from KNN.KNN import PCA

import numpy as np
import matplotlib.pyplot as plt

X = np.empty((100, 2))
X[:, 0] = np.random.uniform(0.0, 100.0, size=100)
X[:, 1] = 0.75 * X[:, 0] + 3.0 + np.random.normal(0.0, 10.0, size=100)

pca = PCA(n_components=1)
pca.fit(X)

x_reduction = pca.transform(X)
x_restore = pca.inverse_transform(x_reduction)

plt.scatter(x_restore[:, 0], x_restore[:, 1])
plt.scatter(X[:, 0], X[:, 1], color='black')
```

### sklearn中的PCA

```python
from sklearn import datasets
from sklearn.model_selection import train_test_split
from sklearn.neighbors import KNeighborsClassifier
from sklearn.decomposition import PCA

digits = datasets.load_digits()
X = digits.data
y = digits.target
X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=666)

# 对原始数据进行操作
knn = KNeighborsClassifier()
knn.fit(X_train, y_train)
print(knn.score(X_test, y_test))  # 0.9866666666666667

# 对数据降维
pca = PCA(n_components=60)
pca.fit(X_train, y_train)
X_train_reduction = pca.transform(X_train)
X_test_reduction = pca.transform(X_test)

knn.fit(X_train_reduction, y_train)
print(knn.score(X_test_reduction, y_test))  # 0.6066666666666667
```

可以看到`n_components=2`时，分数太低。

`PCA`有一个`explained_variance_ratio_`字段，意思是可解释的方差比例，它用来查看每个维度所占分量的重要性。

```python
pca = PCA(n_components=X.shape[1])
pca.fit(X_train, y_train)
print(pca.explained_variance_ratio_)  # [1.45668166e-01 1.37354688e-01 1.17777287e-01 8.49968861e-02 ..]
```

可以通过这个字段看到所有维度的重要程度。

`PCA`可以直接设置想要保留的精度的百分比，来自动完成`n_components`字段的填写：

```python
from sklearn import datasets
from sklearn.model_selection import train_test_split
from sklearn.neighbors import KNeighborsClassifier
from sklearn.decomposition import PCA

digits = datasets.load_digits()
X = digits.data
y = digits.target
X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=666)

# 对原始数据进行操作
knn = KNeighborsClassifier()
knn.fit(X_train, y_train)
print(knn.score(X_test, y_test))  # 0.9866666666666667

# 对数据降维
pca = PCA(0.99)
pca.fit(X_train, y_train)
X_train_reduction = pca.transform(X_train)
X_test_reduction = pca.transform(X_test)

knn.fit(X_train_reduction, y_train)
print(pca.n_components_)  # 41。一共是64个维度，PCA自动保留了41个成分
print(knn.score(X_test_reduction, y_test))  # 0.9822222222222222
```

可以用time来查看用时。所以用很小的精度误差可以换取大量的时间消耗。

### MNIST数据集

使用`MNIST`手写数据集来测试一下`PCA`

> 第一次加载数据会比较耗时

```python
from sklearn.datasets import fetch_openml
from sklearn.neighbors import KNeighborsClassifier
from sklearn.decomposition import PCA

mnist = fetch_openml("mnist_784")

X, y = mnist['data'], mnist['target']
# X.shape  # (70000, 784)

# 数据已完成train_test_split，前60k个是训练数据，后10k个是测试数据
X_train, X_test = X[:60000], X[60000:]
y_train, y_test = y[:60000], y[60000:]
# 这个样本不需要归一化

# 使用kNN进行数据分类
knn = KNeighborsClassifier()
# %time knn.fit(X_train, y_train)  # CPU times: user 14.1 s, sys: 197 ms, total: 14.3 s
knn.fit(X_train, y_train)
# %time knn.score(X_test, y_test)  # user 10min 51s, sys: 3.52 s, total: 10min 55s
print(knn.score(X_test, y_test))  # 0.9688

# 对数据进行降维
pca = PCA(0.90)
pca.fit(X_train)
X_train_reduction = pca.transform(X_train)
X_test_reduction = pca.transform(X_test)

# %time knn.fit(X_train_reduction, y_train) # CPU times: user 1.34 s, sys: 23.5 ms, total: 1.36 s
knn.fit(X_train_reduction, y_train)
# %time knn.score(X_test_reduction, y_test)  # CPU times: user 1min, sys: 399 ms, total: 1min
print(knn.score(X_test_reduction, y_test))  # 0.9728
```

可以看到，虽然只选择了90%的精度，但是结果分数更高一些，因为`PCA`不仅有降维的功能，还有降噪的效果。

> 噪音：数据集本身可能是不准确的，如测量仪器有误差、测量手段有问题等，导致数据产生了噪音。

### 特征脸

> 第一次加载数据会比较耗时

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.datasets import fetch_lfw_people

faces = fetch_lfw_people()

print(faces.data.shape)  # (13233, 2914)
print(faces.image.shape)  # (13233, 62, 47)

random_indexes = np.random.permutation(len(faces.data))  # 生成随机的索引数组
X = faces.data[random_indexes]

# 拿出36个作为示例
example_faces = X[:36, :]
print(example_faces.shape)  # (36, 2914)


# 绘制函数
def plot_faces(faces):
    fig, axes = plt.subplots(
        6,
        6,
        figsize=(10, 10),
        subplot_kw={'xticks': [], 'yticks': []},
        gridspec_kw=dict(hspace=0.1, wspace=0.1)
    )
    for i, ax in enumerate(axes.flat):
        ax.imshow(faces[i].reshape(62, 47), cmap='bone')
    plt.show()


plot_faces(example_faces)

# 后面特征脸就不研究了
```

## 多项式回归与模型泛化

### 多项式回归

很多时候，特征和标记并不是线性关系。

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.linear_model import LinearRegression

# 模拟一个 y = ax^2 + bx +c 这样的曲线
x = np.random.uniform(-3, 3, size=100)  # 生成值分布在-3 ~ 3之间的一维数组
X = x.reshape(-1, 1)  # (100, 1)
y = 0.5 * x ** 2 + x + 2 + np.random.normal(0.0, 1.0, size=100)

plt.scatter(X, y)  # 散点

lin = LinearRegression()
lin.fit(X, y)
y_predict = lin.predict(X)
# 这里传的是小x，不是大X；大X是二维的
plt.plot(x, y_predict, color='black')  # 拟合出一条 直线

# 在第二维度，添加特征
X2 = np.hstack([X ** 2, X])
print(X2.shape)  # (100, 2)

lin2 = LinearRegression()
lin2.fit(X2, y)
y2_predict = lin2.predict(X2)
plt.plot(np.sort(x), y2_predict[np.argsort(x)], color='#f11')  # 拟合出一条 弧线
```

多项式回归是**升维**操作，就是升高了维度。

### scikit-learn中的多项式回归

使用`PolynomialFeatures`

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.preprocessing import PolynomialFeatures
from sklearn.linear_model import LinearRegression

# 模拟一个 y = ax^2 + bx +c 这样的曲线
x = np.random.uniform(-3, 3, size=100)  # 生成值分布在-3 ~ 3之间的一维数组
X = x.reshape(-1, 1)  # (100, 1)
y = 0.5 * x ** 2 + x + 2 + np.random.normal(0.0, 1.0, size=100)

plt.scatter(X, y)  # 散点

poly = PolynomialFeatures(degree=2)  # 为数据集添加几次幂，这里是2次
poly.fit(X)
X2 = poly.transform(X)

print(poly.get_feature_names())  # ['1', 'x0', 'x0^2']
print(X2.shape)  # (100, 3)

lin = LinearRegression()
lin.fit(X2, y)
y_predict = lin.predict(X2)

plt.plot(np.sort(x), y_predict[np.argsort(x)], color='#f22')  # 一条弧线
```

`PolynomialFeatures`会生成完备的多项式。

比如原来有2个特征x1、x2，`degree`为3，最终会生成
$$
1, x_1, x_2, \\
x_1^2, x_2^2, x_1x_2, \\
x_1^3, x_2^3, x_1^2x_2, x_1x_2^2
$$


同时，因为有幂的计算导致数值差别极大，所以还需要做`数据归一化`。

#### Pipeline

> scikit-learn中提供的方法，可以完成一系列的操作。

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import PolynomialFeatures
from sklearn.linear_model import LinearRegression
from sklearn.preprocessing import StandardScaler

# 模拟一个 y = ax^2 + bx +c 这样的曲线
x = np.random.uniform(-3, 3, size=100)  # 生成值分布在-3 ~ 3之间的一维数组
X = x.reshape(-1, 1)  # (100, 1)
y = 0.5 * x ** 2 + x + 2 + np.random.normal(0.0, 1.0, size=100)

plt.scatter(X, y)  # 散点

poly_reg_pipe = Pipeline([
    ('poly', PolynomialFeatures(degree=2)),
    ('std_scaler', StandardScaler()),
    ('lin-reg', LinearRegression())
])
poly_reg_pipe.fit(X, y)
y_predict = poly_reg_pipe.predict(X)

plt.plot(np.sort(x), y_predict[np.argsort(x)], color='#f11')
```

### 过拟合(overfitting)和欠拟合(underfitting)

> 一般情况下，我们认为MSE(均方根误差)越小越好，但实际上可能不是。
>
> > $\sqrt{\frac{1}{m}\sum_{i=1}^{m}(y_i-\hat{y_i})^2}$，其中$y_i$是测试真值，$\hat{y_i}$是预测值。
>
> 过拟合的原因：算法所训练的模型过多地表达了数据间的噪音关系。
>
> **举个例子：**
>
> **猫和狗都认为是狗，就是欠拟合，提取的特征过少；**
>
> **除了眼睛鼻子等等，颜色必须是黄色才是狗，就是过拟合，提取的特征过多实则提取的是噪音。**

如下面代码，degree为100时是过拟合，为1时是欠拟合：

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import PolynomialFeatures
from sklearn.linear_model import LinearRegression
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import mean_squared_error

np.random.seed(666)

# 模拟一个 y = ax^2 + bx +c 这样的曲线
x = np.random.uniform(-3, 3, size=100)  # 生成值分布在-3 ~ 3之间的一维数组
X = x.reshape(-1, 1)  # (100, 1)
y = 0.5 * x ** 2 + x + 2 + np.random.normal(0.0, 1.0, size=100)

plt.scatter(X, y)  # 散点

# degree为2
pipe = Pipeline([
    ('poly', PolynomialFeatures(degree=2)),
    ('std_scaler', StandardScaler()),
    ('lin_reg', LinearRegression())
])

pipe.fit(X, y)
y_predict = pipe.predict(X)
score = mean_squared_error(y, y_predict)
print(score)  # 1.0987392142417856
plt.plot(np.sort(x), y_predict[np.argsort(x)], color='#000')

# degree为100
pipe = Pipeline([
    ('poly', PolynomialFeatures(degree=100)),
    ('std_scaler', StandardScaler()),
    ('lin_reg', LinearRegression())
])

pipe.fit(X, y)
y_predict = pipe.predict(X)
score = mean_squared_error(y, y_predict)
print(score)  # 0.687293250556113
plt.plot(np.sort(x), y_predict[np.argsort(x)], color='#f11')

# degree为1
pipe = Pipeline([
    ('poly', PolynomialFeatures(degree=1)),
    ('std_scaler', StandardScaler()),
    ('lin_reg', LinearRegression())
])

pipe.fit(X, y)
y_predict = pipe.predict(X)
score = mean_squared_error(y, y_predict)
print(score)  # 3.0750025765636577
plt.plot(np.sort(x), y_predict[np.argsort(x)], color='#1f1')
```

### 测试数据集的意义

> 模型的泛化能力：由此及彼的能力。就是学习到的模型对未知数据的预测能力。

上述的情况可以通过train test split来解决。测试数据可以保证是否是欠拟合或过拟合。

如下改进的代码：

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import PolynomialFeatures
from sklearn.linear_model import LinearRegression
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import mean_squared_error
from sklearn.model_selection import train_test_split

np.random.seed(666)

# 模拟一个 y = ax^2 + bx +c 这样的曲线
x = np.random.uniform(-3, 3, size=100)  # 生成值分布在-3 ~ 3之间的一维数组
X = x.reshape(-1, 1)  # (100, 1)
y = 0.5 * x ** 2 + x + 2 + np.random.normal(0.0, 1.0, size=100)
X_train, X_test, y_train, y_test = train_test_split(X, y)

plt.scatter(X, y)  # 散点

# degree为2
pipe = Pipeline([
    ('poly', PolynomialFeatures(degree=2)),
    ('std_scaler', StandardScaler()),
    ('lin_reg', LinearRegression())
])

pipe.fit(X_train, y_train)
y_predict = pipe.predict(X_test)
score = mean_squared_error(y_test, y_predict)
print(score)  # 1.1193064268260262
plt.plot(np.sort(X_test[:, 0]), y_predict[np.argsort(X_test[:, 0])], color='#000')

# degree为100
pipe = Pipeline([
    ('poly', PolynomialFeatures(degree=100)),
    ('std_scaler', StandardScaler()),
    ('lin_reg', LinearRegression())
])

pipe.fit(X_train, y_train)
y_predict = pipe.predict(X_test)
score = mean_squared_error(y_test, y_predict)
print(score)  # 962232878919712.5
plt.plot(np.sort(X_test[:, 0]), y_predict[np.argsort(X_test[:, 0])], color='#f11')

# degree为1
pipe = Pipeline([
    ('poly', PolynomialFeatures(degree=1)),
    ('std_scaler', StandardScaler()),
    ('lin_reg', LinearRegression())
])

pipe.fit(X_train, y_train)
y_predict = pipe.predict(X_test)
score = mean_squared_error(y_test, y_predict)
print(score)  # 2.901241018175763
plt.plot(np.sort(X_test[:, 0]), y_predict[np.argsort(X_test[:, 0])], color='#1f1')
```

### 学习曲线

> 学习曲线是：随着训练样本的逐渐增多，算法训练出的模型的表现能力。

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import PolynomialFeatures
from sklearn.linear_model import LinearRegression
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import mean_squared_error
from sklearn.model_selection import train_test_split

np.random.seed(666)

# 模拟一个 y = ax^2 + bx +c 这样的曲线
x = np.random.uniform(-3, 3, size=100)  # 生成值分布在-3 ~ 3之间的一维数组
X = x.reshape(-1, 1)  # (100, 1)
y = 0.5 * x ** 2 + x + 2 + np.random.normal(0.0, 1.0, size=100)
X_train, X_test, y_train, y_test = train_test_split(X, y)


def plot_learning_curve(algo, X_train, X_test, y_train, y_test):
    """
    绘制学习曲线
    :param algo: 使用的算法实例，比如 lin = LinearRegression
    :param X_train: 训练数据集X
    :param X_test: 测试数据集X
    :param y_train: 训练数据集y
    :param y_test: 测试数据集y
    :return: void
    """
    # 训练数据和测试数据分数的统计
    train_score = []
    test_score = []

    for i in range(1, len(X_train) + 1):
        algo.fit(X_train[:i], y_train[:i])

        y_train_predict = algo.predict(X_train[:i])
        train_score.append(mean_squared_error(y_train[:i], y_train_predict))
        y_test_predict = algo.predict(X_test)
        test_score.append(mean_squared_error(y_test, y_test_predict))

    # 绘制学习曲线
    plt.plot([i for i in range(1, len(X_train) + 1)], np.sqrt(train_score), label='train')
    plt.plot([i for i in range(1, len(X_train) + 1)], np.sqrt(test_score), label='test')
    plt.legend()


# 线性回归学习曲线
plot_learning_curve(LinearRegression(), X_train, X_test, y_train, y_test)

# 多项式回归学习曲线
# 可以看出训练数据分数和测试数据分数的曲线趋近的MSE更小
pipe = Pipeline([
    ('poly', PolynomialFeatures(degree=2)),
    ('std_scaler', StandardScaler()),
    ('lin_reg', LinearRegression())
])
plot_learning_curve(pipe, X_train, X_test, y_train, y_test)
```

### 交叉验证(Cross Validation)

> 测试数据集可以用来测试模型是否是过拟合。
>
> 但是如果如果只是一组固定不变的测试数据，很可能模型是过拟合的，但是它通过了该测试数据集。

我们把数据集分为3部分：**训练数据(train)、验证数据(validation)和测试数据(test)**。

训练数据和验证数据用于模型的生成，其中验证数据是调整超参数使用的数据集。

测试数据集不参与模型的构建，它只作为衡量最终模型性能的数据集。

**交叉验证：将训练数据分为k份，其中1份作为验证数据，k-1份作为训练数据。共执行k次。**（k-folds）

*sklearn中的网格搜索就是使用了交叉验证。*

```python
import numpy as np
from sklearn import datasets
from sklearn.model_selection import train_test_split
from sklearn.neighbors import KNeighborsClassifier
# 交叉验证
from sklearn.model_selection import cross_val_score
from sklearn.model_selection import GridSearchCV

digits = datasets.load_digits()
X = digits.data
y = digits.target
X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=666)

"""不使用交叉验证"""

best_k, best_p, best_score = 0, 0, 0.0

for k in range(1, 11):  # kNN中的k
    for p in range(1, 5):  # kNN中的p
        knn = KNeighborsClassifier(weights='distance', n_neighbors=k, p=p)
        knn.fit(X_train, y_train)
        score = knn.score(X_test, y_test)
        if score > best_score:
            best_k, best_p, best_score = k, p, score

print(best_k)  # 5
print(best_p)  # 2
print(best_score)  # 0.9866666666666667

"""使用交叉验证"""
for k in range(1, 11):  # kNN中的k
    for p in range(1, 5):  # kNN中的p
        knn = KNeighborsClassifier(weights='distance', n_neighbors=k, p=p)
        # 交叉验证返回的分数列表；cv参数是分成几份，默认值就是3
        scores = cross_val_score(knn, X_train, y_train, cv=3)
        score = np.mean(scores)
        if score > best_score:
            best_k, best_p, best_score = k, p, score

print(best_k)  # 9
print(best_p)  # 3
# 这个不是真实的R Squared分数
print(best_score)  # 0.9873661021616412

"""通过best_k和best_p拿到best_score"""
best_knn = KNeighborsClassifier(weights='distance', n_neighbors=9, p=3)
best_knn.fit(X_train, y_train)
print(best_knn.score(X_test, y_test))  # 0.9844444444444445

# 拿网格搜索验证一下
params = [
    {
        'weights': ['distance'],
        'n_neighbors': [i for i in range(2, 11)],
        'p': [i for i in range(6)]
    }
]
grid = GridSearchCV(knn, params, verbose=1)
grid.fit(X_train, y_train)
print(grid.best_score_)  # 0.9873661021616412（不是真实的R Squared分数）
grid.best_params_  # {'n_neighbors': 9, 'p': 3, 'weights': 'distance'}
best_knn_by_grid = grid.best_estimator_
print(best_knn_by_grid.score(X_test, y_test))  # 0.9844444444444445
```

#### 留一法(Leave-One-Out Cross Validation)

训练数据有多少个，就分成多少份。这样的分法使得交叉验证完全不受随机的影响，即最接近模型真正的性能指标。

计算量巨大。

### 偏差方差权衡(Bias Variance Trade off)

模型误差 = 偏差 + 方差 + 不可避免的误差

> 不可避免的误差如：数据集本身是有误差的

偏差如：欠拟合；非线性数据使用线性回归（如特征是姓名，标签是考试分数）

方差如：模型太复杂，如高阶多项式回归，过拟合

高方差算法：如kNN；非参数学习通常都是高方差算法

高偏差算法：如线性回归；参数学习通常都是高偏差算法

> 比如kNN算法：
>
> 1. k为1时，方差最大，偏差最小；此时跟距离的关系性最强；
> 2. k为最大值、即数据集的数量，偏差最大，方差最小；此时kNN跟距离没有关系了，只跟数据集标签数量有关。
>
> 比如多项式回归：
>
> 1. degree为1时，模型最简单，偏差最大，方差最小；
> 2. degree越大，模型越复杂，方差越大。

通常，降低方差会提高偏差，降低偏差会提高方差。

**在算法层面上，机器学习的主要挑战，来自于方差。**（不考虑数据层面）

#### 解决高方差的通常手段

降低模型复杂度；减少数据维度，降噪；增加样本规模；使用验证集；模型正则化等。

### 模型正则化(Regularization)

过拟合有一个明显的特点是：参数值$\theta$会变得很大。

为了防止模型过拟合，我们加入模型正则化。
$$
目标：使
J(\theta) = \sum_{i=1}^m(y^{(i)} - \vec{\theta} \cdot \vec{X^{(i)}})^2
尽可能小， \\
加入模型正则化：\\
J(\theta) = \sum_{i=1}^m(y^{(i)} - \vec{\theta} \cdot \vec{X^{(i)}})^2
+ \alpha \frac12 \sum^n_{i=1}\theta_i^2
\\
这里的\frac12没有实质含义，只是为了方便求导，因为有\alpha这个变量
$$
这种模型正则化的方式称为**岭回归(Ridge Regression)**。

#### sklearn中的Ridge

```python
import numpy as np
from sklearn import datasets
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_squared_error
from sklearn.linear_model import Ridge
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import PolynomialFeatures
from sklearn.preprocessing import StandardScaler

digits = datasets.load_digits()
x = np.random.uniform(-3.0, 3.0, size=100)
X = x.reshape(-1, 1)
y = 0.5 * x + 3 + np.random.normal(0, 1, size=100)
X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=666)


def RidgeRegression(degree, alpha):
    return Pipeline([
        ('poly', PolynomialFeatures(degree=degree)),
        ('std_scaler', StandardScaler()),
        ('ridge', Ridge(alpha=alpha))
    ])


ridge_reg = RidgeRegression(20, 1)
ridge_reg.fit(X_train, y_train)
y_predict = ridge_reg.predict(X_test)
mse = mean_squared_error(y_test, y_predict)
print(mse)  # 0.8095187328236685
```

### LASSO Regularization

> Least Absolute Shrinkage and Selection Operator Regression
>
> [关于lasso的文章](https://zhuanlan.zhihu.com/p/46999826)

$$
目标：使J(\theta) = MSE(y, \hat{y}, \theta) + \alpha \sum^n_{i=1}|\theta_i| \\
和Ridge的区别就是后面那部分
$$



```python
import numpy as np
from sklearn import datasets
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_squared_error
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import PolynomialFeatures
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import Lasso

digits = datasets.load_digits()
np.random.seed(666)
x = np.random.uniform(-3.0, 3.0, size=100)
X = x.reshape(-1, 1)
y = 0.5 * x + 3 + np.random.normal(0, 1, size=100)
X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=666)


def LassoRegression(degree, alpha):
    return Pipeline([
        ('poly', PolynomialFeatures(degree=degree)),
        ('std_scaler', StandardScaler()),
        ('lasso', Lasso(alpha=alpha))
    ])


lasso_reg = LassoRegression(20, 0.015)
lasso_reg.fit(X_train, y_train)
y_predict = lasso_reg.predict(X_test)
mse = mean_squared_error(y_test, y_predict)
print(mse)  # 0.8527492333939035
```

### 范数

$$
形如：\\
||x||_p = (\sum_{i=1}^n |x_i|^p)^{\frac1p} \\
这样的形式被称为Lp范数 \\
$$

p为1时称为L1范数，它表示该点到原点的曼哈顿距离；p为2时是L2范数，它表示该点到原点的欧拉距离。

`LASSO`被称为L1正则项，`Ridge`是L2正则项（没有开方）

`L0正则项`：使要求的参数$\vec{\theta}$的个数尽可能的少。

#### 弹性网(Elastic Net)

$$
J(\theta) = MSE(y, \hat{y}; \theta) + Ridge \cdot r + LASSO \cdot (1-r)
$$

结合两者的优点.. 目前有点听不懂

## 逻辑回归(Logistic Regression)

### 介绍

通常作为分类算法用，只可以解决二分类问题。

在线性回归的基础上，添加上[Sigmoid函数](https://baike.baidu.com/item/Sigmoid%E5%87%BD%E6%95%B0/7981407)。
$$
线性回归：y(\theta) = \theta^T \cdot x_b \\
Sigmoid函数：\sigma(t) = \frac1{1+e^{-t}} \\
套上Sigmoid：p(\theta) = \sigma = \frac1{1+e^{-\theta^T \cdot x_b}}
$$
说明：Sigmoid函数值的范围是(0, 1)，所以，逻辑回归只能做二分类工作，它可以把值反映到0 - 1的范围上。

### 逻辑回归的损失函数(cost function)

p是逻辑回归函数，就是求`y_predict`，范围在(0, 1)之间。

损失函数就是$J(\theta)$。

如果y=1、即真值为1，求得的p越小时，cost越大；

如果y=0、即真值为0，求得的p越大时，cost越大。

所以我们按下面的式子来设定损失函数：
$$
cost = -log(\hat{p}), if \space\space y=1 \\
cost = -log(1-\hat{p}), if \space\space y=0 \\
可以看图片
$$
如果y=1，p趋近于0，cost就趋近于无穷；p趋近于1，cost趋近于0。

合并上面的式子：
$$
cost = -ylog(\hat{p}) - (1-y)log(1-\hat{p})
$$
所以逻辑回归的损失函数是：
$$
J(\theta) = -\frac1m \sum_{i=1}^m ( y^{(i)}log(\hat{p}^{(i)}) + (1-y^{(i)})log(1-\hat{p}^{(i)}) ) \\
其中 \space p = \frac1{1+e^{-\theta^T \cdot x_b}}
$$
只能用梯度下降法求解。

### 梯度下降

> 对$\theta$各个分量进行求导，求损失函数的梯度

线性回归：
$$
\hat{y}(\theta) = \theta^T \cdot X_b \\
\frac {dy} {d\theta_j} = X_j \\
这里的y在下面写作p
$$


Sigmoid函数求导：
$$
Sigmoid函数：\sigma(t) = \frac1{1+e^{-t}} = (1+e^{-t})^{-1} \\
\sigma(t)' = -(1+e^{-t})^{-2} \cdot e^{-t} \cdot (-1) = (1+e^{-t})^{-2} e^{-t}
$$


所以 $logp$求导：
$$
log\sigma(t)' = \frac{1}{\sigma(t)} \cdot \sigma(t)' （这两个值上面都有）\\
= \frac{1}{(1+e^{-t})^{-1}} \cdot (1+e^{-t})^{-2} e^{-t} \\
= \frac{e^{-t}}{1+e^{-t}} \\
= 1 - \frac{1}{1 + e^{-t}} \\
= 1 - \sigma(t) \\
所以： \\
logp(\theta)'= \frac{d(logp)}{d\theta_j}
= (1 - \sigma(t)) \cdot \frac{dt}{d\theta_j} = (1 - \sigma(t)) \cdot X_j
$$
$log(1-\hat{p})$求导：
$$
log(1-\sigma(t))' = \frac{1}{1-\sigma(t)} \cdot (-1) \cdot \sigma(t)' \\
= \frac{1}{1 - (1+e^{-t})^{-1}} \cdot (-1) \cdot (1+e^{-t})^{-2} e^{-t} \\
= -\sigma(t) 没错是这个值 \\
所以： \\
log(1-p(\theta))' = \frac{d(log(1-p))}{d\theta_j} \\
=-\sigma(t) \cdot \frac{dt}{d\theta_j} = -\sigma(t) \cdot X_j
$$


所以 $J(\theta)$：
$$
J(\theta) = -ylog(\hat{p}) - (1-y)log(1-\hat{p}) \\
= -\frac1m \sum_{i=1}^m 
( y^{(i)}log(\hat{p}^{(i)}) + (1-y^{(i)})log(1-\hat{p}^{(i)}) ) \\

所以，\space把上面结果代入：\\
\frac{dJ}{d\theta_j} = -\frac1m \sum_{i=1}^m 
(y^{(i)}(1-\sigma(t)) \cdot X^{(i)}_j + 
(1 - y^{(i)}) \cdot -\sigma(t) \cdot X^{(i)}_j)  \\
= -\frac1m \sum^m_{i=1} y^{(i)}X^{(i)}_j - \sigma(t) \cdot X^({i})_j \\
= \frac1m \sum^m_{i=1} (\hat{y}^{(i)} - y^{(i)}) X^{(i)}_j，
其中 \hat{y}^{(i)} = \sigma(X^{(i)}_b\theta) \\
牛逼
$$

向量化操作：
$$
\frac{dJ}{d\theta_j}
= \frac1m \sum^m_{i=1} (\hat{y}^{(i)} - y^{(i)}) X^{(i)}_j \\
所以，\\
\frac{dJ}{d\theta} = \sum^n_{j=0} \space(
\frac1m \sum^m_{i=1} (\hat{y}^{(i)} - y^{(i)}) X^{(i)}_j
\space) \\
= \frac1m \cdot X^T_b \cdot (\sigma(X_b\theta) - y)
$$

### 代码实现逻辑回归

```python
import numpy as np


class LogisticRegression:
    def __init__(self):
        self.coef_ = None
        self.intercept_ = None
        self._theta = None

    def _sigmoid(self, t):
        """获取y的值"""
        return 1 / (1.0 + np.exp(-t))

    def fit(self, X_train, y_train, eta=1, n_iters=1e4):
        assert X_train.shape[0] == y_train.shape[0], \
            'the size of X_train must be equal to the size of y_train.'

        def J(theta, X_b, y):
            """lost function"""
            y_hat = self._sigmoid(X_b.dot(theta))

            return -np.sum(y * np.log(y_hat) + (1 - y) * np.log(1 - y_hat)) / len(X_b)

        def dJ(theta, X_b, y):
            """gradient"""
            return X_b.T.dot(self._sigmoid(X_b.dot(theta)) - y) / len(X_b)

        def gradient_descent(X_b, y, initial_theta, eta, n_iters=1e4, epsilon=1e-6):
            theta = initial_theta
            cur_iter = 0
            while cur_iter < n_iters:
                gradient = dJ(theta, X_b, y)
                last_theta = theta
                theta = theta - eta * gradient
                if abs((J(theta, X_b, y)) - J(last_theta, X_b, y)) < epsilon:
                    break
                cur_iter += 1
            return theta

        X_b = np.hstack([np.ones((len(X_train), 1)), X_train])
        initial_theta = np.zeros(X_b.shape[1])
        self._theta = gradient_descent(X_b, y_train, initial_theta=initial_theta, eta=eta)
        self.intercept_ = self._theta[0]
        self.coef_ = self._theta[1:]
        return self

    def predict(self, X_predict):
        assert self.coef_ is not None and self.intercept_ is not None, \
            'must fit before predict'
        assert X_predict.shape[1] == len(self.coef_), \
            'the feature number of X_predict must be equal to X_train'

        X_predict_b = np.hstack([np.ones((len(X_predict), 1)), X_predict])
        y_predict = self._sigmoid(X_predict_b.dot(self._theta))
        y_probability = np.array(y_predict > 0.5, dtype=int)
        return y_probability

    def score(self, X_test, y_test):
        def accuracy_score(y_true, y_predict):
            assert y_true.shape[0] == y_predict.shape[0], \
                'the size of y_true must be equal to the size of y_predict'
            return np.sum(y_true == y_predict) / len(y_true)

        y_predict = self.predict(X_test)
        return accuracy_score(y_test, y_predict)

    def __repr__(self):
        return 'Logistic Regression()'
```

测试一下：

```python
from sklearn import datasets
from matplotlib import pyplot as plt
from sklearn.model_selection import train_test_split

iris = datasets.load_iris()
X, y = iris.data, iris.target
X = X[y < 2, :2]  # 只取两个标签由于只能二分类；只取两个特征供绘制图形
y = y[y < 2]

# 散点图
plt.scatter(X[y == 0, 0], X[y == 0, 1], color='red')  # 标签1，就是鸢尾花1
plt.scatter(X[y == 1, 0], X[y == 1, 1], color='b')  # 标签2，就是鸢尾花2

X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=666)

logis = LogisticRegression()
logis.fit(X_train, y_train)
score = logis.score(X_test, y_test)
print(score)  # 1.0
print(logis.predict(X_test))  # [1 1 0 0 0 0 0 1 1 1 0 0 0 0 1 1 1 0 0 0 0 0 1 1 0]
```

### 决策边界

这里的决策边界就是$\theta^T \cdot X_b = 0$，绘制一下：

```python
from sklearn import datasets
from matplotlib import pyplot as plt
from sklearn.model_selection import train_test_split

iris = datasets.load_iris()
X, y = iris.data, iris.target
X = X[y < 2, :2]
y = y[y < 2]

X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=666)

logis = LogisticRegression()
logis.fit(X_train, y_train)


# 绘出决策边界
def x2(x1):
    return (logis.coef_[0] * x1 + logis.intercept_) / (-logis.coef_[1])


x1_plot = np.linspace(4, 8, 1000)
x2_plot = x2(x1_plot)

plt.scatter(X_test[y_test == 0, 0], X_test[y_test == 0, 1], color='red')
plt.scatter(X_test[y_test == 1, 0], X_test[y_test == 1, 1], color='b')
plt.plot(x1_plot, x2_plot, color='#000')
```

这里可以看到一条直线，它就是决策边界。

#### 不规则边界的绘制方法

通过模拟更多的点（在平面上足够密），求出其预测值，然后根据类别就可以对整个区域进行绘制了。不同类别具有不同的背景色，就自动形成了边界。

说的有点乱，反正我也不会画

### 多项式逻辑回归

同样，使用sklearn中的`PolynomialFeatures`对数据进行升维：

```python
from matplotlib import pyplot as plt
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import PolynomialFeatures
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import train_test_split

np.random.seed(666)
X = np.random.normal(0, 1, size=(200, 2))
# 圆内为一个标签，圆外为一个标签
y = np.array(X[:, 0] ** 2 + X[:, 1] ** 2 < 1.5, dtype=int)
plt.scatter(X[y == 0, 0], X[y == 0, 1])
plt.scatter(X[y == 1, 0], X[y == 1, 1])

X_train, X_test, y_train, y_test = train_test_split(X, y)


def polynomial_logistic_regression(degree):
    return Pipeline([
        ('poly', PolynomialFeatures(degree=degree)),
        ('scaler', StandardScaler()),
        ('logis', LogisticRegression())
    ])


poly_regis = polynomial_logistic_regression(degree=2)

poly_regis.fit(X_train, y_train)
score = poly_regis.score(X_test, y_test)
print(score)  # 0.96
```

### 使用正则化

可以使用
$$
J(\theta) = \alpha L_2 \\
或 J(\theta) = \alpha L_1
$$
在`scikit-learn`中使用的是这样的方式：
$$
C \cdot J(\theta) + L_1 \\
或C \cdot J(\theta) + L_2
$$

```python
import numpy as np
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler, PolynomialFeatures
from sklearn.pipeline import Pipeline

np.random.seed(666)
X = np.random.normal(0, 1, size=(200, 2))
y = np.array(X[:, 0] ** 2 + X[:, 1] < 1.5, dtype=int)

# 添加噪音
for _ in range(20):
    y[np.random.randint(200)] = 1

X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=666)


def polynomial_logistic_regression(degree, C, penalty):
    return Pipeline([
        ('poly', PolynomialFeatures(degree=degree)),
        ('scaler', StandardScaler()),
        ('logis', LogisticRegression(C=C, solver='liblinear', penalty=penalty))  # 默认C=1.0, penalty='l2'
    ])


log_reg = polynomial_logistic_regression(degree=30, C=1, penalty='l1')
log_reg.fit(X_train, y_train)

print(log_reg.score(X_train, y_train))  # 0.94
print(log_reg.score(X_test, y_test))  # 0.94
```

### OvR 和 OvO

> 将二分类方法改造为可用于多分类的通用方法，并不是仅逻辑回归可用的。
>
> OvR: One versus Rest (也可能叫OvAll)
>
> OvO: One versus One

#### OvR

进行分类任务：

1. 每一次选出其中一个类别，剩余所有类别归结为一个类别；

2. 进行二分类任务；
3. 选择分类得分最高的结果。

时间复杂度：$n \cdot time$

#### OvO

进行分类任务：

1. 每一次选出其中两个类别；
2. 进行二分类任务；
3. 选择分类得分最高的结果。

时间复杂度：$C_n^2 \cdot time = \frac{n(n-1)}{2} \cdot time$

它的分类准确度更高，但是耗时更高。

#### 在LogisticRegression中使用

```python
log_reg = LogisticRegression(multi_class='ovr')  # OvR
# 如果要使用ovo，必须更改solver字段
log_reg2 = LogisticRegression(multi_class='multinomial', solver='newton-cg')  # OvO
```

#### sklearn封装的OvR和OvO

```python
from sklearn import datasets
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from sklearn.multiclass import OneVsOneClassifier, OneVsRestClassifier

iris = datasets.load_iris()
X = iris.data[:, :2]
y = iris.target

X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=666)

log_reg = LogisticRegression()

ovr = OneVsRestClassifier(log_reg)
ovr.fit(X_train, y_train)
print(ovr.score(X_test, y_test))  # 0.7894736842105263

ovo = OneVsOneClassifier(log_reg)
ovo.fit(X_train, y_train)
print(ovo.score(X_test, y_test))  # 0.8421052631578947
```

## 评价分类结果

### 准确度陷阱

> 如果一个癌症预测系统的预测准确度是99.9%，这个系统好吗？
>
> 不一定。假如癌症产生的概率只有0.1%；我们预测所有人都健康，那么预测准确度就是99.9%。

对于极度偏斜的数据(Skewed Data)，只使用分类准确度是远远不够的。

### 混淆矩阵(Confusion Matrix)

|      | 0                   | 1                   |
| ---- | ------------------- | ------------------- |
| 0    | TN(True Negatives)  | FP(False Positives) |
| 1    | FN(False Negatives) | TP(True Positives)  |

行代表真实值(第一维度)，列代表预测值(第二维度)；0代表阴性(Negative)，1代表阳性(Positive)。

### 精准率(Precision)和召回率(Recall)

$$
Precision = \frac{TP}{TP + FP} \\
Recall = \frac{TP}{TP + FN}
$$

精准率：预测值为1的当中，预测成功的概率

召回率：真实值为1的当中，预测成功的概率

#### 实现

```python
import numpy as np
from sklearn import datasets
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression

digits = datasets.load_digits()
X = digits.data
y = digits.target.copy()

# 把数据变成二分类问题
y[digits.target == 9] = 1
y[digits.target != 9] = 0

X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=666)

log_reg = LogisticRegression(max_iter=5000)
log_reg.fit(X_train, y_train)
y_predict = log_reg.predict(X_test)


# True Negatives
def TN(y_true, y_predict):
    if len(y_true) != len(y_predict):
        raise ValueError('the size of y_true must be equal to the size of y_predict')
    # 返回真值和预测值都为0的个数
    return np.sum((y_true == 0) & (y_predict == 0))


def FN(y_true, y_predict):
    return np.sum((y_true == 1) & (y_predict == 0))


def TP(y_true, y_predict):
    return np.sum((y_true == 1) & (y_predict == 1))


def FP(y_true, y_predict):
    return np.sum((y_true == 0) & (y_predict == 1))


def confusion_matrix(y_true, y_predict):
    return np.array([
        [TN(y_test, y_predict), FP(y_test, y_predict)],
        [FN(y_test, y_predict), TP(y_test, y_predict)]
    ])


def precision_score(y_true, y_predict):
    tp = TP(y_true, y_predict)
    fp = FP(y_true, y_predict)
    try:
        return tp * 1.0 / (tp + fp)
    except:
        return 0.0


def recall_score(y_true, y_predict):
    tp = TP(y_true, y_predict)
    fn = FN(y_true, y_predict)
    try:
        return tp * 1.0 / (tp + fn)
    except:
        return 0.0


print(confusion_matrix(y_test, y_predict))  # [ [403   2], [9   36] ]
print(precision_score(y_test, y_predict))  # 0.9473684210526315
print(recall_score(y_test, y_predict))  # 0.8
```

#### scikit-learn中的使用

```python
import numpy as np
from sklearn import datasets
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import confusion_matrix, precision_score, recall_score

digits = datasets.load_digits()
X = digits.data
y = digits.target.copy()

# 把数据变成二分类问题
y[digits.target == 9] = 1
y[digits.target != 9] = 0

X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=666)

log_reg = LogisticRegression(max_iter=5000)
log_reg.fit(X_train, y_train)
y_predict = log_reg.predict(X_test)

print(confusion_matrix(y_test, y_predict))  # [ [403   2], [9   36] ]
print(precision_score(y_test, y_predict))  # 0.9473684210526315
print(recall_score(y_test, y_predict))  # 0.8
```

### 精准率和召回率的含义

对于不同的算法，可能会导致不同的精准率和召回率。

**比如股票预测，我们更关注精准率。**1代表升值；精准率代表在所有预测升值的次数当中的成功率；召回率代表所有升值股票的数量中预测到的比例。

**比如病人诊断，我们更关注召回率。**1代表阳性；精准率代表在所有预测得病的数量当中的成功率；召回率代表所有患病病人的数量中预测到的比例。

还有很多情况，是综合来看精准率和召回率的。这时候，我们使用**F1 Score**。

F1 Score 是两者的调和平均值。

> 对于调和平均值来说，不同于算术平均值，有一个参数特别小时，结果也会特别小。

$$
\frac1{F1} = \frac12 (\frac1{precision} + \frac1{recall}) \\
F1 = \frac{2 \cdot precision \cdot recall}{precision + recall}
$$

```python
def f1_score(precision, recall):
    try:
        return 2.0 * precision * recall / (precision + recall)
    except:
        return 0.0


print(f1_score(0.1, 0.9))  # 0.18
```

#### 在sklearn中的使用

> 传递参数是 y_true 和 y_predict

```python
from sklearn.metrics import f1_score

f1_score(y_test, y_predict)
```

### Precision-Recall 的平衡

> 这两个指标往往是冲突的；其中一个值提高的情况下，另一个值不可避免的会减小。

$$
\bigstar \bigstar \bigstar \bigstar \mid \spadesuit \bigstar \spadesuit \mid \spadesuit \bigstar \mid \spadesuit \spadesuit \spadesuit
$$

上面是一个二分类问题，$\bigstar$代表0，$\spadesuit$代表1。

1. 如果按照中间那根线作为`决策边界`，$Precision = \frac45, Recall = \frac46$；
2. 如果按照左边那根线作为`决策边界`，$Precision = \frac68, Recall = 1$；
3. 如果按照右边那根线作为`决策边界`，$Precision = 1, Recall = \frac36$。

一般情况下，我们分类的`决策边界`都是$\theta^T \cdot X_b = 0$，如果我们改变这个阈值(threshold)，就是改变决策边界。

```python
import numpy as np
from sklearn import datasets
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import confusion_matrix, precision_score, recall_score

digits = datasets.load_digits()
X = digits.data
y = digits.target.copy()

# 把数据变成二分类问题
y[digits.target == 9] = 1
y[digits.target != 9] = 0

X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=666)

log_reg = LogisticRegression(max_iter=5000)
log_reg.fit(X_train, y_train)
y_predict = log_reg.predict(X_test)

print(precision_score(y_test, y_predict))  # 0.9473684210526315
print(recall_score(y_test, y_predict))  # 0.8

"""现在，我们希望回归率更高一些"""

decision_scores = log_reg.decision_function(X_test)
print(np.min(decision_scores))  # -85.76652882645814
print(np.max(decision_scores))  # 19.97568937249484
# 降低阈值
y_predict2 = np.array(decision_scores >= -5, dtype=int)
print(precision_score(y_test, y_predict2))  # 0.7142857142857143
print(recall_score(y_test, y_predict2))  # 0.8888888888888888
```

### 精准率-召回率曲线

> 通过尽量多的可能的阈值，来绘制精准率-召回率的可能的情况。

代码：

```python
import numpy as np
from sklearn import datasets
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import precision_score, recall_score
from matplotlib import pyplot as plt

digits = datasets.load_digits()
X = digits.data
y = digits.target.copy()

# 把数据变成二分类问题
y[digits.target == 9] = 1
y[digits.target != 9] = 0

X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=666)

log_reg = LogisticRegression(max_iter=5000)
log_reg.fit(X_train, y_train)

decision_scores = log_reg.decision_function(X_test)
precisions = []  # precision scores
recalls = []  # recall scores

thresholds = np.arange(np.min(decision_scores), np.max(decision_scores), 0.1)  # thresholds
for t in thresholds:
    y_predict = np.array(decision_scores >= t, dtype=int)
    precisions.append(precision_score(y_test, y_predict))
    recalls.append(recall_score(y_test, y_predict))

# 在thresholds维度上，precisions和recalls的变化：threshold增加时，precision在增加，recall在减小
plt.plot(thresholds, precisions)
plt.plot(thresholds, recalls)

# Precision-Recall Curve：在一个指标增加的同时，另一个处于下降的状态
plt.plot(precisions, recalls)
```

#### 在scikit-learn中使用

```python
from sklearn import datasets
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from matplotlib import pyplot as plt
from sklearn.metrics import precision_recall_curve

digits = datasets.load_digits()
X = digits.data
y = digits.target.copy()

# 把数据变成二分类问题
y[digits.target == 9] = 1
y[digits.target != 9] = 0

X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=666)

log_reg = LogisticRegression(max_iter=5000)
log_reg.fit(X_train, y_train)

decision_scores = log_reg.decision_function(X_test)

precisions, recalls, thresholds = precision_recall_curve(y_test, decision_scores)

"""
其中最后一个precision的值是1，recall是0，threshold没有对应值
所以，需要将precisions和recalls去掉最后一个值
"""

plt.plot(thresholds, precisions[:-1])
plt.plot(thresholds, recalls[:-1])
```

#### Precision-Recall曲线的意义

多个模型下，如果其中一个模型的Precision-Recall曲线更靠外，就是占的面积更大，那么这个模型大概率就更好。

### ROC曲线

> Receiver Operation Characteristic Curve，它描述了TPR和FPR之间的关系。
>
> TPR(True Positive Rate): $\frac{TP}{TP + FN}$，预测为1且预测正确占真实值为1的百分比。越高越好。
>
> FPR(False Positive Rate): $\frac{FP}{TN + FP}$，预测为1且预测错误占真实值为0的百分比。越低越好。
>
> 一般情况下，TPR和FPR是成正比的。

ROC曲线的面积越大，模型越好。

```python
from sklearn import datasets
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from matplotlib import pyplot as plt
from sklearn.metrics import roc_curve, roc_auc_score

digits = datasets.load_digits()s
X = digits.data
y = digits.target.copy()

# 把数据变成二分类问题
y[digits.target == 9] = 1
y[digits.target != 9] = 0

X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=666)

log_reg = LogisticRegression(max_iter=5000)
log_reg.fit(X_train, y_train)

decision_scores = log_reg.decision_function(X_test)

fprs, tprs, thresholds = roc_curve(y_test, decision_scores)
plt.plot(fprs, tprs)  # 绘制曲线

# 曲线面积
print(roc_auc_score(y_test, decision_scores))  # 0.9868861454046639
```

