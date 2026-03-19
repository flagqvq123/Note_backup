### 复习：
- 图像分类的概念：
	- 将图像映射到几个给定的标签
- 分类的挑战
	- 像素的网格难以描述人眼能看见的特征
	- 图像本身的难度，例如光影、隐藏、变形等人类也需要学习来克服的挑战
	- 这样的挑战让我们难以使用逻辑来分类，所以我们转向数据驱动的学习
- KNN模型：
	- 分配给最“近”邻居的标签
	- 训练集、验证集、测试集的概念
	- 超参数：K、距离函数。
		- 两种不同的距离度量：曼哈顿距离与几何距离
		- 几何距离最优秀的一点在于它保持“距离”不变，这让旋转成为一种不变量
- 线性分类器：
	- 使用线性函数，用权重矩阵乘以图像的矩阵来得到一个关于每一个分类概率的向量
	- 线性模型的三种观点：
		- 每一个分类都是独立的一组向量乘法，得到一个“score”。
		- 可视化分类器的每一组权重，将它们分离得到什么样的图像在这种分类中更青睐
		- 几何观点：在超空间中的一个超平面，将整个图像分为两部分。
			- 这也告诉我们线性模型具有局限性，它不能对所有图像进行分类。（比如这种类型在空间中是一个圆）

##### Loss Function ：一个模型到底好还是坏
- 一整个模型对测试集预测的好坏由loss定义
	- loss被设计为较高时表示模型拟合差、较低时模型拟合好
	- 整个测试集的loss是每一个样本loss的平均数

### 正则化：在训练中更差，在实际中更好
- 正则化的来源：拟合与过拟合
	- 从直觉上来讲，过拟合不一定让拟合变得更精准，反而会让拟合变差
	- 我们更喜欢更简单的拟合。
- 正则化：在线性拟合之后增加一项来改变loss惩罚：$$L(W) = \frac 1N\sum\limits_{i=1}^{n}L(f(x_i,W),y_i) + \lambda R(W)$$
- $\lambda$：正则化强度，0~无穷，超参数。
- 一些简单的例子：
	- L2正则化：$R(W) = \sum_k\sum_lW_{k,l}^2$
		- 将权重矩阵的每一项平方作为正则函数和
		- 当有极小值的情况会变得更小，所以对于极小的value可以近乎忽略
	- L1正则化：$R(W)=\sum_k\sum_l|W_{k,l}|$
		- 直接将权重矩阵绝对值求和
		- L1会得到很多零值。
- 正则化在优化过程中也会改变，并且在实质上增加这一项的确有利于模型的改进。
- 更复杂的正则化模型......
- 为什么要使用正则化模型？
	- 对权重进行偏好的设定（用不同的正则化模型让权重更稀疏？密集？更多0？）
	- 让模型更 *simple* 而不是过拟合
	- 改进优化过程
- 总结：
	- 数据集
	- 损失函数Softmax
	- 正则化得到full loss

### 优化：找到损失函数的最小值
	一个被蒙住眼睛的人应该怎么找到一个“最低点”？
	“维度” -> 每一个数据就代表一个维度
- 策略1：随机选点找到选点的最低点
- 策略2：顺着“斜坡”走
	- 如何沿着斜坡走？
		- 1维导数：在某点进行微小位移，得到高度上的。
		- 多维度：梯度：分别求出几个正交坐标轴的导数，组合成为一个向量，其指向的方向就是“斜坡”的朝向。梯度向量指向的反方向就是“下山”最快的方向。
	- 如何计算梯度？
		- 对每一个方向加上一个微小步长（保持其他不变），求出损失函数变化量来求出梯度。
			- 问题：每一步都要算一次，非常耗时间。
		- 数学方法：通过链式法则等方式求导得出每一个方向的偏导数。
			- 微积分、链式法则......

### 梯度下降方法：Gradient Descent
```
# Vanilla Gradient Descent

while True:
	weights_grad = evaluate_gradient(loss_func, data, weights)
	// 权重的梯度矩阵 = 计算梯度方法(损失函数, 数据集, 权重矩阵)
	// 这里损失函数将数据看作参数, 权重看为变量, 用偏导等计算方式获得权重矩阵的梯度
	weights += - step_size * weights_grad # 向前“走”一步, 更新权重
```
- 每一步的步长相同，然而由于逐渐到底之后梯度变小，有效步长(loss改变量)变小。
- 什么时候停下？
	- 到达预期数量的迭代数停止
	- 直到它的减少量过小时停止

##### Stochastic Gradient Descent (SGD) 随机梯度下降，梯度下降算法的一种变体
- 如果有巨大数据集的时候，计算L(W)是一个巨大的任务
- SGD思路：每次仅仅随机采样一个minibatch，在随机的子集上运行梯度下降。
```
# Vanilla Minibatch Gradient Descent

while True:
	data_batch = sample_training_data(data, 256) # 选择一个256的小参数子集
	weights_grad = evaluate_gradient(loss_fun, data_batch, weights)
	weights += - step_size * weights_grad
	// 每次仅仅对一个小子集进行迭代，减少了开销
```
- 在实践中，人们不会完全随机采样。我们会确保一个周期内所有的数据点都会被训练一遍，这一循环被称为一个`epoch`
- 可能会出现的问题：
	- 1. 过长的步长可能导致跳出山谷
	- 2. 震荡导致收敛速度很慢
	- 3. 局部最小值，或者鞍点（该点的梯度为0，无法继续前进）
	- 4. 收敛过程很嘈杂，因为只看子集不能保证每一步都向最低点移动。
- SGD + Momentum
	给收敛的过程增加一个动力，让它在收敛过程更快，且方向性会朝最低点累加。
	$$v_{t+1}=\rho v_t+\nabla f(x_t)$$$$x_{t+1} = x_t - \alpha v_{t+1}$$ 
	 梯度在这里充当加速度的方向，其中的参数：
	- $\rho$意思是动量，决定继承多少上一次的速度
	- $\alpha$意思是步长
```
vx = 0
while True:
	dx = compute_gradient(x)
	vx = rho * vx + dx
	x -= learning_rate * vx
```
	就我自己的理解，它更像是添加了一种惯性，让它就像小球滚落一样更快，并且由于rho的存在保证了一种能量的损耗，让它不会冲出山谷。并且具有惯性的情况下会让“不平整的噪声地形”被惯性抵消，这就保证了它不会歪歪扭扭而收敛过慢。

- 一些缺点：
	- 由于关系，它的收敛不一定会更快，但是它会更容易找到更低的点。

##### 更复杂的优化器：RMSProp
- 基于每个维度中平方的历史和 (带有衰减) 添加逐元素梯度缩放
- 放大梯度中更小的，缩小里面更大的，能让我们在陡峭的地方走的更慢，平坦的地方走更快。
```
grad_squared = 0 # 累计平方梯度矩阵
while True:
	dx = compute_gradient(x) # 梯度矩阵
	grad_squared = decay_rate * grad_squared + (1 - decay_rate) * dx * dx
		// decay_rate 衰变率
		// 由于平方的作用，陡峭的方向(大梯度)其平方会变得更大
		// 在持续的积累下，一直大的方向就会一直保持其平方梯度大
	x -= learning_rate * dx / (np.sqrt(grad_squared) + 1e-7)
		// 将更新项除以平方梯度的平方根
		// 这里就能看出来，若之前的grad_squared项一直很大，那么向这个方向的前进量就会变小。
		// 这缩小了陡峭方向和平坦方向的选择
```

### Adam（最常用的优化器）
```
first_moment = 0
second_moment = 0
while True:
	dx = compute_gradient(x)
	first_moment = beta1 * first_moment + (1 - beta1) * dx
	second_moment = beta2 * second_moment + (1 - beta2) * dx * dx
	x -= learning_rate * first_moment / (np.sqrt(second_moment) + 1e-7)
```
- 它实质上是两种方法的混合。
	- first_moment -> Momentum
	- second_moment -> RMSProp
- 既添加速度概念，又降温陡峭增温平坦。
- 一些关于beta的行为：
	- 由于初始化beta通常接近于1，所以second_moment 在初始化的时候可能会接近于0。
	- 所以我们需要添加`Bias`项避免这种情况的出现。
		- `bias`修正了`first_moment`和`second_moment`初始量为0的情况
		- 此处的t代表时间步长，unbias项随着时间增加会越来越趋向于原项
		- Adam中最好使用`beta1 = 0.9, beta2 = 0.999, learning_rate = 1e-3 or 5e-4`的参数，适用于大部分模型
```
first_moment = 0
second_moment = 0
while True:
	dx = compute_gradient(x)
	first_moment = beta1 * first_moment + (1 - beta1) * dx
	second_moment = beta2 * second_moment + (1 - beta2) * dx * dx
	first_unbias = first_moment / (1 - beta1 ** t)
	second_unbias = second_moment / (1 - beta2 ** t)
	x -= leaning_rate * first_unbias / (np.sqrt(second_unbias) + 1e-7)
```

##### AdamW：Adam Variant with Weight Decay 带权重衰减的Adam
- example：正则化如何与这个优化器作用？(比如L2正则化)
	- A：不同情况下有不同的作用方式！
	- 