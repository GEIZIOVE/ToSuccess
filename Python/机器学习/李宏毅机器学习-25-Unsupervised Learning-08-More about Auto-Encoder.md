# 李宏毅机器学习-25-Unsupervised Learning-08-More about Auto-Encoder

>   本文主要就Auto-Encoder的内容进行进一步的讨论。主要介绍了Feature Disentangle (特征解耦)和Discrete Representation(离散表示)等内容。

## More about Auto-Encoder

还是**Auto-Encoder**，已经很熟悉那个模型类吧

[![img](E:/Development/Typora/images/20210125202611638.png)](https://img-blog.csdnimg.cn/20210125202611638.png)

​		Auto-encoder是一个基本的**生成模型**，更重要的是它提供了一种**encoder-decoder**的框架思想，广泛的应用在了许多模型架构中。简单来说，Auto-encoder可以看作是如下的结构，它主要包含一个**编码器**（Encoder）和一个**解码器**（Decoder），通常它们使用的都是**神经网络**。

​		Encoder接收一张图像（或是其他类型的数据，这里以图像为例）**输出一个vector**，它也可称为`Embedding`、`Latent Representation`或`Latent code`，不管它叫什么，我们只需要知道它是关于输入图像的表示；然后将vector输入到Decoder中就可以得到重建后的图像，希望它和输入图像越接近越好，即最小化重建误差（reconstruction error），误差项通常使用的**平方误差**。

那么今天主要讲三件事：

-   **More than minimizing reconstruction error**
-   **Feature Disentangle**
-   **More interpretable embedding**



### 1.What is good embedding?(好的embedding是怎样的？)

[![img](E:/Development/Typora/images/20210125202841869.png)](https://img-blog.csdnimg.cn/20210125202841869.png)

​		之前要做embedding的根本原因是：An embedding should represent the object.虽然我们希望Decoder输出的重建图像和输入到Encoder中的图像越接近越好，**但是通常我们并不关注重建后的图像是什么样的，更多的希望得到一个关于输入图像有意义、解释性强的embedding**。

​		如何评价一个embedding是否是好的呢？最直观的想法是它应该包含了关于输入的**关键信息**，从中我们就可以大致知道输入是什么样的。或是从流形学习的角度来看，希望它可以学到关于**高维输入数据的低维嵌入**。比如当我们看到蓝色耳机时，我们想到的是三九，而不应是一花，那么蓝色耳机对于三九就是一个好的embedding，对于一花来说就不是一个好的embedding。



### 2.Beyond Reconstruction：Discriminator

[![img](E:/Development/Typora/images/20210125203004764.png)](https://img-blog.csdnimg.cn/20210125203004764.png)

​		除了使用重建误差来驱动模型训练外，可以使用其他的方式来衡量Encoder是否学到了关于输入的重要表征吗？答案自然是YES！假设我们现在有两类动漫人物的图像，一类是三玖，一类是凉宫春日。如果将三玖的图像丢给Encoder后，它就会给出一个**蓝色**的Embedding；如果Encoder接收的是凉宫春日的图像，它就会给出一个**黄色**的Embedding。

​			那么除了Encoder之外，还有一个**Discriminator**（这里可以就看作一个二分类的Classifier），它接收图像和Embedding，然后给出一个结果表示它们是否是两两对应的。

​		借助GAN的思想，我们用$\phi$来表述Discriminator(**Φ来表述Discriminator参数**)，希望通过训练最小化$D$的损失函数，**loss of the classification task is** 输出与标签的交叉熵$L_D$，那么训练模型参数来最小化$L_D$得到最小的损失值$L^*_D$：
$$
L^*_D = \min\limits_{\phi}L_{D}
$$
其中 **$L^*_D$来评估向量表示的好和不好。**

[![img](E:/Development/Typora/images/20210125203544742.png)](https://img-blog.csdnimg.cn/20210125203544742.png)

-   如果$L_D$的值比较小，就认为Encoder得到的Embedding**很有代表性**
-   相反如何$L_D$的值很大时，就认为得到的Embedding**不具有代表性**

[![img](E:/Development/Typora/images/2021012520380223.png)](https://img-blog.csdnimg.cn/2021012520380223.png)

​		既然我们知道如何来评估向量表示的好和不好，我们就是要调整encoder的参数$\theta$ ，然后用评估方法（根据L∗D 来评估）来让生成向量最优。
$$
\theta^* = arg\max\limits_{\theta}L^*_D
$$
带入$L^*_D$ 得到：
$$
\theta^* = arg\min\limits_{\theta}\min\limits_{\phi}L_D
$$
​		也就是说我们要训练encoder的参数$\theta$和discriminator的参数$\phi$，使得$L_D$ 最小化。

​		这个东西实际可以类比到最原始的auto-encoder模型要同时训练encoder和decoder使得reconstruction error最小。也就是说auto-encoder模型：

[![img](E:/Development/Typora/images/20210125204422661.png)](https://img-blog.csdnimg.cn/20210125204422661.png)

​		我们现在知道，auto-encoder跟刚才所说的，同使训练encoder和binary classifer一样，只是可以将auto-encoder视为一个特别情况。

​		如上图所示，刚才我们所说的discriminator是同时将image与embedding做为输入，但如果先将embedding做为input，得到的结果再跟image相减，那它就是一个reconstruction error了。

### 3.Sequential Data

除了图像数据外，我们也可以在序列数据上使用**Encoder-Decoder**的结构模型。

#### 3.1Skip thought

模型在大量的文档数据上训练结束后，Encoder接收一个句子，然后给出输入句子的上一句和下一句是什么。

[![img](E:/Development/Typora/images/20210125205046923.png)](https://img-blog.csdnimg.cn/20210125205046923.png)

>   👉 [Skip-Thought Vectors](https://papers.nips.cc/paper/5950-skip-thought-vectors.pdf)

​		这个模型训练过程和训练word embedding很像，因为训练word embedding的时候有这么一个原则，**就是两个词的上下文很像的时候**，这两个词的embedding就会**很接近**。换到句子的模型上，如果两个句子的上下文很像，那么这两个句子的embedding就应该很接近。

例如：

-   这个东西多少钱？答：10元。
-   这个东西多贵？答：10元。

发现答案一样，所以问句的embedding是很接近的。



#### 3.2Quick thought

​		而Quick thought是对于Skip thought的改进版本，它**不使用Decoder**，而是使用一个辅助的分类器。它将当前的句子、当前句子的下一句和一些随机采样得到的句子分别送到Encoder中得到对应的Embedding，然后将它们丢给**分类器**。因为当前的句子的Embedding和它下一句的Embedding应该是越接近越好，而它和随机采样句子的Embedding应该差别越大越好，因此分类器应该可以根据Embedding判断出哪一个代表的是当前句子的下一句。

[![img](E:/Development/Typora/images/20210125205117913.png)](https://img-blog.csdnimg.cn/20210125205117913.png)

>   👉 [AN EFFICIENT FRAMEWORK FOR LEARNING SENTENCE REPRESENTATIONS](https://arxiv.org/pdf/1803.02893.pdf)

​		模型中的classifier吃当前句子（Spring had come.）的向量表示，还吃下一句（And yet his crops didn’t grow.）和其它几个随机生成的句子的向量表示，这个classifier可以输出正确的下一句。
​		classifier和encoder是一起训练的。

​		实作上classifier做的事情很简单，就是直接拿当前句子的向量表示和所有句子的向量表示做内积，看谁的内积最大，谁就是下一个句子。这里为了防止机器作弊，直接把输入作为下一句（这样内积最大），还要附加条件：**使得当前句的向量表示和随机句子的向量表示越不像越好。**



#### 3.3Contrastive Predictive Coding(CPC)

这个模型和Quick thought的思想是一样的，不过是用在**声音信号**上的。它称为Contrastive Predictive Coding (CPC)的技术，它同样接收一段序列数据，然后**给出**它的接下来数据的预测结果。模型结构如下所示，具体内容可见原论文。

[![img](E:/Development/Typora/images/20210125205424947.png)](https://img-blog.csdnimg.cn/20210125205424947.png)

>   👉 [Representation Learning with Contrastive Predictive Coding](https://arxiv.org/pdf/1807.03748.pdf)



### 4.Feature Disentangle (特征解耦)

>   这一部分我们后面讲GAN的时候也有介绍和具体做法：👉 [机器学习-40-Feature Extraction(InfoGAN,VAE-GAN,BiGAN,Triple GAN,Feature Disentangle(Voice Conversion))](https://blog.csdn.net/qq_44766883/article/details/112690185)

#### 4.1Feature Disentangle

​		接下来我们来看**如何得到**解释性更好的Embedding，这样的方法也可以称为**Feature Disentangle**（特征结构）。因为对于Encoder的输入数据来说，经过Encoder得到的Embedding其实包含了关于它的很多类型的信息。例如，如果现在输入的是一段声音讯号，那么Embedding可能包含**内容信息、讲话者的信息……**

[![img](E:/Development/Typora/images/20210125205758336.png)](https://img-blog.csdnimg.cn/20210125205758336.png)

Disentangle意思是解开。

​		如果输入的是一段文字，Embedding可能包含关于它的**句法信息、语义信息……**

​		**我们希望计算机告诉我们各个维度的embedding代表的关系**。我们以声音讯号为例，假设通过Encoder得到的Embedding是一个**100维** 的向量，它只包含**内容**和讲话者身份两种信息。我们希望经过不断的训练，它的**前50维**代表内容信息，后50维代表讲话者的身份信息。

[![img](E:/Development/Typora/images/20210125210307634.png)](https://img-blog.csdnimg.cn/20210125210307634.png)

​		举例来说，我们希望输入一段声音讯号，而输出的假设是一个200维的向量，前100维是`语音信息`(语意内容的资讯)，而后100维则是`语者信息`(说话那个人的资讯)。这样训练出来的语音Encoder可以处理与人无关的语音，直接提取出语音特征；训练出来的语者Encoder可以用来做为识别语者身份的声纹特征。

​		又或者，我们可以有两个Encoder，一个负责处理语意内容，一个负责处理说话人的资讯，两个输出再合并给Decoder来做reconstructed。



#### 4.2Feature Disentangle- Voice Conversion

[![img](E:/Development/Typora/images/20210125210449862.png)](https://img-blog.csdnimg.cn/20210125210449862.png)

人和说话的内容是放在同一个向量里面的。

[![img](E:/Development/Typora/images/20210125210647782.png)](https://img-blog.csdnimg.cn/20210125210647782.png)

然后把女生说话的内容，和男生说话的特点合起来，就变成了男生在说How are you?

[![img](E:/Development/Typora/images/20210125210955428.png)](https://img-blog.csdnimg.cn/20210125210955428.png)

在上面这个例子中：

-   李老师：“你想读博吗？”
-   你：“走开！”

如果将语者信息换成新垣结衣的：

-   李老师(用新垣结衣的声音)：“你想读博吗？”
-   你：(lsp)

damn…

那么如何来实现**Feature Disentangle**呢？

​		**一种方法**就是使用GAN对抗的思想，我们在一般的Encoder-Decoder架构中引入一个**Classifier**，判别Embedding某个具体的部分是否代表了讲话者身份的信息，通过不断地训练，希望Encoder得到的Embedding可以骗过Classifier，那么那个具体的部分就表示了讲话者的信息。Classifier用来分辨语者的性别，encoder的原目标要加上一个限制，就是要使得不能让Classifier分辨出语者的性别，因此在训练的过程中就会把语者的性别放到后100维中去了，前100维就只剩下内容的信息了。

[![img](E:/Development/Typora/images/20210125211740204.png)](https://img-blog.csdnimg.cn/20210125211740204.png)

​		在实作过程中，通常是利用GAN来完成这个过程，也就是把encoder看做generator，把Classifier看做discriminator。Speaker classifier and encoder are learned iteratively.

​		**另一种方式**是改变网络结构，比如使用两个Encoder来分别得到内容信息和讲话者身份信息的Embedding，同时在Encoder中使用instance normalization，直接除掉global information。然后将得到的两个Embedding结合起来送入Decoder重建输入数据。这里不展开，两个normalization 的方法应该会有归一化专题讲解。

​		除了将两个Embedding直接组合起来的方式，还可以在Decoder中使用**Adaptive instance normalization。**

[![img](E:/Development/Typora/images/20210125211951379.png)](https://img-blog.csdnimg.cn/20210125211951379.png)

​		所以以上图架构来看，Encoder 1我们加入了IN layer，那就可能可以直接消掉说话人的资讯。但这还不能有任何的保证，因此我们还会在Decoder中加入AdaIN layer(adaptive instance normalization)让Encoder 2的output可以加到AdaIN内，来调整global information。

​		Encoder 1拿掉语者资讯，Encoder 2要确保可以保留语者资讯，这样重构之后的资讯才会是完整的。

[![img](E:/Development/Typora/images/20210125212323536.png)](https://img-blog.csdnimg.cn/20210125212323536.png)

这是学生的作品，训练过程中没有给任何中文的资讯，将李老师所说的话转成是外国人来说。

>   这里也有一个Demo可以看一下：👉 https://jjery2243542.github.io/voice_conversion_demo/



### 5.Discrete Representation(离散表示)

​		如果模型可以**从连续型的向量表示变成输出离散的表示**，那么对于表示的解释性估计会有更好的理解。（·Easier to interpret or clustering）

主要方法有：

-   one-hot
-   Binary vector
-   VQVAE(Vector Quantized Variational Auto-encoder)

#### one-hot

例如：用独热编码表示图片，做法很简单，在连续向量后面接相同维度的独热编码，看连续变量的哪个维度最大就用哪个做独热编码的1。

[![img](E:/Development/Typora/images/20210125212545664.png)](https://img-blog.csdnimg.cn/20210125212545664.png)

>   👉 [CATEGORICAL REPARAMETERIZATION WITH GUMBEL-SOFTMAX](https://arxiv.org/pdf/1611.01144.pdf)

​		通常情况下，Encoder输出的Embedding都是连续值的向量，这样才可以使用反向传播算法更新参数。但如果可以将其转换为离散值的向量，例如one-hot向量或是binary向量，我们就可以更加方便的观察Embedding的哪一部分表示什么信息。当然此时不能直接使用反向传播来训练模型，一种方式就是用强化学习来进行训练。

同样可以用Binary向量来表示图片，只不过判断0/1的方法不一样，这里如果大于0.5取1，小于取0

但是上面的模型在连续向量转离散向量的步骤上是不可做偏导的（无法GD）但是有办法：

当然，上面两个离散向量的模型比较起来，Binary模型要好，原因有两点：

1.  同样的类别Binary需要的参数量要比独热编码少，例如1024个类别Binary只需要10维即可，独热需要1024维；
2.  使用Binary模型可以处理在训练数据中未出现过的类别。



#### VQVAE(Vector Quantized Variational Auto-encoder,矢量量化变分自动编码器)

[![img](E:/Development/Typora/images/2021012521291716.png)](https://img-blog.csdnimg.cn/2021012521291716.png)

>   👉 [Neural Discrete Representation Learning](https://arxiv.org/abs/1711.00937)
>
>   👉 [Unsupervised speech representation learning using WaveNet autoencoders](https://arxiv.org/pdf/1901.08810.pdf)

VQVAE(Vector Quantized Variational Auto-encoder,矢量量化变分自动编码器)，它引入了一个Code book。它是将Embedding分为了很多的vector，然后比较哪一个和输入更像，就将其丢给Decoder重建输入。

1.  先用Encoder抽取为连续型的向量vector；
2.  再用vector与Codebook中的离散变量Vectori 进行相似度计算，例如上图中黄色的Vector3
3.  用Vector3还原输入信息。

上面的模型中，如果输入的是语音信息，那么语者信息和噪音信息会被过**滤掉**，因为上面的**Codebook**中保存的是离散变量，而内容信息是一个个的token，是容易用离散向量来表示的，而其他信息不适合用离散变量表示，因此会被过滤掉。
因此过滤信息是VQVAE的一个应用。



### 6.Sequence as Embedding

#### Sequence as Embedding

既然可以用离散向量来表示输入信息，那么我们也可以考虑用序列来embedding

[![img](E:/Development/Typora/images/20210125213159428.png)](https://img-blog.csdnimg.cn/20210125213159428.png)

>   👉 [Learning to Encode Text as Human-Readable Summaries using Generative Adversarial Networks](https://arxiv.org/abs/1810.02851)

如果我们要训练一个seq2seq2seq的Auto-encoder时，使用其他对抗的思想进行训练，就可以得到类似关于输入文档的摘要信息

-   第一个seq2seq吃的是一篇文章，输出的是一个sequence，也就是一串文字，而不是向量
-   然后第二个seq2seq再吃这串文字来试图还原原本的文章

例如：一篇文章经过encoder得到一串文字，然后这串文字再通过decoder还原回文章。

-   Only need a lot of documents to train the model.

中间的文字是什么？摘要。

但是这个模型效果是不好的，例如：

-   台湾大学会被机器抽取为：湾学，而不是台大。因为模型只要还原原文，回到台湾大学，而没有要求抽取出来的东西要符合语法规则。

因此模型可以改进为：GAN

[![img](E:/Development/Typora/images/20210125213719537.png)](https://img-blog.csdnimg.cn/20210125213719537.png)

如果我们希望可以学到一个人看的懂的summary，那就需要应用到GAN的概念，也就是用一个discriminator来判断这个句子是人写的，还是机器产生的。这样子，encoder就会学着去产出一个可以骗过discriminator又可以给decoder还原为文章的句子。 不过这个训练过程还是需要应用到RF的概念来硬train一发。

encoder产出的摘要要骗过discriminator：他看过很多台北大学，就不会搞出来湾学。discriminator来判断是否是人写的。

#### 例子

[![img](E:/Development/Typora/images/20210117130800372.png)](https://img-blog.csdnimg.cn/20210117130800372.png)

在这两个例子中，第一个例子总结的还算可以，但13个国家等关键字没有体现出来。第二个例子中，机器自动的将中华民国奥林匹克委员会总结成了奥委会，可能它以前训练的时候看过类似的奥林匹克委员会，知道直接简写成奥委会。

[![img](E:/Development/Typora/images/20210117131104221.png)](https://img-blog.csdnimg.cn/20210117131104221.png)

上面这两个例子就不算好了，看到印度尼西亚苏门答腊岛的印度尼西亚苏门直接简写成了印尼门，什么鬼？下面那个例子更离谱，总结出来的根本读都读不通！



### 7.Tree as Embedding

[![img](E:/Development/Typora/images/20210125214346742.png)](https://img-blog.csdnimg.cn/20210125214346742.png)

>   👉 [StructVAE: Tree-structured Latent Variable Models for Semi-supervised Semantic Parsing](https://arxiv.org/abs/1806.07832)
>
>   👉 [Unsupervised Recurrent Neural Network Grammars](https://arxiv.org/abs/1904.03746)

除了向量表示，序列表示，还有tree表示（不展开）