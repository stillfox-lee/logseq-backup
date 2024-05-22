title:: What Is GPT and Why Does It Work?
author:: [[laisky.com]]
url:: https://blog.laisky.com/p/what-is-gpt/?lang=zh#The+Concept+of+Embeddings
tags:: #[[ai]] #[[Readwise]]  
![](https://s3.laisky.com/uploads/2024/02/chatgpt-demo.png)

- > 这就是梯度下降法的局部最优解问题，这个问题也没什么特别好的解决办法，最简单粗暴的方式就是多试几次，每次都从不同的随机位置开始。一般来说，你不会运气差到每次都走进同一个小凹坑吧。（这也是为什么有人把训练戏称为炼丹，运气在其中占了很大比重） ([View Highlight](https://read.readwise.io/read/01hs7m7f00ph1jwtfwfb27gyr5))
	- 实际工程上，如何确定自己是在“小凹坑”？还是已经到了山底？或者说，其实模型效果已经够好就行了，不需要走到山底？
- > 这些等效模型某种意义上可以解释天才的存在。
  
  即是是完全一样的网络结构，只是各自有着不同的随机化初始值。 然后接受了一模一样的训练，在校验集上也取得了一模一样的成绩。 但其实每个网络学会的参数都是不一样的。
  
  在更大的范围内进行推理时，这个差距才会显现出来，有些人就是能取得更小的过拟合，在人身上被称为天才的洞察力。
  
  换句话说，一样的天资，一样的后天教育，仍然会随机的诞生天才，这就是神经网络的内在随机性。 ([View Highlight](https://read.readwise.io/read/01hs7mk1s7sftb8bydjfewqbs6))
	- 有趣，人类的情况也是类似的。
- > 2017 年 6 月 26 日，Google 的研究人员提交了论文《Attention Is All You Need》，提出了名为 transformer 的神经网络结构。
  
  Google 发现，其实根本不需要 RNN，纯粹只靠 attention 就足够了，一系列 attention heads 的组合就可以产生非常好的输出，这就是 transformer，也是那篇著名论文名字的来源。
  
  Attention 的具体构建方式为： （这个 pipeline 的每一个组成部分，都是纯粹的神经网络，没有任何的预先设计）
  
  1.  将输入的文字转换为 token
  2.  将 token 值用神经网络转换为 embedding
  3.  将每一个 token 在输入中的所在位置，也进行一轮 embedding
  4.  将两轮 embedding 的结果相加，交给 softmax 函数输出概率向量
  
  ![](https://s3.laisky.com/uploads/2024/02/llm-gpt-2.png) ([View Highlight](https://read.readwise.io/read/01hs7rzfjc33a5qm39f57wpdx1))
- > 为什么要对 token-value 和 token-position 分别 embedding，然后再相加？
  
  没人知道。
  
  2017 年的某一天 Google 的某个人这么做了，然后发现效果很好… ([View Highlight](https://read.readwise.io/read/01hs7rzvdw1y6jt0ev4qh6j9bp))
	- 又是不可解释