title:: Building Performant RAG Applications for Production#
author:: [[llamaindex.ai]]
url:: https://docs.llamaindex.ai/en/stable/optimizing/production_rag/

![](https://readwise-assets.s3.amazonaws.com/static/images/article4.6bc1851654a0.png)

- > General Techniques for Building Production-Grade RAG[#](https://docs.llamaindex.ai/en/stable/optimizing/production_rag/#general-techniques-for-building-production-grade-rag)
  
  Here are some top Considerations for Building Production-Grade RAG
  
  •   Decoupling chunks used for retrieval vs. chunks used for synthesis
  •   Structured Retrieval for Larger Document Sets
  •   Dynamically Retrieve Chunks Depending on your Task
  •   Optimize context embeddings
  
  We discussed this and more during our [Production RAG Webinar](https://www.youtube.com/watch?v=Zj5RCweUHIk). Check out [this Tweet thread](https://twitter.com/jerryjliu0/status/1692931028963221929?s=20) for more synthesized details. ([View Highlight](https://read.readwise.io/read/01hw7ntq9nde8k3w3ynznmn7qk))
- > A key technique for better retrieval is to decouple chunks used for retrieval with those that are used for synthesis. ([View Highlight](https://read.readwise.io/read/01hw9hza303acetpar8zrcpgv1))
	- 将用于向量查找的chunk和最后给 LLM 用于生成的 chunk 解耦。
- > The optimal chunk representation for retrieval might be different than the optimal consideration used for synthesis. For instance, a raw text chunk may contain needed details for the LLM to synthesize a more detailed answer given a query. However, it may contain filler words/info that may bias the embedding representation, or it may lack global context and not be retrieved at all when a relevant query comes in. ([View Highlight](https://read.readwise.io/read/01hw9jhm580nqh7fzm46v5va55))
- > This can help retrieve relevant documents at a high-level before retrieving chunks vs. retrieving chunks directly (that might be in irrelevant documents). ([View Highlight](https://read.readwise.io/read/01hw9n0g8xqcydghkaw4qgfqen))
	- 通过嵌入文档的摘要，并将其链接到与文档相关的块，可以在高层次上先检索到相关文档，然后再检索块，而不是直接检索可能位于不相关文档中的块。