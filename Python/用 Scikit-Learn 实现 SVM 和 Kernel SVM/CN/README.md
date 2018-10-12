## 用 Scikit-Learn 实现 SVM 和 Kernel SVM

支持向量机（SVM）是一种监督学习分类算法。支持向量机提出于 20 世纪 60 年代在 90 年代得到了进一步的发展。然而，由于能取得很好的效果，最近才开始变得特别受欢迎。与其他机器学习算法相比，SVM 有其独特之处。


本文先简明地介绍支持向量机背后的理论和如何使用 Python 中的 Scikit-Learn 库来实现。然后我们将学习高级 SVM 理论如 Kernel SVM，同样会使用 Scikit-Learn 来实践。




### 简单 SVM

考虑二维线性可分数据，如图 1，典型的机器学习算法希望找到使得分类错误最小的分类边界。如果你仔细看图 1，会发现能把数据点正确分类的边界不唯一。两条虚线和一条实线都能正确分类所有点。


![](https://user-gold-cdn.xitu.io/2018/8/24/1656b52b11b94056?imageslim)



图 1：多决策边界
SVM 通过最大化所有类中的数据点到决策边界的最小距离的方法来确定边界，这是 SVM 和其他算法的主要区别。SVM 不只是找一个决策边界；它能找到最优决策边界。

能使所有类到决策边界的最小距离最大的边界是最优决策边界。如图 2 所示，那些离决策边界最近的点被称作支持向量。在支持向量机中决策边界被称作最大间隔分类器，或者最大间隔超平面。


![](https://user-gold-cdn.xitu.io/2018/8/24/1656b52b13a9809c?imageslim)


图 2：决策边界的支持向量

寻找支持向量、计算决策边界和支持向量之间的距离和最大化该距离涉及到很复杂的数学知识。本教程不打算深入到数学的细节，我们只会看到如何使用 Python 的 Scikit-Learn 库来实现 SVM 和 Kernel-SVM。


### 通过 Scikit-Learn 实现 SVM



我们的任务是通过四个属性来判断纸币是不是真的，四个属性是小波变换图像的偏度、图像的方差、图像的熵和图像的曲率。我们将使用 SVM 解决这个二分类问题。剩下部分是标准的机器学习流程。

#### 导入库

下面的代码导入所有需要的库：


    import pandas as pd
    import numpy as np
    import matplotlib.pyplot as plt
    %matplotlib inline


#### 导入数据


数据可以从下面的链接下载：

[drive.google.com/file/d/13nw…](https://link.juejin.im/?target=https%3A%2F%2Fdrive.google.com%2Ffile%2Fd%2F13nw-uRXPY8XIZQxKRNZ3yYlho-CYm_Qt%2Fview)

数据的详细信息可以参考下面的链接：

[archive.ics.uci.edu/ml/datasets…](https://link.juejin.im/?target=https%3A%2F%2Farchive.ics.uci.edu%2Fml%2Fdatasets%2Fbanknote%2Bauthentication)

从 Google drive 链接下载数据并保存在你本地。这个例子中数据集保存在我 Windows 电脑的 D 盘 “Datasets” 文件夹下的 CSV 文件里。下面的代码从文件路径中读取数据。你可以根据文件在你自己电脑上的路径修改。

读取 CSV 文件的最简单方法是使用 pandas 库中的 read_csv 方法。下面的代码读取银行纸币数据记录到 pandas 的  dataframe:


    bankdata = pd.read_csv("D:/Datasets/bill_authentication.csv")


### 探索性数据分析
使用 Python 中各种各样的库几乎可以完成所有的数据分析。为了简单起见，我们只检查数据的维数并查看最前面的几条记录。查看数据的行数和列数，执行下面的语句：


    bankdata.shape


你将看到输出为（1372, 5）。这意味着数据集有 1372 行和 5 列。

为了对数据长什么样有个直观感受，可以执行下面的命令：

    bankdata.head()

输出下面如下:

![](https://user-gold-cdn.xitu.io/2018/8/24/1656b52aa327b0f8?imageslim)

你可以发现所有的属性都是数值型。类别标签也是数值型即 0 和 1。


### 数据预处理
数据预处理包括（1）把属性和类表标签分开和（2）划分训练数据集和测试数据集。

把属性和类别标签分开，执行下面的代码：

    X = bankdata.drop('Class', axis=1)
    y = bankdata['Class']


上面代码第一行从 bankdata dataframe 中移除了类别标签列 “Class” 并把结果赋值给变量 X。函数 drop() 删除指定列。

第二行，只将类别列存储在变量 y 里。现在变量 X 包含所有的属性变量 y 包含对应的类别标签。

现在数据集已经将属性和类别标签分开，最后一步预处理是划分出训练集和测试集。幸运的是 Scikit-Learn 中 model_selection 模块提供了函数 train_test_split 允许我们优雅地把数据分成训练和测试两部分。

执行下面的代码完成划分

    from sklearn.model_selection import train_test_split
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size = 0.20)


### 算法训练

我们已经把数据分成了训练集和测试集。现在我们使用训练集来训练。Scikit-Learn 库中的 svm 模块实现了各种不同的 SVM 算法。由于我们要完成一个分类任务，我们将使用由 Scikit-Learn 中 svm 模块下的 SVC 类实现的支持向量分类器。这个类需要一个参数指定核函数的类型。这参数很重要。这里考虑最简单的 SVM 把类型参数设为 linear 线性支持向量机只适用于线性可分数据。我们在下一部分介绍非线性核。
把训练数据传给 SVC 类 fit 方法来训练算法。执行下面的代码完成算法训练：


    from sklearn.svm import SVC
    svclassifier = SVC(kernel='linear')
    svclassifier.fit(X_train, y_train)

### 做预测

SVC 类的 predict 方法可以用来预测新的数据的类别。代码如下：

    y_pred = svclassifier.predict(X_test)


