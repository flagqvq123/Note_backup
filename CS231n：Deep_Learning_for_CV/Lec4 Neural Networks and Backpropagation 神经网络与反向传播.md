### Recap 回顾
- loss func 与 正则化
	- 线性分类器、打分函数、标准化
	- Softmax function 转化为概率
		- 还有一些其他的loss func
- 优化：对神经网络找到最适合的参数
	- SGD，batch
	- 梯度下降算法、动量、降温(RMSProp)、Adam、AdamW
	- 学习率与迭代次数的关系

### Neural networks 神经网络
- 从线性分类器到神经网络
	- $f=Wx, x\in R^D,W\in R^{C ×D}$
	- C是输入的维数，D是输出的数量
- 在第二层建立神经网络：
	- $f = W_2max(0,W_1x),x\in R^D,W_1 \in R^{H×D}, W_2 \in R^{C×H}$
	- H定义了隐藏层的输出量
		- 在实际上，我们会在里面添加bias偏移量
	-  max函数的意义？：创造“非线性”
	- non-linearity：非线性
		- 线性的分类器具有天然的缺陷
		- 我们需要一种变换，将原来的分类变为线性分类可完成的形式。
	- 此类型的网络：仅取决于W、输入、层，计算方式全是乘法：全连接网络、MLP
- 隐藏层的深层含义：创造"部分模板“
- 删除max之后：
	- $f=W_2W_1x =W_3x$ 这实际上与单层的线性分类器没有任何区别
	- 这种非线性函数的名称是Activation functions
		- ReLU(max(0,x))
		- 其他选择:ELU、GELU......
		- Sigmoid、Tanh （可能有问题会导致线性消失）
		- ReLU是一个很不错的默认选择。

- example: 一个三层神经网络
```
# forward-pass of a 3-layer neural network:
f = lambda x: 1.0/(1.0 + np.exp(-x)) # activation function (here use sigmoid)
x = np.random.randn(3,1)             # random input vector of 3 numbers (3×1)
h1 = f(np.dot(W1, x) + b1)           # calculate first hiden layer activations(4×1)
h2 = f(np.dot(W2, h1) + b2)          # calculate second hidden layer activations
out = np.dot(W3, h2) + b3            # output neuron
```

- Setting the number of layers and their sizes 如何设计中间层数量和大小？
	- 更多的神经元意味着更有可能的过拟合
	- 不要使用神经网络的大小来作为一个“正则化器”，而是使用正则化来调整过拟合现象。