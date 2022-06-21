---
layout: post
cid: 44
title: 我教我自己：Spark 编程基础（Scala 版本）- 第八章 Spark MLlib
slug: 44
date: 2021/04/13 17:38:00
updated: 2021/09/13 20:17:30
status: publish
author: AntiTopQuark
categories: 
  - Spark
  - 大数据
tags: 
  - value
  - 类型
  - 逻辑
  - 函数
  - 变量
  - 实例
  - 方法
  - 编译
  - 语句
  - 语言
  - ll
  - 数据库
  - 系统
  - 扩展
  - 存储
  - 用户
  - 文件
  - spark
  - 数据
  - 操作
  - 学习
  - 编码
  - 模型
  - 流程
  - 过程
  - 索引
  - set
  - 参数
  - 表
  - index
  - 字符
  - 测试
  - 自然
  - 处理
  - 文本
  - 单词
  - 文档
  - idf
  - 大数据
  - val
  - 转换
  - dataframe
  - 计算
  - 流水线
  - 特征
  - 机器学习
  - 步骤
  - 问题
  - 集合
  - 分隔
customSummary: 
mathjax: auto
noThumbInfoStyle: default
outdatedNotice: no
reprint: standard
thumb: 
thumbChoice: default
thumbDesc: 
thumbSmall: 
thumbStyle: default
---


# 8.1 基于大数据的机器学习
Spark 发展到目前，已经拥有了实时批计算、批处理、算法库、SQL、流计算等模块，成为了一个全平台的系统，把机器学习作为关键模块加入到 Spark 中也是大势所趋。
# 8.2 机器学习库 MLlib 概述
MLlib 主要包括以下几方面的内容：
- 算法工具：常用的学习算法，如分类、回归、聚类和协同过滤； 
- 特征化工具：特征提取、转化、降维和选择工具； 
- 流水线（Pipeline）：用于构建、评估和调整机器学习工作流的工具； 
- 持久性：保存和加载算法、模型和管道； 
- 实用工具：线性代数、统计、数据处理等工具。
MLlib 支持的机器学习算法：
![MLlib 支持的机器学习算法](http://www.sukidesu.top/usr/uploads/2019/12/882115911.png)
MLlib 库从 1.2 版本以后分为两个包： 
（1）spark.mllib 包含基于 RDD 的原始算法 API。Spark MLlib 历史比较长，在 1.0 以前的版本中就已经包含，提供的算法实现都是基于原始的 RDD。 
（2）spark.ml 则提供了基于 DataFrame 的、高层次的 API，其中，ML Pipeline API 可以用来构建机器学习流水线（PipeLine），弥补了原始 MLlib 库的不足，向用户提供了一个基于 DataFrame 的机器学习流水线式 API 套件。
使用 ML Pipeline API 可以很方便地进行数据处理、特征转换、规范化，以及将多个机器学习算法联合起来构建一个单一完整的机器学习工作流。这种方式提供了更灵活的方法，更符合机器学习过程的特点，也更容易从其他语言进行迁移。因此，Spark 官方推荐使用 spark.ml 包。如果新的算法能够适用于机器学习流水线（Pipeline）的概念，就应该将其放到 spark.ml 包中，如特征提取器和转换器等。需要注意的是，从 Spark2.0 开始，基于 RDD 的 API 进入维护模式，即不增加任何新的特性，并预期于 3.0 版本的时候被移除出 MLlib。
# 8.3 基本数据类型
spark.ml 包提供了一系列基本数据类型以支持底层的机器学习算法，主要的数据类型包括本地向量、标注点、本地矩阵等。
## 8.3.1 本地向量
本地向量分为**稠密向量（DenseVector）**和**稀疏向量（SparseVector）**两种。稠密向量使用双精度浮点型数组来表示每一维的元素，稀疏向量则是基于一个整型索引数组和一个双精度浮点型的值数组。例如，向量(1.0, 0.0, 3.0)的稠密向量表示形式是[1.0,0.0,3.0]，而稀疏向量形式则是（3, [0,2], [1.0, 3.0]），其中，3 是向量的长度，[0,2]是向量中非 0 维度的索引值，表示位置为 0、2 的两个元素为非零值，而[1.0，3.0]则是按索引排列的数组元素值。
所有本地向量都以 org.apache.spark.ml.linalg.Vector 为基类，DenseVector 和 SparseVector 分别是它的两个继承类，故推荐使用 Vectors 工具类下定义的工厂方法来创建本地向量。需要注意的是，Scala 会默认引入 scala.collection.immutable.Vector，如果要使用 spark.ml 包提供的向量类型，则要显式地引入 org.apache.spark.ml.linalg.Vector 这个类。下面给出一个实例。
![实例](http://www.sukidesu.top/usr/uploads/2019/12/2382068639.png)
## 8.3.2 标注点
标注点（Labeled Point）是一种带有标签（Label/Response）的本地向量，通常用在监督学习算法中，它可以是稠密或者是稀疏的。由于标签是用双精度浮点型来存储的，因此，标注点类型在回归（Regression）和分类（Classification）问题上均可使用。例如，对于二分类问题，则正样本的标签为 1，负样本的标签为 0；对于多类别的分类问题来说，标签则应是一个以 0 开始的索引序列：0，1，2…
![09431-m7fcss35zz.png](http://www.sukidesu.top/usr/uploads/2019/12/3321918589.png)
在实际的机器学习问题中，稀疏向量数据是非常常见的。**MLlib 提供了读取 LIBSVM 格式数据的支持，该格式被广泛用于 LIBSVM、LIBLINEAR 等机器学习库。**在该格式下，每一个带标签的样本点由以下格式表示：
label index1:value1 index2:value2 index3:value3 … 
其中，label 是该样本点的标签值，一系列 index:value 则代表了该样本向量中所有非零元素的索引和元素值。需要特别注意的是，index 是以 1 开始并递增的。
下面读取一个 LIBSVM 格式文件生成向量：
![10418-knpmfsh8mb.png](http://www.sukidesu.top/usr/uploads/2019/12/3889186770.png)
这里，examples.collect()把 RDD 转换为了向量，并取第一个元素的值。每个标注点共有 692 个维，其中，第 127 列对应的值是 51.0，第 128 列对应的值是 159.0，以此类推。
## 8.3.3本地矩阵
本地矩阵具有整型的行、列索引值和双精度浮点型的元素值，它存储在单机上。MLlib 支持稠密矩阵 DenseMatrix 和稀疏矩阵 SparseMatrix 两种本地矩阵。稠密矩阵将所有元素的值存储在一个列优先（Column-major）的双精度型数组中，而稀疏矩阵则将非零元素以列优先的 CSC（Compressed Sparse Column）模式进行存储。
本地矩阵的基类是 org.apache.spark.ml.linalg.Matrix，DenseMatrix 和 SparseMatrix 均是它的继承类。和本地向量类似，spark.ml 包也为本地矩阵提供了相应的工具类 Matrices，调用工厂方法即可创建实例。下面创建一个稠密矩阵：
![41617-jyv35yliibk.png](http://www.sukidesu.top/usr/uploads/2019/12/810686785.png)
创建稀疏矩阵：
![51241-b7xejrqg8w5.png](http://www.sukidesu.top/usr/uploads/2019/12/664345784.png)
这里创建了一个 3 行 2 列的稀疏矩阵[ [9.0,0.0]，[0.0,8.0]，[0.0,6.0]]。Matrices.sparse 的参数中，3 表示行数，2 表示列数。第 1 个数组参数表示列指针，其长度 = 列数 +1，表示每一列元素的开始索引值。第 2 个数组参数表示行索引，即对应的元素是属于哪一行，其长度 = 非零元素的个数。第 3 个数组即是按列先序排列的所有非零元素。在上面的例子中，（0,1,3）表示第 1 列有 1 个（=1-0）元素，第 2 列有 2 个（=3-1）元素；第二个数组(0, 2, 1)表示共有 3 个元素，分别在第 0、2、1 行。因此，可以推算出第 1 个元素位置在（0,0），值是 9.0。
# 8.4 机器学习流水线
## 8.4.1 流水线概念
机器学习流水线（Machine Learning Pipeline）是对流水线式工作流程的一种抽象，它包含了以下几个概念。 
（1）DataFrame。即 Spark SQL 中的 DataFrame，可容纳各种数据类型。与 RDD 数据集相比，它包含了模式（schema）信息，类似于传统数据库中的二维表格。流水线用 DataFrame 来存储源数据。例如，DataFrame 中的列可以是文本、特征向量、真实标签和预测的标签等。 
（2）转换器（Transformer）。转换器是一种可以将一个 DataFrame 转换为另一个 DataFrame 的算法。例如，一个模型就是一个转换器，它把一个不包含预测标签的测试数据集 DataFrame 打上标签，转化成另一个包含预测标签的 DataFrame。技术上，转换器实现了一个方法 transform()，它通过附加一个或多个列，将一个 DataFrame 转换为另一个 DataFrame。 
（3）评估器（Estimator）。评估器是学习算法或在训练数据上的训练方法的概念抽象。在机器学习流水线里，通常是被用来操作 DataFrame 数据并生成一个转换器。评估器实现了方法 fit()，它接受一个 DataFrame 并产生一个转换器。例如，一个随机森林算法就是一个评估器，它可以调用 fit()，通过训练特征数据而得到一个随机森林模型。 
（4）流水线（PipeLine）。流水线将多个工作流阶段（转换器和评估器）连接在一起，形成机器学习的工作流，并获得结果输出。 
（5）参数（Parameter）。即用来设置转换器或者评估器的参数。所有转换器和评估器可共享用于指定参数的公共 API。
## 8.4.2 流水线工作过程
要构建一个机器学习流水线，首先需要定义流水线中的各个 PipelineStage。PipelineStage 称为工作流阶段，包括转换器和评估器，如指标提取和转换模型训练等。有了这些处理特定问题的转换器和评估器，就可以按照具体的处理逻辑，有序地组织 PipelineStage 并创建一个流水线。例如：

    val pipline= new Pipeline().setStages(Array(stage1,stage2,stage3,))
在一个流水线中，上一个 PipelineStage 的输出，恰好是下一个 PipelineStage 的输入。流水线构建好以后，就可以把训练数据集作为输入参数，调用流水线实例的 fit()方法，***以流的方式来处理源训练数据***。该调用会返回一个 PipelineModel 类的实例，进而被用来预测测试数据的标签。更具体地说，流水线的各个阶段按顺序运行，输入的 DataFrame 在它通过每个阶段时会被转换，对于转换器阶段，在 DataFrame 上会调用 transform()方法，对于评估器阶段，先调用 fit()方法来生成一个转换器，然后在 DataFrame 上调用该转换器的 transform()方法。
![67020-8sfk848l0eo.png](http://www.sukidesu.top/usr/uploads/2019/12/2115489634.png)
一个流水线具有 3 个阶段，前两个阶段（Tokenizer 和 HashingTF）是转换器，第三个阶段（Logistic Regression）是评估器。下面一行表示流经这个流水线的数据，其中，圆柱表示 DataFrame。在原始 DataFrame 上调用 Pipeline.fit()方法执行流水线，每个阶段运行流程如下：
（1）在 Tokenizer 阶段，调用 transform()方法将原始文本文档拆分为单词，并向 DataFrame 添加一个带有单词的新列； 
（2）在 HashingTF 阶段，调用其 transform()方法将 DataFrame 中的单词列转换为特征向量，并将这些向量作为一个新列添加到 DataFrame 中； 
（3）在 LogisticRegression 阶段，由于它是一个评估器，因此会调用 LogisticRegression.fit()产生一个转换器 LogisticRegressionModel；如果工作流有更多的阶段，则在将 DataFrame 传递到下一个阶段之前，会调用 LogisticRegressionModel 的 transform()方法。
流水线本身就是一个评估器，因此，在流水线的 fit()方法运行之后，会产生一个流水线模型（PipelineModel），这是一个转换器，可在测试数据的时候使用。
![79749-sqatjwg580t.png](http://www.sukidesu.top/usr/uploads/2019/12/3411280141.png)
如图 8-3 所示，PipelineModel 具有与原流水线相同的阶段数，但是，原流水线中的所有评估器都变为转换器。调用 PipelineModel 的 transform()方法时，测试数据按顺序通过流水线的各个阶段，每个阶段的 transform()方法更新数据集（DataFrame），并将其传递到下一个阶段。通过这种方式，流水线和 PipelineModel 确保了训练和测试数据通过相同的特征处理步骤。这里给出的示例都是用于线性流水线的，即流水线中每个阶段使用由前一阶段产生的数据。但是，也可以构建一个有向无环图（DAG）形式的流水线，以拓扑顺序指定每个阶段的输入和输出列名称。流水线的阶段必须是唯一的实例，相同的实例不应该两次插入流水线。但是，具有相同类型的两个阶段实例，可以放在同一个流水线中，流水线将使用不同的 ID 创建不同的实例。此外，DataFrame 会对各阶段的数据类型进行描述，流水线和流水线模型（PipelineModel）会在实际运行流水线之前，做类型的运行时检查，但不能使用编译时的类型检查。
MLlib 评估器和转换器，使用统一的 API 指定参数。其中，Param 是一个自描述包含文档的命名参数，而 ParamMap 是一组（参数，值）对。将参数传递给算法主要有以下两种方法： 
（1）设置实例的参数。例如，lr 是的一个 LogisticRegression 实例，用 lr.setMaxIter(10)进行参数设置以后，可以使 lr.fit()至多迭代 10 次； 
（2）传递 ParamMap 给 fit()或 transform()函数。ParamMap 中的任何参数，将覆盖先前通过 set 方法指定的参数。 需要特别注意参数同时属于评估器和转换器的特定实例。如果同一个流水线中的两个算法实例（比如 LogisticRegression 实例 lr1 和 lr2），都需要设置 maxItera 参数，则可以建立一个 ParamMap，即 ParamMap（lr1.maxIter -> 10, lr2.maxIter -> 20），然后传递给这个流水线。
# 8.5 特征提取、转换和选择
## 8.5.1 特征提取
特征提取（Feature Extraction）是指利用已有的特征计算出一个抽象程度更高的特征集，也指计算得到某个特征的算法。本节列举 spark.ml 包含的特征提取操作，并讲解 TF-IDF 的操作实例。
**1.特征提取操作** 
spark.ml 包提供的提取操作包括以下几种。 
（1）TF-IDF。词频－逆向文件频率（Term Frequency–Inverse Document Frequency,TF-IDF）是文本挖掘领域常用的特征提取方法。给定一个语料库，TF-IDF 通过词汇在语料库中出现次数和在文档中出现次数，来衡量每一个词汇对于文档的重要程度，进而构建基于语料库的文档的向量化表达。 
（2）Word2Vec。Word2Vec 是由 Google 提出的一种词嵌入（Word Embedding）向量化模型。有 CBOW 和 Skip-gram 两种模型，spark.ml 使用的是后者。 
（3）CountVectorizer。CountVectorizer 可以看成是 TF-IDF 的退化版本，它仅通过度量每个词汇在文档中出现的次数（词频），来为每一个文档构建出向量化表达，可以通过设置超参数来限制向量维度，过滤掉出现较少的词汇。
例子：
在 spark.ml 库中，TF-IDF 被分成两部分：TF（+hashing）和 IDF。 
（1）TF。HashingTF 是一个转换器，在文本处理中，接收词条的集合，然后把这些集合转化成固定长度的特征向量。这个算法在哈希的同时，会统计各个词条的词频。 
（2）IDF。IDF 是一个评估器，在一个数据集上应用它的 fit()方法，产生一个 IDFModel。该 IDFModel 接收特征向量（由 HashingTF 产生），然后计算每一个词在文档中出现的频次。IDF 会减少那些在语料库中出现频率较高的词的权重。
例子：
![29658-sfao9mv26k.png](http://www.sukidesu.top/usr/uploads/2019/12/2907623368.png)
用 HashingTF 的 transform()方法把每个「词袋」哈希成特征向量。这里设置哈希表的桶数为 2000：
![75117-ctcphykznt7.png](http://www.sukidesu.top/usr/uploads/2019/12/1152977213.png)
可以看出，「词袋」中的每一个单词被哈希成了一个不同的索引值。以「I heard about Spark and I love Spark」为例，表 8-2 给出 featurizedData.foreach{println}执行结果的含义。
![93356-uf75eoautno.png](http://www.sukidesu.top/usr/uploads/2019/12/3043453566.png)
调用 IDF 方法来重新构造特征向量的规模，生成的变量 idf 是一个评估器，在特征向量上应用它的 fit()方法，会产生一个 IDFModel（名称为 idfModel）。
![76266-8di31t0nilo.png](http://www.sukidesu.top/usr/uploads/2019/12/1017920966.png)
调用 IDFModel 的 transform()方法，可以得到每一个单词对应的 TF-IDF 度量值。
![51238-5fksr3gzsgt.png](http://www.sukidesu.top/usr/uploads/2019/12/624515325.png)

「[240,333,1105,1329,1357,1777]」代表着「i, heard, about, and, love, spark」的哈希值。通过第 1 句与第 2 句的词汇对照，可以推测出 1329 代表了「i」，而 1105 代表了「spark」，其 TF-IDF 值分别是 0.5753641449035617 和 0.6931471805599453。这两个单词都在第一句中出现了两次，而「i」在第二句中还多出现了一次，从而导致「i」的 TF-IDF 度量值较低。相对而言，「spark」可以对文档进行更好地区分。通过 TF-IDF 得到的特征向量，在机器学习的后续步骤中可以被应用到相关的学习方法中。
# 8.5.2 特征转换
机器学习处理过程经常需要对数据或者特征进行转换，通过转换可以消除原始特征之间的相关或者减少冗余，得到新的特征。本节介绍 spark.ml 包含的特征转换操作，并给出相关实例。
1.特征转换操作 
spark.ml 包提供了大量的特征转换操作。
（1）Tokenizer 
Tokenizer 可以将给定的文本数据进行分割（根据空格和标点），将**文本中的句子变成独立的单词序列，并转为小写表达**。spark.ml 还提供了带规范表达式的升级版本 RegexTokenizer，可以为其指定一个规范表达式作为分隔符或词汇的模式（Pattern），还可以指定最小词汇长度来过滤掉那些「很短」的词汇。
（2）StopWordsRemover 
StopWordsRemover 可以将文本中的停止词（出现频率很高、但对文本含义没有大的贡献的冠词、介词和部分副词等）去除，spark.ml 中已经自带了常见的西方语言的停止词表，可以直接使用。需要注意的是，StopWordsRemover 接收的文本，必须是已经过分词处理的单词序列。
（3）NGram 
NGram 将经过分词的一系列词汇序列，转变成自然语言处理中常用的「n-gram」模型，即通过该词汇序列可构造出的、所有由连续相邻的 n 个词构成的序列。需要注意的是，当词汇序列小于 n 时，NGram 不产生任何输出。
（4）Binarizer 
Binarizer 可以根据某一给定的阈值，将数值型特征转化为 0-1 的二元特征。对于给定的阈值来说，特征大于该阈值的样本会被映射为 1.0，反之，则被映射为 0.0。
（5）PCA 
PCA（主成分分析）是一种通过数据旋转变换进行降维的统计学方法，其本质是在线性空间中进行一个基变换，使得变换后的数据投影在一组新的「坐标轴」上的方差最大化，并使得变换后的数据在一个较低维度的子空间中，尽可能地表示原有数据的性质。
（6）PolynomialExpansion 
PolynomialExpansion 对给定的特征进行多项式展开操作，对于给定的「度」（如 3），它可以将原始的数值型特征，扩展到相应次数的多项式空间（所有特征相乘组成的 3 次多项式集合构成的特征空间）中去。
（7）Discrete Cosine Transform (DCT) 
离散余弦变换（DCT）是快速傅里叶变换（FFT）的一种衍生形式，是信号处理中常用的变换方法，它将给定的 N 个实数值序列从时域上转变到频域上。spark.ml 库中提供的 DCT 类使用的是 DCT-II 的实现。 
（8）StringIndexer 
StringIndexer 可以把一列类别型的特征（或标签）进行编码，使其数值化，索引的范围从 0 开始，该过程可以使得相应的特征索引化，使得某些无法接受类别型特征的算法可以被使用，并提高决策树等机器学习算法的效率。 
（9）IndexToString 
与 StringIndexer 相对应，IndexToString 的作用是把已经索引化的一列标签重新映射回原有的字符形式。主要使用场景一般都是和 StringIndexer 配合，先用 StringIndexer 将标签转化成标签索引，进行模型训练，然后，在预测标签的时候，再把标签索引转化成原有的字符标签（原有标签会从列的元数据中获取）。
(10）OneHotEncoder 
OneHotEncoder 会把一列类别型特征（或称名词性特征，Nominal/Categorical Features），映射成一系列的二元连续特征，原有的类别型特征有几种可能的取值，这一特征就会被映射成几个二元连续特征，每一个特征代表一种取值，若某个样本表现出该特征，则取 1，否则取 0。
（11）VectorIndexer 
VectorIndexer 将整个特征向量中的类别型特征处理成索引形式。当所有特征都已经被组织在一个向量中，又想对其中某些单个类别型分量进行索引化处理时，VectorIndexer 根据用户设定的阈值，自动确定哪些分量是类别型，并进行相应的转换。 
（12）Interaction 
Interaction 可以接受多个向量或浮点数类型的列，并基于这些向量生成一个包含所有组合的乘积的新向量（可以看成各向量的笛卡儿积的无序版本）。新向量的维度是参与变换的所有向量的维度之积。 
（13）Normalizer 
Normalizer 可以对给定的数据集进行「规范化」操作，即根据设定的范数（默认为 L2-norm），将每一个样本的特征向量的模进行单位化。规范化可以消除输入数据的量纲影响，已经广泛应用于文本挖掘等领域。 
（14）StandardScaler 
StandardScaler 将给定的数据集进行「标准化」操作，即将每一个维度的特征都进行缩放，以将其转变为具有单位方差以及/或 0 均值的序列。
（15）MinMaxScaler
 MinMaxScaler 根据给定的最大值和最小值，将数据集中的各个特征缩放到该最大值和最小值范围之内，当没有具体指定最大/最小值时，默认缩放到[0,1]区间。
 （16）MaxAbsScaler 
MaxAbsScaler 用每一维特征的最大绝对值对给定的数据集进行缩放，实际上是将每一维度的特征都缩放到[-1,1]区间中。
（17）Bucketizer 
Bucketizer 对连续型特征进行离散化操作，使其转变为离散特征。用户需要手动给出对特征进行离散化的区间的分割位置（如分为 n 个区间，则需要有 n+1 个分割值），该区间必须是严格递增的。 
（18）ElementwiseProduct ElementwiseProduct 适合给整个特征向量进行「加权」操作，给定一个权重向量指定出每一特征的权值，它将用此向量对整个数据集进行相应的加权操作。其过程相当于代数学上的哈达玛乘积（Hadamard product）。 
（19）SQLTransformer SQLTransformer 通过 SQL 语句对原始数据集进行处理，给定输入数据集和相应的 SQL 语句，它将根据 SQL 语句定义的选择条件对数据集进行变换。目前，SQLTransformer 只支持 SQL SELECT 语句。
（20）VectorAssembler VectorAssembler 可以将输入数据集的某一些指定的列组织成单个向量。特别适用于需要针对单个特征进行处理的场景，当处理结束后，将所有特征组织到一起，再送入那些需要向量输入的机器学习算法，如 Logistic 回归或决策树。 
（21）QuantileDiscretizer QuantileDiscretizer 可以看成是 Bucketizer 的扩展版，它将连续型特征转化为离散型特征。不同的是，它无需用户给出离散化分割的区间位置，只需要给出期望的区间数，即会自动调用相关近似算法计算出相应的分割位置。
例子：
1.StringIndexer
![25897-sqmjuqp9br.png](http://www.sukidesu.top/usr/uploads/2019/12/2836749467.png)
2.IndexToString
![02086-774hrdeddru.png](http://www.sukidesu.top/usr/uploads/2019/12/605335152.png)


