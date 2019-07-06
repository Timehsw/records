# 逐步介绍Lstm和GRU算法

![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/dl/cnn/lstm01.png)

# 短时记忆问题
RNN(Recurrent Neural Networks)拥有短期记忆的功能。如果一个句子足够长，那么最后的一个单词，就很难从早期的单词中获取足够的信息了。因此，如果你想尝试从段落文本中去做预测，RNN可能就从一开始就遗落了很多重要的信息。

在反向传播的过程中，RNN还会面临梯度消失的问题。梯度用于更新神经网络权重的值。梯度消失是指随着时间的推移，当梯度进行传播的时候，梯度会收缩。如果梯度值变得非常小，则不会产生太多的学习效果了。
![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/dl/rnn/gradient_update_rule.png)

因此在RNN中，小梯度更新的层就会停止更新。这些层通常是早期的层，也就是离当前时间越远的层。因为这些层无法学习了，所以RNN就忘记了它在较长序列中看到的内容，这也就是RNN具有短期记忆的原因了。

ok，这是RNN网络结构，面临的一个问题。那么如果解决这个梯度消失的问题了。有人提出了LSTM和GRU的网络结构，他们的网络正好就可以解决这个问题

# Lstm和GRU的解决方案
Lstm和GRU就是作为短期记忆的解决方案而被发明的。它们具有称为门(gate)的内部机制，用于调节信息的流动。
![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/dl/rnn/lstm_gru.png)

这些门可以学习到在序列中保持和丢弃哪些数据是重要的。通过这样做，它可以沿着长链序列传递相关的有用信息进行预测。几乎所有的基于递归神经网络的现有技术结果都是通过这两个网络实现的。LSTM和GRU现在广泛用于语音识别，语音合成，文本生成等等领域中。它们甚至可以为视频生成字幕。

接下来，详细的解释为什么LSTM和GRU可以做到这些的原因。

# 一个直观的例子
OK，当我们在京东或者淘宝上在线购物的时候，我们会通过看别人的评论来决定究竟买不买这个商品。也就是说，我们首先看评论来决定别人认为它是好还是坏。
![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/dl/rnn/demo_shop.png)

当我们看到这个评论的时候，我们的潜意识仅仅只记住一些重要的单词。我们可能记住一些单词，像“amazing”,"perfectly balanced breakfast". 我们并不关心一些这些的词，比如"this","gava","all","should"....如果你的朋友第二天问你评论说了啥。你可能不会一字一句的全记住了。那你可能就只记住了一些重要的单词，比如“will definity be buying again”.如果你和我很像，那么其他的话就从记忆中慢慢的消失了，遗忘了。
![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/dl/rnn/demo2_shop.gif)

这基本上就是LSTM和GRU的作用。它能够学习到保留哪些相关的有用的信息去做预测，而遗忘掉不相关的数据。这上面的例子中，你记住的单词让你判断这个商品是好的。




# 参考：
https://towardsdatascience.com/illustrated-guide-to-lstms-and-gru-s-a-step-by-step-explanation-44e9eb85bf21