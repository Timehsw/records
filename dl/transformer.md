说到自然语言处理, 语言模型, 命名实体识别, 机器翻译, 可能很多人想到的**LSTM等循环神经网络**, 但目前其实LSTM起码在自然语言处理领域已经过时了, 在Stanford阅读理解数据集**(SQuAD2.0)**榜单里, 机器的成绩已经超人类表现, 这很大程度要归功于**transformer的BERT预训练模型**.    
今天我们来讲一下**transformer模型**, 你不需要有很多深度学习和数学基础, 我来用简单的语言和可视化的方法从零讲起.   
transformer是谷歌大脑在2017年底发表的论文**attention is all you need**中所提出的seq2seq模型. 现在已经取得了大范围的应用和扩展, 而BERT就是从transformer中衍生出来的预训练语言模型.    

在我们开始之前, 允许我简单说一下目前自然语言处理领域的现状, 目前**transformer**模型已经得到广泛认可和应用, 而应用的方式主要是先进行**预训练语言模型**, 然后把预训练的模型适配给下游任务, 以完成各种不同的任务, 如分类, 生成, 标记等等, 预训练模型非常重要, 预训练的模型的性能直接影响下游任务的性能, 通过我制作的这几期视频, 我有信心让小伙伴们充分理解transformer并具备一定衍生模型的设计和编写能力.

为了让大家充分理解和初步使用transformer和训练BERT, 并应用到自己的需求上, 这个连载课程将包括以下几个视频来完成, 今天讲解第一部分:  

__一. transformer编码器(理论部分)__:

0. $transformer$模型的直觉, 建立直观认识;
1. $positional \ encoding$, 即**位置嵌入**(或位置编码);
2. $self \ attention \ mechanism$, 即**自注意力机制**与**注意力矩阵可视化**;
3. $Layer \ Normalization$和残差连接.
4. $transformer \ encoder$整体结构.   

__二. transformer代码解读, 语料数据预处理, BERT的预训练和情感分析的应用__:

__三. sequence 2 sequence(序列到序列)模型或Name Entity Recognition(命名实体识别)(待定)__:   
此部分根据前面的反馈待定.

## 0. $transformer$模型的直觉, 建立直观认识;

首先来说一下**transformer**和**LSTM**的最大区别, 就是LSTM的训练是迭代的, 是一个接一个字的来, 当前这个字过完LSTM单元, 才可以进下一个字, 而transformer的训练是并行了, 就是所有字是全部同时训练的, 这样就大大加快了计算效率, transformer使用了位置嵌入$(positional \ encoding)$来理解语言的顺序, 使用自注意力机制和全连接层来进行计算, 这些后面都会详细讲解.   
transformer模型主要分为**两大部分**, 分别是**编码器**和**解码器**, **编码器**负责把自然语言序列映射成为**隐藏层**(下图中**第2步**用九宫格比喻的部分), 含有自然语言序列的数学表达. 然后解码器把隐藏层再映射为自然语言序列, 从而使我们可以解决各种问题, 如情感分类, 命名实体识别, 语义关系抽取, 摘要生成, 机器翻译等等, 下面我们简单说一下下图的每一步都做了什么:   
1. 输入自然语言序列到编码器: Why do we work?(为什么要工作);
2. 编码器输出的隐藏层, 再输入到解码器;
3. 输入$<start>$(起始)符号到解码器;
4. 得到第一个字"为";
5. 将得到的第一个字"为"落下来再输入到解码器;
6. 得到第二个字"什";
7. 将得到的第二字再落下来, 直到解码器输出$<end>$(终止符), 即序列生成完成.

![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/dl/transformer/transformer.jpg)

我们这节课的内容限于**编码器**部分, 即把**自然语言序列映射为隐藏层的数学表达的过程**, 因为理解了编码器中的结构, 理解解码器就非常简单了,最重要的是**BERT预训练模型**只用到了编码器的部分, 也就是先用编码器训练一个语言模型, 然后再把它适配给其他五花八门的任务.   
如果你不知道**语言模型**是什么, 没关系, 这丝毫不影响本节课内容的理解, 下次我们讲**BERT**的时候会讲.   
而且我们用编码器就能够完成一些自然语言处理中比较主流的任务, 如情感分类, 语义关系分析, 命名实体识别等, 解码器的内容和序列到序列模型有机会我们会涉及到.

### Transformer Block结构图,   注意: 为方便查看, 下面的内容分别对应着上图第1, 2, 3, 4个方框的序号:

![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/dl/transformer/block.jpg)

## 1. $positional \ encoding$, 即**位置嵌入**(或位置编码);

由于transformer模型**没有**循环神经网络的迭代操作, 所以我们必须提供每个字的位置信息给transformer, 才能识别出语言中的顺序关系.   
现在定义一个位置嵌入的概念, 也就是$positional \ encoding$, 位置嵌入的维度为$[max \ sequence \ length, \ embedding \ dimension]$, 嵌入的维度同词向量的维度, $max \ sequence \ length$属于超参数, 指的是限定的最大单个句长.   
注意, 我们一般以字为单位训练transformer模型, 也就是说我们不用分词了, 首先我们要初始化字向量为$[vocab \ size, \ embedding \ dimension]$, $vocab \ size$为总共的字库数量, $embedding \ dimension$为字向量的维度, 也是每个字的数学表达.    
在这里论文中使用了$sine$和$cosine$函数的线性变换来提供给模型位置信息:   

$$PE_{(pos,2i)} = sin(pos / 10000^{2i/d_{\text{model}}}) \quad PE_{(pos,2i+1)} = cos(pos / 10000^{2i/d_{\text{model}}})\tag{eq.1}$$

上式中$pos$指的是句中字的位置, 取值范围是$[0, \ max \ sequence \ length)$, $i$指的是词向量的维度, 取值范围是$[0, \ embedding \ dimension)$, 上面有$sin$和$cos$一组公式, 也就是对应着$embedding \ dimension$维度的一组奇数和偶数的序号的维度, 例如$0, 1$一组, $2, 3$一组, 分别用上面的$sin$和$cos$函数做处理, 从而产生不同的周期性变化, 而位置嵌入在$embedding \ dimension$维度上随着维度序号增大, 周期变化会越来越慢, 而产生一种包含位置信息的纹理, 就像论文原文中第六页讲的, 位置嵌入函数的周期从$2 \pi$到$10000 * 2 \pi$变化, 而每一个位置在$embedding \ dimension$维度上都会得到不同周期的$sin$和$cos$函数的取值组合, 从而产生独一的纹理位置信息, 模型从而学到位置之间的依赖关系和自然语言的时序特性.   
下面画一下位置嵌入, 可见纵向观察, 随着$embedding \ dimension$增大, 位置嵌入函数呈现不同的周期变化.