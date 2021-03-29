## ML Strategy

### 什么是ML strategy

根据经验迅速定位机器学习模型算法遇到的问题，然后去改善它。

### 正交化(Orthogonalization)

就是针对不同的阶段，遇到不同的问题时，做出区分、的有效的处理。

### 单一数字评估指标(single number evaluation metric)

精准率(Precision)：预测为1成功的数量 / 预测为1的数量 (True Positive / TP + FP)

召回率(Recall)：预测为1成功的数量 / 真实为1的数量 (True Positive / TP + FN)

两个评估指标并不方便比较，所以更平常的是使用 F1 Score 作为评估指标。F1 Score 是 Precision 和 Recall 的调和平均值
$$
f1\_score = \frac2{\frac1{precision} + \frac1{recall}}
$$

### 满足和优化指标

当有N个指标时，通常的做法是，选择一个指标进行优化工作，其余 N-1 个指标只需要满足某个阈值即可。

### Train/dev/test 

确保训练数据集和测试数据集来自于同一分布(same distribution)。

数据集大小：

1. 如果数据量很小(如1k)，可以使用传统的切割数据的方式(old way)，60%/20%/20%
2. 灵活分割数据集。如1m，则可以使用 98%/1%/1% 的比例

评估指标根据情况来修改，比如：识别猫的两个模型，模型A误差为3%，模型B误差为5%，但是模型A会把色情图片认为是猫；那么此时可以增加A对于色情图片错误认识的惩罚，就是设置识别色情图片为猫时的损失值更大(如10倍)。

### 人的表现(Human level performance)

通常人的表现是比较接近**贝叶斯最优误差(Bayes optimal error)**，比如人的误差是7.5%(human-level error)，模型的训练误差是8%，那么；如果人的误差是1%、即贝叶斯误差大概是小于1%一点点，模型的训练误差是8%，那么就需要优化偏差。

可避免偏差(avoidable bias)：训练误差 - 贝叶斯误差。注意训练误差不能小于贝叶斯误差，那样的话大概率是过拟合了。但是贝叶斯误差值其实也只是一个估值，所以究竟是过拟合还是更好的性能，在模型接近人的表现时，它的提升会越来越困难。

### 超越人的表现(Surpassing human-level performance)

对于结构化的数据(structured data)，比如在线广告、商品推荐等，超越人是更简单的，因为计算机可以对海量数据进行统计分析。对于自然感知任务(natural perception task)，如计算机视觉、自然语言处理等，超越人的表现是较难的，因为人最擅长这类任务。

### 改善模型表现

降低偏差：

1. 更大数据集
2. 更好的算法如Adam
3. 更好的超参数或更好的模型架构如RNN、CNN

降低方差：

1. 更多数据
2. 正则化、随机失活(dropout)

## ML Strategy 2

### 误差分析(Error analysis)

误差分析就是将模型对开发集(dev set)的预测结果的错误部分进行分析。把所有预测错误的部分拿出来，然后统计错误记录。比如对猫的预测：大猫图片(狮子、豹)、狗图片、模糊图片、Instagram滤镜等等，可以得出哪些因素是导致误差变大的。

### 数据集的标签错误

数据集可能会有标签错误的情况。比如一张猫的图片，被标记为不是猫。

对于少量的随机错误(random errors)，ML算法是不太在意的。仍然可以通过误差分析，加上标签错误一栏，然后观察权重较大的误差。如果标签错误占比较高则需要修改，如果占比不高则优先修改权重更大的误差。

但是对于系统错误(systemic errors)，ML算法是会受到很大影响的。比如，所有白色狗狗都被标记为猫。

### 划分数据集

很多时候数据集的来源是多方的，比如：

猫识别模型，200K来自于网络爬虫，10K来自于用户上传，此时可以将200+5K作为训练集，2.5K作为开发集，2.5K作为测试集。

### 误差原因

很多时候训练集和开发集不具有相同的分布。可以将 training set 再分一小部分出来，成为 training-dev set。

1. Human error 和 Training error 的差，是偏差(avoidable bias)
2. Training error 和 Training-dev error 的差，是方差(variable)
3. Training-dev error 和 Dev error 的差，是数据不匹配(data mismatch)
4. Dev error 和 Test error 的差，是过拟合(overfitting)

### 数据不匹配

数据不匹配的情况，比如说作语音识别，在训练集的语音都是清晰的，但是开发集的语音都是有汽车背景声的。

此时可以人工合成语音，将汽车背景声合成到训练集的数据中。注意，尽量选取不同的汽车背景声到不同的语音，否则模型很可能过拟合，把汽车背景声认作为特征的一部分。

### 迁移学习(transfer learning)

就是从一个任务学习，迁移到另一个任务的学习。比如模型可以识别猫，迁移到让它去识别狗。

### 多任务学习(Multi-task learning)

需要足够大的数据量。使用较少。

### 端到端的深度学习(End-to-end DL)

比如要做人脸识别的自动门。一般的设计是：先识别人找到人脸，再判断人脸的身份。

端到端就是一步到位，直接从给定数据得到结果。

端到端需要大量的数据。而且很多场景都是不适用的，比如自动驾驶，不能通过一张图片就判断汽车是加速还是减速或者转动方向。