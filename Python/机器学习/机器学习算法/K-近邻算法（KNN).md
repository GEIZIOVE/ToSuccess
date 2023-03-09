1.K-近邻算法
---------------------------------------------------------------------------

​		 K近邻（K-Nearest Neighbor, KNN）是一种最经典和最简单的_有监督学习_方法之一。K-近邻算法是最简单的分类器，_没有显式的学习过程或训练过程_，是_懒惰学习_（Lazy Learning）。当对数据的分布只有很少或者没有任何先验知识时，K 近邻算法是一个不错的选择。

​	

> K近邻算法既能够用来解决分类问题，也能够用来解决回归问题。该方法有着非常简单的原理：
>
> 当对测试样本进行分类时，首先通过扫描训练样本集，找到与该测试样本最相似的个训练样本，根据这个样本的类别进行投票确定测试样本的类别。也可以通过个样本与测试样本的相似程度进行加权投票。如果需要以测试样本对应每类的概率的形式输出，可以通过个样本中不同类别的样本数量分布来进行估计。

1.  三个基本要素
    
     **$k$  值的选择**、**距离度量**和**分类决策规则**
    
2.  为什么不具有显式的学习过程或训练过程？
    

### 1.1小例子

```tex
简单说就是如果测试样本在特征空间中的k个最邻近的样本中，大多数样本属于某个类别，则该测试样本也划分到这个类别，KNN里的K就是最邻近的K个数据样本。
```

![image-20210121155819348](E:/Development/Typora/images/e211536cca024535725c4c4a72353d03-1676947286251-1.png)

要确定绿圆属于哪个类别，如果k=3，在其最近的3个样本中红色三角形数量最多，绿圆属于红色三角形类别，如果k=5，在其最近的5个样本中蓝色矩形数量最多，绿圆属于蓝色矩形类别，可见k的选择很重要（k的选择我们后面再讨论）。

### 1.2评价

在对测试样本进行预测时，因为只用到训练样本中与其最接近的K个 样本，K近邻算法的偏置（Bias）往往很低，而方差（Variance）则 很高。当训练集较小的时候，K近邻算法易出现过拟合。

2.K-近邻模型
---------------------------------------------------------------------------

**KNN中，当训练集、距离度量、k值及分类决策规则确定后，对于任何一个新的输入实例，它所属的类唯一地确定**。这相当于根据上述要素将特征空间划分为一些子空间,确定子空间里的每个点所属的类。

*   _单元(cell)_：特征空间中对每个训练实例点x，距离该点比其他点更近的所有点组成的一个区域叫做单元。
    
*   _类标记( classlabel)_：每个训练实例点拥有一个单元，所有训练实例点的单元构成对特征空间的一个划分。❎最近邻法将实例  $\large x_i$ 的类 $\large y_i$ 作为其单元中所有点的类标记( classlabel)。这样,每个单元的实例点的类别是确定的。图3.1是二维特征空间划分的一个例子。
    

### 2.1 距离度量

 _**特征空间中两个实例点之间的距离是二者相似程度的反应**_，所以K近邻算法中一个重要的问题是计算样本之间的距离，以确定训练样本中哪些样本与测试样本更加接近。  
 在实际应用中，我们往往需要根据应用的场景和数据本身的特点来选择距离计算方法。当已有的距离方法不能满足实际应用需求时，还需要针对性地提出适合具体问题的距离度量方法。

#### $L_p$ 距离

![image-20210121165253838](E:/Development/Typora/images/27cf06751d4f58768a1432ad96636893-1676947286251-3.png)

当 $p = 2$ ，为欧氏距离（Euclidean Distance）

当 $p=1$，为曼哈顿距离（Manhattan Distance）

当 $p = ∞$ ，为各个坐标距离的最大值

下图为二维空间中，与原点的  $L_p$ 距离为1的点的图形（ $L_p = 1$ ）

![image-20210121171439008](E:/Development/Typora/images/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0-KNN%20image-20210121171439008-1676947286251-5.png) ![image-20210121182019340](E:/Development/Typora/images/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0-KNN%20image-20210121182019340-1676947286251-7.png)

#### 欧氏距离（Euclidean Distance）

欧氏距离是最常见的距离度量，**衡量的是多维空间中两个样本点之间的绝对距离**。

![image-20230221111653348](E:/Development/Typora/images/image-20230221111653348.png)

 可见，欧氏距离由两个样本之间每一维度之差的平方和计算而来。当维度之间的**取值范围差别太大时**，**欧氏距离容易被那些取值范围大的变量所主导**，从而会大大降低模型的效果。因此，在实际应用K近邻算法来解决分类等问题时，如果采用欧氏距离作为相似度度量，**最好提前对数据进行标准化转换**。

#### 曼哈顿距离（Manhattan Distance）

![image-20230221111752273](E:/Development/Typora/images/image-20230221111752273.png)

 曼哈顿距离等于两个样本之间每一维度之差的绝对值之和。曼哈顿距离的含义可以对应到规划为方框建筑的城市（如曼哈顿），两个地点的出租车最短行驶距离。**在使用曼哈顿距离时，也需要考虑变量之间取值范围不同对结果的影响**。

#### 其它

 当已有距离度量方法不能满足需求时，可以探索符合需求的距离度量方法。

 实际的数据中，往往是离散型变量和连续型变量同时存在，如何计算这种混合变量下的样本相似度是一个开放性的问题。一种简单的方法是，在进行距离计算之前对样本中的离散型变量进行One-Hot编码，然后选取上述介绍的距离计算方法进行处理。

### 2.2k 值的选择

#### 不同k值的影响

一般而言，从  $k = 1$开始，随着的逐渐增大，K近邻算法的分类效果会逐渐提升；在增到某个值后，随着的进一步增大，K近邻算法的分类效果会逐渐下降。

k值较小，相当于用较小的邻域中的训练实例进行预测，只有距离近的（相似的）起作用

*   单个样本影响大
    
*   “学习”的近似误差(approximation error)会减小，但估计误差(estimation error)会增大
    
*   噪声敏感
    
*   整体模型变得复杂，容易发生过拟合
    

k值较大，这时距离远的（不相似的）也会起作用

*   近似误差会增大，但估计误差会减小
    
*   整体的模型变得简单
    

#### 特例：$k=1$ （最近邻算法）

此时，KNN的泛化错误率上界为贝叶斯最优分类器错误率的两倍（证明见最后）

#### 特例：$k=N$ 

K近邻算法对每一个测试样本的预测结果将会变成一样（属于训练样例中最多的类）。

#### k值选择

*   一般k值较小。
*   k通常取奇数，避免产生相等占比的情况。
*   往往需要通过**交叉验证(Cross Validation)**等方法评估模型在不同取值下的性能，进而确定具体问题的K值。

### 2.3 分类决策规则

 一般都是多数表决规则(majority voting rule)，即把k个邻近的多数类别作为测试样本的类别

3\. 代码实现
---------------------------------------------------------------------------

### kNN的完整实现

1）确定K的大小和距离计算方法

2）从训练样本中得到K个与测试最相似的样本

 ①计算测试数据与各个训练数据之间的距离；  
 ②按照距离的递增关系进行排序；  
 ③选取距离最小的K个点；  
 ④确定前K个点所在类别的出现频率；  
 ⑤返回前K个点中出现频率最高的类别作为测试数据的预测分类。

3）根据K个组相似样本的类别，通过投票的方式来确定测试样本的类别

```python
import csv
import random
import math
import operator

# 加载数据集
def loadDataset(filename, split, trainingSet = [], testSet = []):
    with open(filename, 'r') as csvfile:
        lines = csv.reader(csvfile)
        dataset = list(lines)
        for x in range(len(dataset)-1):
            for y in range(4):
                dataset[x][y] = float(dataset[x][y])
            if random.random() < split:  #将数据集随机划分
                trainingSet.append(dataset[x])
            else:
                testSet.append(dataset[x])

# 计算点之间的距离，多维度的
def euclideanDistance(instance1, instance2, length):
    distance = 0
    for x in range(length):
        distance += pow((instance1[x]-instance2[x]), 2)
    return math.sqrt(distance)

# 获取k个邻居
def getNeighbors(trainingSet, testInstance, k):
    distances = []
    length = len(testInstance)-1
    for x in range(len(trainingSet)):
        dist = euclideanDistance(testInstance, trainingSet[x], length)
        distances.append((trainingSet[x], dist))   #获取到测试点到其他点的距离
    distances.sort(key=operator.itemgetter(1))    #对所有的距离进行排序
    neighbors = []
    for x in range(k):   #获取到距离最近的k个点
        neighbors.append(distances[x][0])
        return neighbors

# 得到这k个邻居的分类中最多的那一类
def getResponse(neighbors):
    classVotes = {}
    for x in range(len(neighbors)):
        response = neighbors[x][-1]
        if response in classVotes:
            classVotes[response] += 1
        else:
            classVotes[response] = 1
    sortedVotes = sorted(classVotes.items(), key=operator.itemgetter(1), reverse=True)
    return sortedVotes[0][0]   #获取到票数最多的类别

#计算预测的准确率
def getAccuracy(testSet, predictions):
    correct = 0
    for x in range(len(testSet)):
        if testSet[x][-1] == predictions[x]:
            correct += 1
    return (correct/float(len(testSet)))*100.0

def main():
    #prepare data
    trainingSet = []
    testSet = []
    split = 0.67
    loadDataset(r'irisdata.txt', split, trainingSet, testSet)
    print('Trainset: ' + repr(len(trainingSet)))
    print('Testset: ' + repr(len(testSet)))
    #generate predictions
    predictions = []
    k = 3
    for x in range(len(testSet)):
        # trainingsettrainingSet[x]
        neighbors = getNeighbors(trainingSet, testSet[x], k)
        result = getResponse(neighbors)
        predictions.append(result)
        print ('predicted=' + repr(result) + ', actual=' + repr(testSet[x][-1]))
    print('predictions: ' + repr(predictions))
    accuracy = getAccuracy(testSet, predictions)
    print('Accuracy: ' + repr(accuracy) + '%')

if __name__ == '__main__':
main()

```

4\. 实际应用
---------------------------------------------------------------------------

需要考虑的三个问题：

1.  通过何种方法寻找测试样本的“近邻”，即如何计算样本之间的距离或相似度？
2.  如何选择K值的大小才能达到最好的预测效果？
3.  当训练样本或者变量维度很大时，如何更快地进行预测？

> *   公众号 Python编程和深度学习[《机器学习系列（一）K-近邻算法》](https://mp.weixin.qq.com/s?__biz=MzUxNTY1MjMxNQ==&mid=2247484363&idx=1&sn=c6f5654bc5d530498c52a901e72e902f&chksm=f9b22c7fcec5a569982be25f259f3d0d135e2262c7ecb0c3f5f1e954603041b5eb8bc272c9d2&scene=178&cur_album_id=1338169891282845696#rd)
> *   《统计学习方法》第二版
> *   数据酷客
> *   [JYRoy博文《kNN算法：K最近邻(kNN，k-NearestNeighbor)分类算法》](https://www.cnblogs.com/jyroy/p/9427977.html)

 
