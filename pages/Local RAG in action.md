- > 个人的RAG 应用开发实录
- #RAG #AI
- [参考 Langchain RAG quickstart](https://python.langchain.com/docs/use_cases/question_answering/quickstart#indexing-store)
- 使用本地LLM：llama2-chat
	- llama2量化的过程记录
	- TODO Llama2 量化后的 max_token和temperature 怎么设置，Langchain 中如何初始化？
- ollama 作为 serving 框架如何使用
- 三方库的安装问题
	- split 阶段有很多 parse
-
- index 阶段
	- `split`将加载的文本内容按照一定的长度进行切割。
	- 需要 split 的原因有两个：
		- 主要是因为 LLM 单次会话有长度限制（context window），无法将全量文本内容传递给 LLM。
		- LLM 在 long context 场景中 Reasoning 能力的表现不太好。
	- #+BEGIN_NOTE
	  split 的长度如何确定是一个需要思考的问题。它最终如何影响搜索的效果？
	  #+END_NOTE
- retrieval 阶段
	- 可以使用 similarity 进行搜索，在 vector 空间中进行匹配。
	- #+BEGIN_CAUTION
	  这里可以使用多种的算法进行匹配，实际需要结合场景以及数据针对性做优化。选取不同的算法。
	  #+END_CAUTION
	- 首先通过 embedding model，将知识库内容生成一系列的 vector array。再将 vector 存储到向量数据库中。然后将 query 通过相同的 embedding model 转换为 vector。最后通过算法在向量数据库中查找到最相近的知识库内容。那么：**余弦近似值的算法与 embedding model是否存在关系？**
	-