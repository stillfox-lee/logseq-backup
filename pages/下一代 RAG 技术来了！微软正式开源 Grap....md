title:: 下一代 RAG 技术来了！微软正式开源 Grap...
author:: [[@tuturetom on Twitter]]
url:: https://twitter.com/tuturetom/status/1808320407583576163

![](https://pbs.twimg.com/profile_images/1033199673035522048/WI-JLSAc.jpg)

- > 下一代 RAG 技术来了！微软正式开源 GraphRAG🔥 
  
  通过 LLM 构建知识图谱结合图机器学习，极大增强 LLM 在处理私有数据时的性能，同时 GraphRAG 具备连点成线的跨大型数据集的复杂语义问题推理能力。
  
  \- 提供 Index、Query 工具链与建模工具
- > GraphRAG 介绍地址：https://t.co/RcCcrdaF0u
  
  普通 RAG 技术在私有数据，如企业的专有研究、商业文档表现非常差
  
  \- 问答问题需要遍历不同信息片段提供综合简介
- > GraphRAG 基于微软图形机器学习的研究和工具沉淀的解决方案：
  
  \- 企业图谱：https://t.co/gTZVzopHNW
- > GraphRAG 的流程：
  
  \- Index：将输入切分成 TextUtils，使用 LLM 提取三元组和实体关系、使用 Leiden 对图进行聚类分层  形成层级社区，并且提供每个社区的语义化摘要
- > Leiden 技术参考论文：https://t.co/GypKBpPWfO ([View Tweet](https://twitter.com/tuturetom/status/1808322421130514628))
- > Global Search 参考 DSL 和 Prompt 实现：https://t.co/9pUCuy3Gy4
  
  Local Search 参考：https://t.co/Hjd7ywSjtJ ([View Tweet](https://twitter.com/tuturetom/status/1808322893652414837))
- > 为了适应 GraphRAG 的技术，有必要微调 Prompt，GraphRAG 也提供的全面的 Prompt 微调教程：https://t.co/B3Rov95VTp ([View Tweet](https://twitter.com/tuturetom/status/1808323206492966979))
- > 其中 Prompt 微调也引入了 Auto Templating + 手动配置的技术，可以根据图谱的形态自动生成 Prompt Template：https://t.co/oD5gSFrs0n ([View Tweet](https://twitter.com/tuturetom/status/1808323616062558500))
- > GraphRAG 论文已经发布：https://t.co/IxYL3nIPhK 
  
  ![](https://pbs.twimg.com/media/GRh1B2xb0AI6sXg.jpg) ([View Tweet](https://twitter.com/tuturetom/status/1808323971051671660))
- > 微软还发布了一个解决方案加速器，如果你不想自己搞整套流程，可以使用 Azure 的解决方案：https://t.co/eaDUoZew2p ([View Tweet](https://twitter.com/tuturetom/status/1808324695722545504))
- > GraphRAG 的核心数据流如下：https://t.co/B5vy5bh3M9 
  
  ![](https://pbs.twimg.com/media/GRh1531b0AQnccs.jpg) ([View Tweet](https://twitter.com/tuturetom/status/1808324963201700196))
- > 关于如何进行 Global Search 和 Local Search，GraphRAG 也提供了两个 Notebook 实例：
  
  \- https://t.co/ShE5rn4RRz ([View Tweet](https://twitter.com/tuturetom/status/1808325144940798037))
- > GraphRAG 对于「全局问题」的优势分析：
  
  \- 朴素 RAG 只考虑 K 个最相似的文本块，并无依赖关系，甚至可能是表面相似的答案，导致问题
- > Global Search 原理：
  
  \- 按 LLM 窗口大小拆分社区总结
- > GraphRAG 在大规模播客数据、以及新闻数据上进行评估：
  
  \- 播客数据：8564 个节点，20691 条边
- > 关于全局问题探索，可以参考 https://t.co/BcnVWP0snq ([View Tweet](https://twitter.com/tuturetom/status/1808328774515392700))
- > GraphRAG 的局限：
  
  \- Index 和全局搜索等成本较高
- > 微软发布开源 GraphRAG 的原文：
  
  https://t.co/SFxmsETUNG ([View Tweet](https://twitter.com/tuturetom/status/1808329818536305088))