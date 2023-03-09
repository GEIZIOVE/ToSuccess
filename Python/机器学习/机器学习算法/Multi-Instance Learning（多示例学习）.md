## **Multi-Instance Learning（多示例学习）综述**

```
涉及到专业词汇的，我这里不强行翻译成中文，要不然更加难懂。
```

## **1.什么是multi-instance learning？**

### **1.1 定义**

[multi-instance learning](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Multiple_instance_learning)

​		 MIL的数据集的数据的单位是bag，以二分类为例，一个bag中包含多个instance，==如果**所有**的instance都被标记为negative，那么这个包就是negative，反之这个包为positive。==

​		设Y为包X的label，$X = {x_{1},x_{2},...,x_{n}}$，每个示例$x_i$对应一个标签$y_i$，则包的标签可以表示为：

![image-20230221115226229](E:/Development/Typora/images/image-20230221115226229.png)



### **tasks:**

1. **Classification**
2. **Regression**
3. **Ranking**
4. **Clustering** 以上学习目标都可以分为bag维度的和instance维度的。

## **1.2 example**

 我刚接触到MIL这个概念的时候还是有点懵的，这里贴一个由Babenko (2008)提出的非常简单的例子。

> 设想有若干个人，每个人手上有一个钥匙串(bag)，串有若干个钥匙(instance)。已知某个钥匙串能否打开特定的一扇门(training set)。**我们的任务是要学习到哪一串钥匙串能打开这扇门，以及哪个钥匙能打开这扇门。**

## **1.3 起源**

### **1.3.1 Background**

​		MIL最早由Dietterich et al. [2]在Drug Activity Prediction中提出。大多数药物是由一些小分子构成，这些小分子可以bind在大型蛋白质上。对于能够制作成药物的分子，它的其中一种low-engergy shape可以紧密地bind在目标位置上。一个分子可以有多个low-engergy shape，但是在当时学者们只能判断一个分子能否制成药物，而不能判断到底是分子的哪个low-energy shape在起作用。假如我们用常见的分类算法，把所有能制药的分子的low-energy shape当作正例，反之当作负例。那么我们的训练结果会非常不准确，因为我们有太多的false positive。一个能制药的分子可能有上百个low-engergy shape，但是真正起作用的可能只是其中一个low-engergy shape。

### **1.3.2 Representation**

 基于此，Dietterich et al. [2]提出了MIL的概念。他把一个low-engergy shape当作一个instance，一个分子所有的low-engergy shape构成一个bag。能制药的分子自然是positive bag，反之则为negative bag。通过从同一位置射出的方向不同的162束射线，构造出分子的表面，所产生的162个参数以及分子的其他参数共同构成特征空间。

### **1.3.3 APR Algorithm**

 Dietterich提出了三种APR（*axis-parallel rectangle*）算法，都还挺有趣的，这里简单介绍 GFS elim-count APR算法思想。如图三所示，白色表示正例，黑色表示负例。首先在特征空间中找到一个包含所有positive bag的APR。接下来用贪心算法的思想将APR不断缩小到仅包含positive bag，就像图中虚线表示的APR一样。对于每个negative bag，图中都标识出了为了将其排除在APR外需要付出的**代价**（需要附带排除的最少的positive bag数量）。GFS elim-count APR算法每次选择最小的代价，将APR的边界不断缩小，得到最终的结果。GFS kde APR算法在GFS elim-count APR的基础上，还将APR中某个instance的数目也考虑到**代价**中，显然，某个instance在APR中的数目越少，排除它所需要付出的代价就越大。

![img](E:/Development/Typora/images/v2-62a611f34a45d0534e2e18d86fa1a1a0_720w.jpeg)

## **1.4 Learning Algorithm**

 周志华老师在[1]中提到了5种算法，Diverse Density, Citation-kNN, ID3-MI,RIPPER-MI, BP-MIP， 本质上分别对应贝叶斯分类器，KNN，决策树，规则归纳算法以及神经网络，这里就不展开了。

## **2 MIL的特点**

![img](E:/Development/Typora/images/v2-8cc894e479585324c1a0d6244c9f0439_720w.webp)

MIL主要有如上图所示的四个特点，分别是task, bag composition, data distributions and label ambiguity。[3]

## **2.1 prediction level**

![img](E:/Development/Typora/images/v2-7dff0d27019136834a9780c8a5cb356f_720w.webp)

预测等级可以分为包预测和示例预测。如上图所示，对包预测而言，红色虚线和紫色虚线都是最优解。但是对示例预测，只有紫色虚线才是最优解。因此，包预测任务的表现基本上代表不了示例预测的表现。

两种预测级别最大的区别在于对示例预测错误所付出的代价。包分类的情况下，如果一个包中发现witness，那么它被划分为postive，而包中其他示例的label可以被忽略。因此，此时FP和FN对包分类的准确度没有影响，但是会增加示例分类错误率。而当我们考虑负包时，一个FP会让一个包被错误分类（*只要负包中有一个positive label instance，这个包会被划分为正包*）。设想所有负包中1%的示例分类错误，此时负包分类准确率为0%，但是负示例分类准确率却有99%。基于此，有的研究者会针对正包和负包设计不同的loss function。

## **2.2 Bag Composition**

### **2.2.1 witness rate(WR)**

��=# �������� ���������# ����� ���������

 当WR很高时，正包里基本上只包含正示例，此时示例分类等价于包分类，就可以用常规的监督学习算法来训练。WR很低时，会影响到许多算法的表现。许多学者提出了针对低WR情况下的优化，比如在miGraph[4]中，类似的instance被划到一个组（clique）中，每个instance的重要性和其所属的组的大小成反比。

### **2.2.2 Relations Between Instances**

 目前大多数MIL算法都假设（often not explicitly），正示例和负示例分别独立地从正示例分布和负示例分布中采样，然而这并不符合现实世界的情况。在许多应用中，由于包和示例之间的结构和相互关系，IID假设往往不成立。[3]中阐述了以下三种不同的关系。

### **Intra-Bag Similarities**

 在有些问题中，同一个包中的示例之间有相似性，并且与其他包中的示例之间没有。比如图中的图像块（instance），它们的部分像素是完全相同的，因此它们之间的相关性很高。Intra-Bag Similarities给学习带来了很多问题。目前解决了这个问题的算法很少。miGrapa[4]对每个包建立了一张图，相似的示例被划为到一个组中，从而通过组大小一起调整它们的相对重要性。在CCE[5]中，每个instance都被划分到一个cluster中，一个bag被表示为一个二元向量（每个值表示是否包含某个cluster）。相似的instance在同一个cluster中，因此具备对intra-bag similarity的鲁棒性。

![img](E:/Development/Typora/images/v2-15ccb720cb365e2af81d8e8101e237fe_720w.webp)

### **Instance Co-occurrence**

 有些具有语义相关性的instance会同时出现在一个包内。比如图中熊，更有可能出现在野外(nature)而不是夜总会（误）中。因此，对nautre segmentation的观测，对熊的识别是有帮助的。有许多算法利用了instance co-occurence，这里不展开了。

 instance co-occurrence 在包分类中很有用，但是在示例分类中，却很可能给算法带来误导。如果一个正示例经常和一个负示例同时出现，那么算法会极大可能将负示例分类为正示例，导致FPR(false positive rate)升高。

![img](E:/Development/Typora/images/v2-c40dfe12549d05570b59c996b29e41a5_720w.webp)

### **Instance and Bag Structure**

 在有些问题中，示例之间甚至包之间存在结构关系。结构比一般的co-occurrence要复杂一些，因为它们可能是有顺序的，或者有更高级意义的联系。能捕捉到这种结构，可以提高分类的表现。这些结构有可能是空间上的，时间上的，关系的，也有可能是随机的。比如说，如果一段视频作为一个包，所有的视频帧是有时间顺序的。在web数据挖掘中，网页是包，链接是示例，两个互相链接的网站之间是存在语义关系的。图模型可以很好得捕捉实体之间的关系，GNN最近也是挺火的研究热点。

## **2.3 Data Distributions**

 机器学习算法一般都对数据的分布作了隐式假设，比如1.3.3中的APR算法。

### **Multimodal Distributions of Positive Instances**

 APR算法假设正例位于某一个聚簇或区域内，一些早期的MIL算法也作出了相同的假设。这种假设在大多数情况下是不成立的。以图像分类为例，目标有可能跟许多聚簇是相关的。比如下图中，蚂蚁外表可以是黄色，红色或者是黑色，它们的翅膀跟身体都可以有不同的形状。显然在某一个位置上，包含以上所有这些变量特点是不太可能的。

 有许多可以进行多模态正例概念的MIL算法，[3]中提到了如下几种**非参数法**：

 Citation-kNN and MInD：这两种方法都基于对包与包之间距离的计算，因此适用于不同类型的分布。

 DD-SVM and MILES：用一些原型来作为包的表示，不同类型的正聚簇可以被表示为不同的原型。

 mi-SVM：示例级的SVM算法。用一种kernel来处理不连通的正示例区域。

![img](E:/Development/Typora/images/v2-ccee57b43c5cee52786d048b6ba6bca1_720w.webp)

### **Non-Representative Negative Distribution**

 总的来说，负分布在MIL一般不怎么被关注。很多方法是直接计算某个包到某个点或区域的距离，如果这个距离非常大，这个包就被认为是负包。

## **2.4 Label Ambiguity**

### **Label Noise**

 有些MIL算法（比如APR和DD），特别是依循标准的MIL假设的算法，**严重**依赖于标签的正确性。

**解决方法：**

 1. 符合**collective assumption**的算法。负包里的正示例影响较小，因为算法不只靠正示例来进行分类。比如：1.将包表示为分布的算法[6]，一个false positive不会显著的改变分布。2.包由所有示例平均表示，NSK-kernel。

 2. 统计包中正示例的个数，获取对正示例分类的阈值，减弱负包中少量正示例的影响。

### **Different Label Spaces**

 MIL问题中，可能存在instance跟bag的label space是不同的。比如下图中的例子，我们的目标是检测斑马，但是右边几个图片中的patches也可能落入到斑马的region中。这些instance都不能被很明确地打上标签。

**解决方法：**

 1.符合**collective assumption**。比如Vocabulary-based methods[6]，[6]将instance和words（比如prototypes or clusters）联系起来，包的表示建立在这些words的基础上。

 2. 利用基于到原型instance的距离。比如MILES和MILIS。

![img](E:/Development/Typora/images/v2-3486056b9ffff1b7509dc2b841417515_720w.webp)

## **3 在cv里的应用**

 使用MIL，我们可以利用爬虫技术所爬到的图片自带的tags进行训练，而不是针对每个部分进行标注的图片。[3]中介绍了MIL在生物，化学，网络挖掘等方面的应用，本文只介绍跟CV有关的。

**MIL为什么有用**

 包的类别不能由单独某个instance学习到，但是可以通过多个instance相互作用学习到。[3]比如说要检测到沙滩，必须要同时检测到沙和水。

### **Content Based Image Retrieval（图像检索 CBIR）**

 CBIR的目标是根据图片中所包含的内容对它们进行分类，并不关心目标的具体位置，因此这是一个典型的包分类问题。图片被切分成patches，对应着instance，整个图对应着bag。分割图片的方法有很多种[3]，比如按网格分割，按关键点分割，也可以按语义区域来分割。一个包中有可能包含很多相似的示例，尤其是在高密度网格分割中。按语义分割的话这种情况稍微少一点，因为一个语义区域通常只有一个目标。

**总结：**

1. 能利用到目标的co-occurrence现象（比如天空和鸟），算法会有更好的表现。
2. 由于一个包的主题可能是多种概念的组合，因此在collective MIL assumption假设下工作的算法效果更好。
3. 相同的物体在不同的角度下可能有不同的形状。而且许多物体有不同的形状或者颜色。因此单峰分布不太能表示整个类。

### **Object Localization and Segmentation**

 目标定位和分割是从包中学习对示例的分类。比如说，从弱标签数据集训练一个识别系统。弱标签数据集是指，对单张图片来说，只标记这张图片里有哪些物体，而不标记这些物体对应的bounding box或者是像素级的标记。这种强标记需要大量的人工。笔者曾尝试用labelme做过标记，单张图片耗时20分钟。

> 但是这些弱数据从哪来呢？

1. 描述性的句子。
2. 搜索引擎的提供的结果。
3. 相似图片的标记
4. 网页上和图片相联系的单词。

示例分类在视频中也有应用，曾被用来识别视频中的一些复杂事件，如“生日派对”等。为了解决这个问题，一段视频被分成了若干段，这些片段被分别进行分类。这种方法也曾被用来检测一些不适合儿童观看的视频片段。也有学者将MIL应用到视频目标追踪中，一个分类器被训练用于识别并追踪某个物体。这个分类器提出一些候选框，这些候选框构成一个包，被用来训练MIL分类器。

### **Computer Aided Diagnosis and Detection**

 弱标签（比如说，对某个ct片的整体诊断，而不是具体哪一块区域有问题）在医学诊断中更常见。一个X光图或ct图中，所有区域都是正常的，那么该bag就是健康的，只要有一个区域是不正常的，那么该bag就是不健康的。非常符合MIL框架的定义。

 在进行包分类的医学图像处理任务中，MIL分类器利用到了co-occurrence和instance structure的信息，比一般的监督学习表现得要好一些。

 示例级的任务在医学图像处理中不太常见，因为示例级的标签很难获取（没有ground truth）。这也是比较矛盾的一点，使用MIL框架的motivation本来就是因为缺少示例级的标签。但是这里也能作为日后的研究方向吧，这其实也是整个医学图像处理中比较坑的一点，有能力去做标注的人本来就很少。

 总之CAD方向的挑战还是很多的，真正实际应用时受到很多方面的影响，比如病人的年龄，性别，体重等等。而且疾病一般来说是分为不同阶段的，但是有些算法[3]把某种疾病的全部阶段都作为同一个positive class。

## **4.总结**

 说了这么多，我个人觉得MIL这种框架还是有搞头的（好编故事），比较符合现实世界的情况（weak labels）。这几年MIL也算是一个研究热点，还是有很多研（guan）究（shui）方向可以挖掘的。

## **Reference**

1. Z.-H. Zhou. [Multi-instance learning: A survey.](https://link.zhihu.com/?target=https%3A//cs.nju.edu.cn/zhouzh/zhouzh.files/publication/techrep04.pdf) Technical Report, AI Lab, Department of Computer Science & Technology, Nanjing University, Nanjing, China, Mar. 2004.
2. T.G. Dietterich, R.H. Lathrop, and T. Lozano-P´erez. Solving the multiple-instance problem with axis-parallel rectangles. Artiﬁcial Intelligence, vol.89, no.1–2, pp.31–71, 1997.
3. Carbonneau M A, Cheplygina V, Granger E, et al. Multiple instance learning: A survey of problem characteristics and applications[J]. Pattern Recognition, 2018, 77: 329-353.
4. Z.-H. Zhou, Y.-Y. Sun, and Y.-F. Li, “Multi-Instance Learning by Treating Instances As nonI.I.D. Samples,” in ICML, 2009.
5. Z.-H. Zhou and M.-L. Zhang, “Solving Multi-instance Problems with Classiﬁer Ensemble Based on Constructive Clustering,” Knowl. Inf. Syst., vol. 11, no. 2, pp. 155–170, 2007.
6. J. Amores, “Vocabulary-Based Approaches for Multiple-Instance Data: A Comparative Study,” in ICPR, 2010.