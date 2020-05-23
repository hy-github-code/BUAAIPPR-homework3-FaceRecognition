# 图像处理第三次大作业 - 人脸识别

17373489 张佳一



## 1 前言

人脸识别是指能够识别或验证图像或视频中的主体的身份的技术。自七十年代以来，人脸识别已经成为了计算机视觉和生物识别领域被研究最多的主题之一。基于人工设计的特征和传统机器学习技术的传统方法近来已被使用非常大型的数据集训练的深度神经网络取代。

在本次作业中，我对比较流行的几种人脸识别方法进行了较为全面的总结及对比分析，其中既包括比较经典的传统方法：*特征脸方法（Eigenfaces）*、*线性判别分析法（Fisher Faces）*、*局部二值模式直方图法*，也有~~深度学习方法~~。



## 2 数据集介绍

本次作业使用了助教提供的第一个数据集 **PIE dataset** ，包含了68个人在**五种不同姿态下**的共11554张面部图像数据，并以MAT格式的文件保存，如下：

![image-20200523232433516](assets/image-20200523232433516.png)

其中 `Pose05_64x64.mat` 的文件信息如下：

![image-20200523232942477](assets/image-20200523232942477.png)



由上图可知，每条 `fea` 与 `gnd` 分别为图像数据和标签，`isTest` 为1时，表明为测试图，不参与训练。 此外，我们还能看到姿态为**Pose05** 的图片共有3332张，每张图片数据为一个 64x64 = 4096 的像素矩阵构成。其余四种姿态的数据信息与之类似，这里不在一一列举。

下面展示的是 `label` 为1的人的在五种姿态下的图像（具体见 `/iamge/sample` 文件夹下）：

![image-20200523234024129](assets/image-20200523234024129.png)

可见，这五张图像分别是从测试者的“左侧”、“仰视”、“俯视”、“正视” 以及 “右侧” 角度拍摄的。

>  在这里要感谢助教对数据集提前做了筛选与处理，相比图像格式的数据，Mat格式不仅占用空间小，也极大的方便了我们读取图像数据，给助教一个大大的赞！



## 3 传统人脸识别方法

### 3.1 Eigen Faces 

Eigenfaces就是特征脸的意思，是一种从主成分分析（Principal Component Analysis，PCA）中导出的人脸识别和描述技术。特征脸方法的主要思路就是将输入的人脸图像看作一个个矩阵，通过在人脸空间中一组正交向量，并选择最重要的正交向量，作为“主成分”来描述原来的人脸空间。为了更好地理解特征脸方法，需要先了解PCA的主要过程。

#### PCA主要过程
在很多应用中，需要对大量数据进行分析计算并寻找其内在的规律，但是数据量巨大造成了问题分析的复杂性，因此我们需要一些合理的方法来减少分析的数据和变量同时尽量不破坏数据之间的关联性。于是这就有了主成分分析方法，PCA作用：

数据降维。减少变量个数；确保变量独立；提供一个合理的框架解释。
去除噪声，发现数据背后的固有模式。
PCA的主要过程：

特征中心化：将每一维的数据（矩阵A）都减去该维的均值，使得变换后（矩阵B）每一维均值为0；
计算变换后矩阵B的协方差矩阵C；
计算协方差矩阵C的特征值和特征向量；
选取大的特征值对应的特征向量作为”主成分”，并构成新的数据集；



#### 特征脸方法

特征脸方法就是将PCA方法应用到人脸识别中，将人脸图像看成是原始数据集，使用PCA方法对其进行处理和降维，得到“主成分”——即特征脸，然后每个人脸都可以用特征脸的组合进行表示。这种方法的核心思路是认为同一类事物必然存在相同特性（主成分），通过将同一目标（人脸图像）的特性寻在出来，就可以用来区分不同的事物了。

特征脸方法的过程（先计算特征脸，然后识别人脸）：

将训练集中的N个人脸拉成一列（reshape(1,1)），然后组合在一起形成一个大矩阵A。若人脸图像大小为m * m，则矩阵A的维度是m * m * N；
将N个人脸在对应的维度求平均，得到一个“平均脸”；
将矩阵A中N个图像都减去“平均脸”，得到新矩阵B；
计算B的协方差矩阵；
计算协方差矩阵的特征值和特征向量（特征脸）；
将训练集图像和测试集图像都投影到特征向量空间中，再使用聚类方法（最近邻或k近邻等）得到里测试集中的每个图像最近的图像，进行分类即可。

#### 特征脸识别的局限性

要让系统准确识别需要保证人脸图像满足：

待识别图像中人脸尺寸接近特征脸中人脸的尺寸；
待识别人脸图像必须为正面人脸图像。
若不满足此条件，识别错误率很高。从PCA方法的过程可以看出，特征脸识别的方法是以每张人脸的一个维度（可以看出是矩阵的一列）为单位进行处理的，求得的特征向量（特征脸）中包含训练集每个纬度的绝大部分信息。但是若测试集中人脸尺寸不同，那么与特征脸中维度的也就没法对应起来。





### 3.2 Fisher Faces

Fisher线性判别分析（linear discriminant analysis，LDA）
线性判别分析是由Fisher提出的线性判别方法，可以用来处理两类的线性判别问题。两类的线性判别问题可以看做所有的样本投影到一个方向（或者说是一个维度空间中），然后再这个空间中确定一个分类的阈值。过这个阈值点且与投影方向垂直的超平面就是分类面。判别思路是选择投影方向，使得投影后两类相隔尽可能远，类内又尽可能聚集（类间方差最大，类内方差最小）。
它的过程分为：

确定最优的投影方向：
在这个方向上确定分类阈值；





### 3.3 Local Binary Pattern Histograms





## 4 基于深度神经网络的人脸识别方法

卷积神经网络（CNN）是人脸识别方面最常用的一类深度学习方法。深度学习方法的主要优势是可用大量数据来训练，从而学到对训练数据中出现的变化情况稳健的人脸表征。这种方法不需要设计对不同类型的类内差异（比如光照、姿势、面部表情、年龄等）稳健的特定特征，而是可以从训练数据中学到它们。深度学习方法的主要短板是它们需要使用非常大的数据集来训练，而且这些数据集中需要包含足够的变化，从而可以泛化到未曾见过的样本上。

用于人脸识别的 CNN 模型可以使用不同的方法来训练。其中之一是将该问题当作是一个分类问题，训练集中的每个主体都对应一个类别。训练完之后，可以通过去除分类层并将之前层的特征用作人脸表征而将该模型用于识别不存在于训练集中的主体。



对于基于 CNN 的人脸识别方法，影响准确度的因素主要有三个：训练数据、CNN 架构和损失函数。因为在大多数深度学习应用中，都需要大训练集来防止过拟合。一般而言，为分类任务训练的 CNN 的准确度会随每类的样本数量的增长而提升。这是因为当类内差异更多时，CNN 模型能够学习到更稳健的特征。但是，对于人脸识别，我们感兴趣的是提取出能够泛化到训练集中未曾出现过的主体上的特征。因此，用于人脸识别的数据集还需要包含大量主体，这样模型也能学习到更多类间差异。



## 5 实验结果对比与分析



## 6 总结





特征脸算法对光照十分敏感。



（1）主成分分析（PCA）——Eigenfaces（特征脸）——函数：cv2.face.EigenFaceRecognizer_create（）

PCA：低维子空间是使用主元分析找到的，找具有最大方差的哪个轴。

缺点：若变化基于外部（光照），最大方差轴不一定包括鉴别信息，不能实行分类。



（2）线性判别分析（LDA）——Fisherfaces（特征脸）——函数： cv2.face.FisherFaceRecognizer_create()

LDA:线性鉴别的特定类投影方法，目标：实现类内方差最小，类间方差最大。

（3）局部二值模式（LBP）——LocalBinary Patterns Histograms——函数：cv2.face.LBPHFaceRecognizer_create()

PCA和LDA采用整体方法进行人脸辨别，LBP采用局部特征提取，除此之外，还有的局部特征提取方法为：

盖伯小波（Gabor Waelets）和离散傅里叶变换（DCT）。




## 参考文献

[1] http://openbio.sourceforge.net/resources/eigenfaces/eigenfaces-html/facesOptions.html

[2] https://blog.csdn.net/wanghz999/article/details/78817265

