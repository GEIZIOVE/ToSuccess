# 李宏毅机器学习-36-GAN-01-Basic Idea

>   主要介绍生成式对抗网络(Generative Adversarial Network)的基本概念。Generator + Discriminator。

## Generative Adversarial Network

**生成式对抗网络**（Generative Adversarial Network，又称GAN，一般读作“干！”）计算机科学领域里是一项非常年轻的技术，2014年才由伊安·好伙伴教授（Ian Goodfellow，这姓氏实在是太有趣以至于印象深刻）系统地提出。但是一经提出，就引发了学术界对GAN如火如荼的研究，同时在最原始的GAN的基础上，针对不同的应用场景提出了许多GAN的变体。使用GAN网络，输入已知数据，计算机可以学习并创建全新的合成数据。Facebook AI部长Yann LeCun对GAN的评价是"Generative Adversarial Networks is the most interesting idea in the last the years in machine learning."GAN的迷人之处在于它不是一个传统的计算工具，通过机器学习，计算机可以更好地认识事物，而通过GAN，计算机可以去创造事物。

### 1.All Kinds of GAN

[![img](E:/Development/Typora/images/2021011218162211.png)](https://img-blog.csdnimg.cn/2021011218162211.png)

如上图所示，GAN的变种实在是太多了。下面给出一个例子和paper链接:

👉 [Mihaela Rosca, Balaji Lakshminarayanan, David Warde-Farley, Shakir Mohamed, “Variational Approaches for Auto-Encoding Generative Adversarial Networks”, arXiv, 2017](https://arxiv.org/abs/1706.04987v2)

本来要叫AEGAN，但是发现有人已经用掉了，所以只能叫α-GAN。



### 2.outline

本文章主要将从以下五个方面来进行介绍

>   1.  Basic ldea of GAN
>   2.  GAN as structured learning
>   3.  Can Generator learn by itself?
>   4.  Can Discriminator generate?
>   5.  A little bit theory



### 3.Basic Idea of GAN

#### 3.1原理

​		GAN是建立于神经网络的基础上的，其核心思想是“**生成**”与“**对抗**”，GAN网络结构包含两个模型，一个是`生成模型（Generator）`，另一个是`判别模型（Discriminator）`。**生成模型**一般是使用反卷积神经网络或者全连接神经网络，通过输入的数据生成二维图像，就像一个画家，在不断的学习和模仿中画出越来越像真品的作品。而**判别模型**是一个卷积神经网络二元分类器，它的工作主要是负责判断生成模型所生成的图像是来自数据集的真实图像，还是由人工伪造的假图像，就像是鉴赏家，鉴定由画家画出的作品是否是正品。也就是说，生成模型要尽力生成更加真实的结果来骗过判别模型，而判别模型要尽力区分真伪不被生成模型欺骗，二者之间形成一种相互对抗的关系，在反复的博弈训练中，画家的画功越来越娴熟，作品越来越以假乱真，而鉴赏家的眼光也越来越毒辣，识别能力越来越强，二者共同进步成长，直到最后，当判别模型分辨不出生成结果是否真实的时候（判别概率为0.5），该网络达到最理想的状态。 如何去训练这个模型呢？首先应该对判别模型先进行训练，因为必须先有一个好的判别器，使得能够比较好地区分出真实样本和生成样本之后，才好更为准确更新生成器的参数。当输入真实样本x时，$D(x)=1$，输入生成的样本$x’=G(z)$时，$D(x’)=0$。然后再将二者共同训练，判别器和生成器的的权重weight和偏差bias都是通过反向传播训练的。具体的训练算法使用stochastic gradient descent随机梯度下降法。 为了判别样本并归类，判别器采用了交叉熵（cross entropy）作为loss的计算方法来衡量相似性。

这里看不懂没关系，这里仅仅是做一个大概的描述，我们接下来会详细介绍。

#### 3.2Generation

GAN主要用于生成东西：生成图片，生成句子等。

1.  Image Generation

    ![img](E:/Development/Typora/images/20210112182433258.png)

    In a specific range，一般都是从一个分布（高斯）随机采样一个向量，根据这个向量生成图片。

    >   这里有一个挺有意思的生成动漫人物的小网站 👉 http://mattya.github.io/chainer-DCGAN/

2.  Sentence Generation

    ![img](E:/Development/Typora/images/2021011218252683.png)

    We will control what to generate latter. → Conditional Generation（这块前面有提到，后面有详细讲解）



#### Generator

正如我们前面所说的，GAN里面有两个重要的东西，其中一个就是`Generator`，Generator可以是一个NN。

[![img](E:/Development/Typora/images/20210112182852477.png)](https://img-blog.csdnimg.cn/20210112182852477.png)

它的输入是一个vector，它的输出是一个更高维的vector，也就是我们所需要产生的东西，在这里就是一张图片啦。

输入向量每一个维度都对应生成对象的某些特征。Each dimension of input vector represents some characteristics.例如：

[![img](E:/Development/Typora/images/20210112182926898.png)](https://img-blog.csdnimg.cn/20210112182926898.png)

看图中，左上角是我们原始的vector以及其对应所生成的对象。现在我们分别改变了三个维度的数值，生成的图像会有所不同：

-   右上角：改变第一个维度，即改变生成对象的头发长度
-   左下角：改变倒数第二个维度，即改变了生成对象的头发颜色
-   右下角：改变倒数第一个维度，即改变了生成对象的嘴巴的张开程度



#### Discriminator

在GAN中，有两个重要的东西，除了前面所提及的`Generator`，还有一个就是我们接下来要说的`Discriminator`。Discriminator也可以是一个NN。但注意它的输入和上面的Generator的不同。

[![img](E:/Development/Typora/images/20210112183718344.png)](https://img-blog.csdnimg.cn/20210112183718344.png)

Discriminator的输入是一张图片，输出是一个scalar，scalar用于判断这张图片的质量。

-   当这个输出值越高，则越像真实图片
-   当这个输出值越低，则越不像真实图片

我们看下面几个例子：

[![img](E:/Development/Typora/images/2021011218422211.png)](https://img-blog.csdnimg.cn/2021011218422211.png)

我们这里有四张二次元图片，其中上面两张是直接从动漫头像数据库中获取的，下面两张是通过Generator生成的。

-   上面两张图片通过Discriminator输出的scalar是很高的，表示是很好的二次元图片
-   下面两张图片通过Discriminator输出的scalar是很低的，表示是很垃圾的二次元图片



#### Generator and Discriminator(两者的关系)

我们先来看下面一张图

[![img](E:/Development/Typora/images/20210112184739368.png)](https://img-blog.csdnimg.cn/20210112184739368.png)

在生物进化的过程中，被捕食者会慢慢演化自己的特征，从而达到欺骗捕食者的目的，而捕食者也会根据情况调整自己对被捕食者的识别，共同进化，上图中的啵啵鸟和枯叶蝶就是这样的一种关系。生成器代表的是枯叶蝶，鉴别器代表的是啵啵鸟。它们的对抗思想与GAN类似，但GAN却有所不同。

GAN之所以有所不同，这里的原因是**GAN所作的工作与自然界的生物进化不同，它是已经知道最终鉴别的目标是什么样子，不知道假目标是什么样子，它会对生成器所产生的假目标做惩罚和对真目标进行奖励，这样鉴别器就知道什么目标是不好的假目标，什么目标是好的真目标，而生成器则是希望通过进化，产生比上一次更好的假目标，使鉴别器对自己的惩罚更小。以上是一个轮回，下一个轮回，鉴别器通过学习上一个轮回进化的假目标和真目标，再次进化对假目标的惩罚，而生成器不屈不挠，再次进化，直到以假乱真，与真目标一致，至此进化结束。**

[![img](E:/Development/Typora/images/20210112184857330.png)](https://img-blog.csdnimg.cn/20210112184857330.png)

假设让机器做二次元头像的生成，首先你要准备一个database，里面有很多真实的二次元人物的头像。

一开始你的生成器网络的参数是随机的，它自己不知道如何产生二次元人物头像。就先随便生成一些奇怪的东西。

然后去learn第一代的discriminator，方法是根据real image跟generator产生的image去调整里面的参数。即生成器产生一堆假的image，标记为0,；然后还有真正的image，都标记为1.然后用这些数据去learn一个神经网络判别分类器。

调生成器的参数得到新一代的生成器，让它生成的图片能够骗过前代的判别器。 如何调？ 将V1生成器和V1判别器组合在一起组合成一个大网络，在固定v1判别器部分网络参数的同时，梯度下降调整生成器模型参数，使输出为1.就得到了V2生成器。

以上图为例，我们最开始画人物头像只知道有一个头的大致形状，有眼睛有鼻子等等，但画得不精致，后来通过找老师学习，画得更好了，有模有样，直到，我们画得与专门画头像的老师一样好。这里的`我们`就像是`生成器`，一步步进化（对应生成器不同的等级），这里的`老师`就像是`鉴别器`（*这里只是比喻说明，现实世界的老师已经是一个成熟的鉴别器，不需要通过假样本进行学习，这里有那个意思就行*）

[![img](E:/Development/Typora/images/20210112185302337.png)](https://img-blog.csdnimg.cn/20210112185302337.png)

上图是一个形象的比喻，我们刚开始画人物头像的时候，只画了一个大概，并没有画眼睛，然后通过向老师学习，我们画出来的头像有了眼睛；但是此时我们的头像并没有涂颜色，于是我们继续向老师学习，画出来的头像有了颜色。…一直循环这个过程，直到我们画的头像和老师画的一样好。

这里我们就像是一个`生成器`，一步步的进化(对应生成器不同的等级)；老师就像是`鉴别器`(注意一点的是，这里仅仅是一个比喻，现实世界的老师已经是一个成熟的鉴别器，不需要通过假样本进行学习)

这里还引出了两个问题：

-   `Generator为什么不自己学，还需要Discriminator来指导？`
-   `Discriminator为什么不自己直接做？`

二者的关系可以描述为：写作敌人，念做朋友：

左边是棋魂，右边是火影。他们都是对立又互相帮助的典型，促使对方进步。

[![img](E:/Development/Typora/images/20210112185515972.png)](https://img-blog.csdnimg.cn/20210112185515972.png)

详细的我们下面会有所说明。

#### Algorithm(算法说明)

接下来要讲的是Generator和Discriminator是咋样训练出来的，Generator和Discriminator都是neuron network，而它们的架构是咋样模样，这取决于想要做的任务。举例来说：你要generator产生一张图片，那显然generator里面有很多的convolution layer。若要generator产生一篇文章或者句子，显然generator就要使用RNN来产生句子。

我们今天不讲generator跟discriminator的架构，这应该是取决于你想要做什么样的事情generator跟discriminator是某种network。

训练neuron network时要随机初始化generator和discriminator的参数。

[![img](E:/Development/Typora/images/20210112190753767.png)](https://img-blog.csdnimg.cn/20210112190753767.png)

在初始化参数以后，在每一个`training iteration`要做两件事：

[![img](E:/Development/Typora/images/20210112190859342.png)](https://img-blog.csdnimg.cn/20210112190859342.png)

第一件事情：固定generator，然后只训练discriminator（看到好的图片给高分，看到不好的图片给低分）。从一个资料库（都是二次元的图片）随机取样一些图片输入 discriminator，对于discriminatro来说这些都是好的图片。因为generator的参数都是随机给定的，所以给generator一些向量，输出一些根本不像二次元的图像，这些对于generator来说就是不好的图片。接下来就可以教discriminator若看到上面的图片就输出1，看到下面的图片就输出0。训练discriminatro的方式跟我们一般训练neuron network的方式是一样的。

有人会把discriminator当做回归问题来做，看到上面的图片输出1，看到下面的图片就输出0也可以。有人把discriminator当做分类的问题来做，把上面好的图片当做一类，把下面不好的图片当做另一类也可以。训练discriminator没有什么特别之处，跟我们训练neuron network或者binary classifier是一样的。唯一不同之处就是：假设我们训练binary classifier的方式来训练discriminator时，不一样的就是binary classifier其中类别的资料不是人为标记，而是机器自己生成。

[![img](E:/Development/Typora/images/20210112190732111.png)](https://img-blog.csdnimg.cn/20210112190732111.png)

第二件事情：固定discriminator，然后只训练generator。一般我们训练network是minimize 人为定义的loss function，在训练generator时，generator学习的对象不是人为定义而是discriminator。你可以认为discriminator就是定义了某一种loss function，等于机器自己学习的loss function。

generator学习的目标就是为了骗过discriminator，我们让generator产生一些图片，在将这些图片输入进discriminator，然后discrimination就会给出这些图片一些分数。接下来我们把generator和discriminator串在一起视为一个巨大的network。这个巨大的network输入 是一个向量，输出是一个分数，在这个network中间hidden layer的输出可以看做是一张图片。

我们训练的目标是让输出越大越好，训练时依然会做backpropagation，只是要固定住discriminator对应的维度，只是调整generator对应的维度。调整完generator后输出会发生改变，generator新的输出会让discriminator给出高的分数。

#### Algorithm(算法描述)

[![img](E:/Development/Typora/images/20210118204210506.png)](https://img-blog.csdnimg.cn/20210118204210506.png)

1.  首先初始化判别器和生成器

2.  然后从database中抽取m个图片（like batch size）；

    从一个分布中抽取m个vector

    使用m个vector产生m个image。 之后去调整判别器：

    首先把m张真实图片都拿出来，经过判别器得到分数，然后经过log再统统平均起来（当然希望这个越大越好，因为希望真实的图片得分高）；对于生成器生成的m张图片当然希望值越小越好，因此用1-值，其越大越好。因此使用梯度上升的方法，调节判别器参数。（实际训练过程是给真实图片赋值为1，生成图片赋值为0；**训练二分类器；等同于上述过程**）

3.  从一个分布中抽取m个vector

    重新生成m张图片，G（Z）就是一张图片，再把它丢到判别器中D（G(Z)）;再对所有的生成的求平均，在D不改变的情况下，希望这个值越大越好



#### 例子:Anime Face Generation

>   这里先给出文章链接 👉 [GAN学习指南：从原理入门到制作生成Demo](https://zhuanlan.zhihu.com/p/24767059)

100 updates

[![img](E:/Development/Typora/images/20210112201215808.png)](https://img-blog.csdnimg.cn/20210112201215808.png)

1000 updates

[![img](E:/Development/Typora/images/20210112201159209.png)](https://img-blog.csdnimg.cn/20210112201159209.png)

2000 updates

[![img](E:/Development/Typora/images/20210112201254694.png)](https://img-blog.csdnimg.cn/20210112201254694.png)

5000 updates

[![img](E:/Development/Typora/images/20210112201340987.png)](https://img-blog.csdnimg.cn/20210112201340987.png)

10,000 updates

[![img](E:/Development/Typora/images/20210112201400677.png)](https://img-blog.csdnimg.cn/20210112201400677.png)

20,000 updates

[![img](E:/Development/Typora/images/20210112201420859.png)](https://img-blog.csdnimg.cn/20210112201420859.png)

50,000 updates

[![img](E:/Development/Typora/images/20210112201441963.png)](https://img-blog.csdnimg.cn/20210112201441963.png)

另外一个结果（学生的作业，使用了自己收集的数据集）

[![img](E:/Development/Typora/images/20210112201512726.png)](https://img-blog.csdnimg.cn/20210112201512726.png)

人脸生成上的研究：

[![img](E:/Development/Typora/images/20210112201542190.png)](https://img-blog.csdnimg.cn/20210112201542190.png)

调整生成的向量，可以得到：

[![img](E:/Development/Typora/images/20200502143707303.png)](https://img-blog.csdnimg.cn/20200502143707303.png)

### 4.GAN as structured learning

#### structured learning and application(结构化学习和GAN的应用)

首先，我们要知道`结构化学习`（**Structured Learning**），GAN也是结构化学习的一种。与分类和回归类似，结构化学习也是需要找到一个�→�X→Y的映射。

[![img](E:/Development/Typora/images/20210112201906233.png)](https://img-blog.csdnimg.cn/20210112201906233.png)

但结构化学习的输入和输出多种多样，可以是序列（sequence）到序列，序列到矩阵（matrix），矩阵到图（graph），图到树（tree）等等。这样，GAN的应用就十分广泛了。例如，**机器翻译**（machine translation）,**语音识别**（speech recognition）以及**聊天机器人**（chat-bot）:

[![img](E:/Development/Typora/images/20210112202003990.png)](https://img-blog.csdnimg.cn/20210112202003990.png)

在图像方面，我们可以做**图像转图像**(image-to-image)，**彩色化**（colorization），还有**文本转图像**（text-to-image）

[![img](E:/Development/Typora/images/20210112202057223.png)](https://img-blog.csdnimg.cn/20210112202057223.png)

当然，GAN的应用远不止这么些，有非常有趣的变脸，图像自动打马赛克，自动生成多表情图像，年轻转年老等等，更多cool又`skr`的应用静待各位挖掘！

#### Structured Learning面临的挑战

One-shot/Zero-shot Learning：就是训练数据中可能没有或者只有很少的某个类别的数据，而要求模型生成对应的东西

>   -   In classification, each class has some examples.
>   -   In structured learning,
>   -   If you consider each possible output as a “class” ……
>   -   Since the output space is huge, most “classes” do not have any training data.
>   -   Machine has to create new stuff during testing.
>   -   Need more intelligence
>   -   Machine has to learn to do planning
>   -   Machine generates objects component-by-component, but it should have a big picture in its mind.
>   -   Because the output components have dependency, they should be considered globally.

在传统的分类问题中，每个分类都会有大量的样本。

在Structured Learning中，如果你将每个可能的输出都视为“类”，由于输出空间很大，`大多数“类”都没有任何训练数据`，也就是说大多数“类”我们在训练的时候并没有遇到过，这就导致了机器必须在testing期间创建新的东西。因此要求机器要更加的智能。

另外，`机器需要有大局观`。机器逐个组件地生成对象，但是它应该具有广阔的前景。由于输出组件具有依赖性，因此应全局考虑它们。

例如在图像生成的时候，每次生成一个像素，但是所有像素合起来要像一个人脸。不能人脸有三个眼睛，两个嘴巴，因此在Structured Learning中生成不是最重要的，重要的是component与component之间的关系。

[![img](E:/Development/Typora/images/20210112203014543.png)](https://img-blog.csdnimg.cn/20210112203014543.png)

-   生成数字，先生成一个点，这个时候并不能说这个点生成得好或者不好
-   生成文本，要看整体效果，只看部分是不够的

那么如何把GAN 和 structured learning联系起来？

#### Structured Learning Approach

[![img](E:/Development/Typora/images/2021011220315174.png)](https://img-blog.csdnimg.cn/2021011220315174.png)

我们可以把Generator看做Structured Learning中生成很小的component的部分

Discriminator看做整体的评估方法。

二者合起来，就是GAN.



### 5.Can Generator learn by itself?(生成器能否自己学习)

先来看下，如果不用GAN，怎么来训练Generator

#### Generator

之前说过，Generator就是一个NN，输入一个向量，生成一个图片，如果现在是做生成手写数字，那么就是输入一个向量，得到一个手写数字。输入不同的向量，得到不同的数字。
如果我们用不同的向量来代表不同的手写数字：

[![img](E:/Development/Typora/images/20210112213624385.png)](https://img-blog.csdnimg.cn/20210112213624385.png)

如何训练这个生成器呢？可以用监督学习的方法来进行训练：

[![img](E:/Development/Typora/images/20210112213655356.png)](https://img-blog.csdnimg.cn/20210112213655356.png)

在传统的监督学习中，给NN一个输入和输出的pairs，就这样一直train下去就好。比如有一系列的图片，随便给他一个code，然后训练即可，使输出和label越接近越好。这就和我们训练分类器的过程一样。

现在的问题是表示图片的code从哪里来？随机？不行啊，因为`当输出类似的时候，输入由于是随机的，有很大差别，这样很难训练出一个网络`。例如：

[![img](E:/Development/Typora/images/20200502153250549.png)](https://img-blog.csdnimg.cn/20200502153250549.png)

这里的两个code如果差很远，你叫机器如何学习？可以看到这里的第一维0.1可能表示数字1，然后第二维表示的是倾斜角度。

**如何得到与实际数据有关的输入vector呢？**

之前学过Encoder，可以把图片压缩为一个向量。你可以训练一个encoder，给encorder一个图片，它将输出一个相应的code。

[![img](E:/Development/Typora/images/20210112214114963.png)](https://img-blog.csdnimg.cn/20210112214114963.png)

#### Auto-encoder

我们先来回顾一下auto-encoder：

[![img](E:/Development/Typora/images/20210112214455620.png)](https://img-blog.csdnimg.cn/20210112214455620.png)

分析上面的结构，发现Auto-encoder的Decoder就是根据c（向量）生成图片，这不就是和Generator干的事情是一样的吗？

[![img](E:/Development/Typora/images/2021011221460119.png)](https://img-blog.csdnimg.cn/2021011221460119.png)

我们看能不能把这块单独拿出来，Randomly generate a vector as code，然后生成图片。

通过随机生成的二维向量，然后生成手写数字图片。

[![img](E:/Development/Typora/images/20210112214721323.png)](https://img-blog.csdnimg.cn/20210112214721323.png)

假设给定下图中的两个二维向量得到的结果分别是手写数字0和1

[![img](E:/Development/Typora/images/20200502154054507.png)](https://img-blog.csdnimg.cn/20200502154054507.png)

那么，随着向量在坐标轴上的变换，生成图片如下图所示：

[![img](E:/Development/Typora/images/20200502154144687.png)](https://img-blog.csdnimg.cn/20200502154144687.png)

通过上面的实验，我们可以下如下结论：
如果有两个向量a和b，那么我们可以分别生成两个手写数字：

[![img](E:/Development/Typora/images/20200502154401998.png)](https://img-blog.csdnimg.cn/20200502154401998.png)

但是存在一个问题：你的training data里面的image是有限的，当生成器在看到a的情况下生成偏左的1，在看到b的情况下也生成偏右1，当a和b 的平均，会产生什么样的图片呢？事实是由于网络是非线性的，得到的不一定是正的1，也可能是噪声。

[![img](E:/Development/Typora/images/20200502154417727.png)](https://img-blog.csdnimg.cn/20200502154417727.png)

可以使用VAE来解决！

#### Variational Auto-Encoder(VAE,变分编码器)

为了解决上面的问题，因此我们有了VAE；VAE不仅产生一个code（m1，m2，m3）还会产生每一个维度的方差；然后将方差和正态分布中抽取的噪声进行相乘，之后加上code上去，相当于加上noise的code；之后输入的decoder中就得到图片；这样情况下，decoder不只是看到a或b产生一些数字，当看到a或b加上一些noise也要产生数字；这样使decoder更加鲁邦。

[![img](E:/Development/Typora/images/20200502154528671.png)](https://img-blog.csdnimg.cn/20200502154528671.png)

#### VAE vs GAN（VAE方法的缺陷）

但是这种VAE和decoder的生成器少了什么东西？

生成器在train的时候，我们希望的是最终生成器生成的图片和我们给定的图片在像素级别上越像越好。但是肯定会产生一些mistake。

[![img](E:/Development/Typora/images/20210112215652945.png)](https://img-blog.csdnimg.cn/20210112215652945.png)

这个差距往往是通过两个图片逐像素进行比较得到的，例如上图中的箭头就是两个图片对应的像素，我们比较这些所有的像素，计算差距，换句话说就是用两个图片的向量，计算每个维度之间的距离来衡量差距。

当然如果生成的图片如果和真实图片一模一样当然没有问题，现在问题是生成的图片总是会和真实图片有一些差异，在计算差异的时候就会出现问题：

[![img](E:/Development/Typora/images/2021011221581642.png)](https://img-blog.csdnimg.cn/2021011221581642.png)

我们的目标是要生成右上角的图片，但是实际上肯定会有一些mistake，

可以看到，虽然上面两个按差异计算来说比下面的图片的计算结果要好，但是从图片整体来看，反而下面的图片符合手写图片的规律。

也就是说模型的优化目标**不是单纯的让你的生成结果与真实结果越接近越好**。而是要**使得component与component之间的关系符合现实规律**。例如：

[![img](E:/Development/Typora/images/20210112220142669.png)](https://img-blog.csdnimg.cn/20210112220142669.png)

左边多了一个像素是不行的，如果在多出来的像素附近填满像素当然是可以的。但是这个事情在NN中很难做到。

看我们最右边的图，第二个神经元输出有颜色的时候，那么它会希望旁边的神经元也生成颜色，这样符合手写数字的规律，但是NN中，**每个神经元都是独立的！！！**

我们需要structure 来约束这个事情。就是在后面在加几个隐藏层，就可以调整第L层的神经元输出。

也就是说理论上，**VAE要想获得GAN的效果，它的网络要比GAN要深才行**。

而且**由于VAE算法采用的分布采样，因此做一些离得比较散的目标效果不好**：

下图中绿色是目标，蓝色是VAE学习的结果

[![img](E:/Development/Typora/images/2020050216220873.png)](https://img-blog.csdnimg.cn/2020050216220873.png)

### 6.Can Discriminator generate?(判别器能否自己生成图片)

#### Discriminator

鉴别器是给定一个输入，输出一个[0,1]的置信度，越接近1则置信越高，越接近0则置信度越低，如图所示：

[![img](E:/Development/Typora/images/20210112220743239.png)](https://img-blog.csdnimg.cn/20210112220743239.png)

>   Discriminator 在不同文献中有不同的名字：Evaluation function, Potential Function, Energy Function …

对于单纯的生成器，想要去判别不同像素之间的关联是很困难的；但鉴别器的优势在于它可以很轻易地捕捉到元素之间的相关性，例如自编码器中出现的像素问题就不会在鉴别器中出现，因为你是把输出好的图片丢给判别器的，然后去评价它好不好。比方说有一个滤波器，他会去检索有没有独立的像素点，有的话就是低分。

[![img](E:/Development/Typora/images/20210112221009979.png)](https://img-blog.csdnimg.cn/20210112221009979.png)

假如说已经有了一个判别器，它能够鉴别一个图片是好的还是不好的，我们就可以用这个判别器去生成图片。穷举所有的输入x，看看判别器给它的分数，找到最大的就是判别器生成的（如何找，别管他），结束。所以很麻烦。

[![img](E:/Development/Typora/images/20210112221318547.png)](https://img-blog.csdnimg.cn/20210112221318547.png)

#### Discriminator - Training

现在我们手上只有真实的对象，因此仅仅用这个东西来训练，得到的Discriminator会比较笨，看什么都是真实的。

[![img](E:/Development/Typora/images/2021011222150487.png)](https://img-blog.csdnimg.cn/2021011222150487.png)

因此我们需要一些负样本来进行训练。

一个办法是直接在真实对象上加noise，让它成为负样本(图：左二)：

[![img](E:/Development/Typora/images/20210112221819342.png)](https://img-blog.csdnimg.cn/20210112221819342.png)

但是这样就会出现一个问题，一旦出现一张稍微有点像动漫头像的picutre就会得到很高的分数(图：左三)。

我们想要的判别器是，如果两只眼睛不一样，就能判断出这不是一张人脸的判别器(图：右二)。但是怎样产生这些好的负样本呢？

我们需要好的负样本才能训练处一个判别器，我们需要一个好的判别器才能找出好的负样本

需要一个迭代的算法

1.  开始的时候你有一堆正样本和负样本：正样本是真实图片，负样本是加了噪声后的图片

2.  开始迭代

    1). 你的判别器需要做的就是去给正样本高的分数，负样本低的分数

    2). 当你学出一个判别器后，再用判别器去做生成（就是一个argmax的过程，具体怎么做，不知道呀），生成一堆它觉得好的的图片，之后作为负样本重复 1）过程

下面我们看一下算法和算法的一个例子：

[![img](E:/Development/Typora/images/20210112222417556.png)](https://img-blog.csdnimg.cn/20210112222417556.png)

上图中我们已经通过了算法中的第一次迭代，通过计算��� ����(�)arg maxD(x)获得了Discriminator认为好的图片；

然后将这些图片作为负样本，和正样本一起进行训练Discriminator，即给正样本高的分数，负样本低的分数；看下图：

[![img](E:/Development/Typora/images/20210112222725532.png)](https://img-blog.csdnimg.cn/20210112222725532.png)

再一次通过计算��� ����(�)arg maxD(x)获得了Discriminator认为好的图片，完成第二次迭代；

[![img](E:/Development/Typora/images/20210112222942400.png)](https://img-blog.csdnimg.cn/20210112222942400.png)

上面的过程不断反复…

接下来我们从可视化和概率的角度来看一下整个过程

[![img](E:/Development/Typora/images/20210112223149785.png)](https://img-blog.csdnimg.cn/20210112223149785.png)

输入一个对象x（例如图像），输出一个向量，用于评判x到底好还是不好，好就是真实对象，不好就是生成对象。

为了简单，我们把输出D(x)看成一维的。那么红色的分布曲线就是D(x)。这里我们希望对于真实对象（绿色）的得分集中在凸起部分，生成对象在其他部分。

但是由于D(x)一般是高维的，那么很难做到某个地方就是这么简单按取值来判定，高维空间的分布非常复杂，不能把没有真实对象出现的地方就弄成低分。

事实上去学D（x）的过程是一个迭代的过程：

-   加入蓝色的是判别器生成图片的分布，绿色的是真实图片分布，我们需要做的是让D（x）给绿色的点高分，蓝色的点低分；（如果只训练一次的话，可能有的地方的分数比real分数还高）
-   之后去寻找找出判别器的弱点（得分最大的地方让它变成负样本）
-   如何寻找。之后让判别器去生成图片（也就是会给高分的图片，其实就是生成器生的），然后让判别器让这些地方变成低分。
-   反复迭代。最终正样本和负样本就会重合在一起。

最终生成器只会生成真实图片分布中的picture，然后判别器最终会判断不出好坏…

[![img](E:/Development/Typora/images/20210112223543916.png)](https://img-blog.csdnimg.cn/20210112223543916.png)

其实用discriminator来生成对象就是GAN中的一种生成方法，记得在RL中里面有提到过，critical policy的生成方法。

最后贴一张图（Structured Learning的分支），图中的Graphical Model实际上就是相当于discriminator，整个训练也是迭代方式进行的

[![img](E:/Development/Typora/images/20210112223826773.png)](https://img-blog.csdnimg.cn/20210112223826773.png)

### 7.Generator v.s. Discriminator(单纯的生成器和单纯的判别器的对比)

[![img](E:/Development/Typora/images/20210112223903382.png)](https://img-blog.csdnimg.cn/20210112223903382.png)

生成器和判别器的对比：

-   单纯的生成器：很容易生成图片，因为就是一个前向输出过程。但是它不考虑像素之间的联系（只学到表象，没学到精神或全局信息）
-   单纯的判别器：可以学习考虑大局，但是很难去生成图片（要去**解一个argmax的问题**，要做一些线性假设，线性则限制了它的能力，非线性无法解）

### 8.Generator + Discriminator

GAN就是取代了这个argmax的过程；现在我们用生成器去得到好的负样本来取代argmax D（x）；因为生成器在train的时候就是去学一些image，是可以躲过判别器的（给高分的）。

[![img](E:/Development/Typora/images/20210112224511267.png)](https://img-blog.csdnimg.cn/20210112224511267.png)

从上面可以看出生成器和鉴别器的优缺点是可以互补的，这也就是GAN的优势。（**生成器+鉴别器**）

[![img](E:/Development/Typora/images/20210112224650925.png)](https://img-blog.csdnimg.cn/20210112224650925.png)

-   从生成器角度出发，虽然在生成图片过程中的像素之间依然没有联系，但是它的图片好坏是由有大局观的判别器来判断的。从而能够学到有大局观的生成器。
-   从鉴别器的角度出发，利用生成器去生成样本，去求解argmax问题

当然，GAN也是又缺点的，它是一种隐变量模型，可解释没有生成器和鉴别器强，另外GAN是不好进行训练。我在训练DAGAN的时候就成功造成了鉴别器的误差为0，无法进行反向传播更新梯度。



### 9.GAN vs VAE

GAN与VAE的生成对比:

VAE(蓝色是generator生成的结果，绿色是真实对象):

[![img](E:/Development/Typora/images/2020050216220873.png)](https://img-blog.csdnimg.cn/2020050216220873.png)

GAN(红色是generator生成的结果，蓝色是真实对象):

[![img](E:/Development/Typora/images/20210112225022913.png)](https://img-blog.csdnimg.cn/20210112225022913.png)

VAE生成的人脸比较模糊，GAN的比较清晰。

[![img](E:/Development/Typora/images/20200503114748872.png)](https://img-blog.csdnimg.cn/20200503114748872.png)

某篇论文中不同GAN和VAE的对比。从图中我们看到，实验数据集分别是MNIST和CIFAR10，然后评价指标是FID，这个指标具体后面还会讲，这里只要知道越小越好，结果显示所有的GAN其实都差不多，都有很大的范围，因为GAN的算法对于参数非常敏感，很容易产生很大的偏差，但是总体而言，GAN对比VAE，VAE的variance比较小，GAN比VAE的效果（应该是看星号位置）要好。

[![img](E:/Development/Typora/images/20200503114958383.png)](https://img-blog.csdnimg.cn/20200503114958383.png)

### 10.A little bit theory

这块内容有点多，单独拿出来了，链接：

👉 [机器学习-35-Theory behind GAN(GAN背后的数学理论)](https://blog.csdn.net/qq_44766883/article/details/112614962)