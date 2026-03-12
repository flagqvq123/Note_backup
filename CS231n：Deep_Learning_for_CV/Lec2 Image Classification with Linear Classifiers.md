
>[!tasks]+ Today:
>- The image classification task
>- Two basic data-driven approaches to image classification(图像分类的两种基本数据驱动方法)
>	- K-nearest neighbor and linear classifier

##### Image Classification: CV的核心任务
	给定图像，在一些备选项的分类中选出一个最有可能的输出。

- 机器如何接受这些数据？
	- 由每一个取值介于[0, 255]之间的数，组成的大小为[x, y, 3]的向量构成的矩阵。(3channels RGB)
	- Challenges: 
		- 移动：对摄像机即使简单的的移动会完全改变数据的位置
		- 光照：同一对象在涉及到不同光照条件会导致数据变化
		- 背景：背景会对识别产生噪音
		- 分辨率、缩放
		- “闭塞”：对象不在图片中显示完全或者“藏”在背景中
		- 变形
		- 类内变异：对同一种类可以有不同颜色、不同种类...
		- 内容：例如光影投射的阴影可能被识别为条纹...

- An image classifier:
```
	def classify_image(image):
	#...magic codes
	return class_lable
```
	对图像的识别算法并不像一般算法一样简单和直观。

- Attempts have been made
	- 寻找图像边缘，寻找拐角，并进行识别。
	- 尽管它已经取得了一些成效，然而依旧具有局限性；
	- 基于逻辑、程序的算法设计在图像识别领域并不成功。也许带有数据驱动的方法会是更好的。

##### Machine Learning: Data-Driven Approach
	三步方案
- 1.收集图像和标签的数据集
	使用互联网等
- 2.使用机器学习算法来训练分类器。
- 3. 在新的图像数据集中测试这个分类器。
```
	def train(images, lables):          # 训练函数，构造出模型
		# Machine leaning!
		return model
	
	def predict(model, test_images):    # 预测函数，返回预测的标签
		# Use model to predict labels
		return test_labels	
```

### Nearest Neighbor Classifier
- 训练函数仅需记录所有学习过的数据和标签
- 预测函数寻找最相似的图像并且返回其标签

##### Distance Metric: 定义两张图片的差值，并且返回一个数。
	定义了两张图片之间的相似性，有许多不同方法可以做到这一点。
- L1 distance：求出所有像素矩阵的差，取绝对值相加作为差值。
	Q: 在N个数据的情况下，这样的算法时间复杂度是？
	A: Train O(1), but predict O(N)
	我们想要的是预测时间更少，而训练时间无所谓。