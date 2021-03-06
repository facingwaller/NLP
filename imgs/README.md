0: QA_LSTM_CNN_ATTENTION_IBM
---
- 解决的问题: answer selection. 即给定问题，计算多个答案的匹配率，选取最好的那个，适用于单轮对话. <br>
- 根据这篇文章我选用了bidirectional-lstm+hidden state maxpooling的方法，改进loss函数之后，比较有效的解决了问题到答案的映射问题。 这篇文章提出了多个模型，实际上都差不太多，作者是个比较有好的人，我给他发邮件询问模型的正负样本配比还回我了。
- 先来看看最基本的模型结构:
	- ![Alt text](/papers/imgs/bilstm.png)
	- 就是用两个biLSTM分别应用于问题和答案；这里的答案如果是假答案，则label 0，反之则label 1；
	- biLSTM分别run过一个句子之后，会得到n个中间hidden state和一个最终的hidden state；
	- 可以直接用最终的hidden state做cosine距离计算，从而比较真假答案和原问题之间的距离；
	- 也可以利用所有的hidden state，上图提到了做mean，或者max pooling；
- biLSTM + cnn
	- ![Alt text](/papers/imgs/bilstm_cnn.png)
	- 略
- biLSTM + cnn + attention
	- 注意到这里的attention是将问题向量应用到答案向量里面去；
	- 应用的方式是，先将问题的hidden state做mean，然后根据这个向量分别和所有的答案hidden state做计算出weigth，然后答案hidden state乘以做个weight作为attention based答案hidden state
	- 然后使用cnn分别对问题hidden states，和attention based答案hidden states做抽象；
- 评价：一篇简洁易懂又实用的论文，值得一看的；


1: Sequential Matching Network by MS:
---
- 原标题：Sequential Matching Network: A New Architecture for Multi-turn Response Selection in Retrieval-Based Chatbots<br>
- 解决的问题: 多轮对话里面的答案选取，本质上是一个匹配模型，而不是生成式模型。<br>
模型结构：<br>
![Alt text](/papers/imgs/sequential_matching.png)
- 结构讲解:<br>
	- First Layer: 说的是考虑了上下文，其实就是把历史对话逐个和带选取的答案进行比较，但是比较的方式比较特别。以前我们用的lstm都是直接拿问题和答案的最终hidden state进行consine距离计算或者全连接得到一个相似度，denoising autoencoder的话是直接问题答案的hidden state进行cosine相似度；但是在这里，它是先拿一个句子里面的所有词向量和另一个句子里面的所有词向量进行相似度计算之后得到一个相似度矩阵，即M1，然后拿GRU去run这两个句子，得到两个hidden states序列，再进行全部的相似度计算，得到M2，之后做卷积，即Covolution和pooling部分。<br>
	- Matching Accumulation: 卷积过后，每个历史问题和待选答案之间得到一个向量，这样得到一个序列的向量，再用第二个GRU去run这个向量，中间的hidden states用attention机制来做加权平均得到一个最终向量，（或者直接取最终向量拉倒）再经过全连阶层得到一个最终相似度。<br>
- 评价：<br>
	- 作者没分词，直接用单个的汉字来做的，第一层里面用词之间的相似度矩阵做convolution的方式比较特别，值得拿来研究。<br>
<br><br>

2: EFFICIENT VECTOR REPRESENTATION FOR DOCUMENTS THROUGH CORRUPTION
---
- 简介：就是用来把一句话转换成一个向量，但是不同于word2vec里面的方法，这个用了个dropout的方法训练document程度的vector
- 模型结构图：
![Alt text](/papers/imgs/doc2vecc.png)
	- 不同于word2vec的地方：
	<img src="/papers/imgs/doc2vecc_2.png" width="80%" height="80%">
	这里上面的那个式子我理解为每个doc vector是不断变化的(corruption)，但是初始为所有词向量的平均值；
	- 其他地方都很简单；
	- 具体效果有待实验，不过这个方法可以同时训练词向量；



<br><br>
3: Introduction to different gradient descend methods:
---
- 原文档连接：http://sebastianruder.com/optimizing-gradient-descent/index.html#stochasticgradientdescent <br>
- 目的：理解几个常用的gradient descend的机制和优劣
- Stochastic Gradient Descend:
![Alt text](/papers/imgs/SGD.PNG)
	- SGD 是相当于batch gradient descend来说的，SGD对每一个训练样本都更新其参数，这样频繁的更新会使得目标函数波动幅度很大，同时收敛迅速；
	![Alt text](/papers/imgs/sgd_fluctuation.png)
	- 当我们慢慢减少SGD的learning rate的时候，几乎可以保证损失函数能收敛到局部最小值(non-convex函数)和全局最小值(convex函数)
	- 优点就是收敛迅速，缺点个人认为是要不断调整learning rate.

- Adadelta
![Alt text](/papers/imgs/adadelta.png)
![Alt text](/papers/imgs/adadelta_decay.png)
	- Adadelta可以看作Adagrad变种而来，它们都假设gradients的方差遵循一个自相关的过程；作为一个自适应的gradient的方法，adadelta不需要设定learning rate；
	- 优点是训练初中期，加速效果不错，很快， 缺点是训练后期，反复在局部最小值附近抖动


- Adam
![Alt text](/papers/imgs/adam.png)
	- adam结合gradient的一阶和二阶矩估计来调整learning rate；上图中mt和vt分别是梯度的一阶和二阶矩估计；
	- 适用于大多非凸优化 - 适用于大数据集和高维空间
