title:: 🗺️ How an AI Plans Code Changes Across an Entire GitHub Repository ️人工智能如何计划整个GitHub存储库中的代码更改
author:: [[sweep.dev]]
url:: https://docs.sweep.dev/blogs/ai-code-planning

![](https://docs.sweep.dev/og_image.png)

- > We've tried a lot of strategies to manipulate and improve the input context. We encountered an interesting issue everytime we added additional inputs. This will hurt our performance in the short term because of how GPT4 currently degrades in performance (both planning, coding, and instruction following) after ~20k tokens.  
  我们已经尝试了很多策略来操作和改进输入上下文。每次添加额外的输入时，我们都会遇到一个有趣的问题。这将在短期内损害我们的性能，因为GPT4目前在大约20k令牌之后的性能（规划，编码和指令遵循）会下降。
  
  This is a prevalent effect known as lost-in-the-middle (LIM) where LLMs only focus on the context at the beginning and end of the prompt. Thus, when we try to add more context, GPT4 will also ignore a lot of it. ([View Highlight](https://read.readwise.io/read/01j2xbxm2ayfxw5q4j7zcgpvdg))
- > We used the CST of python to extract all spans that define or mention a certain class, function, or class method AKA entity (we can just check if `entity in span`). This is not as accurate as static analysis tools like LSPs and type-checkers, but it's a 99% solution. After getting this we can prune each file to only what’s needed to make the code change to it. But how do we get the entities?
  
  We can use the above graph with the initial searched files as the root nodes.
  
  Now we have an initial node set, and we can expand this out by one degree. We clearly can’t fit all of it’s degree 2 neighbors in context, but the extracted entities can fit in context. Ideally the ones that lie within the extracted snippets will be mentioned, and we can prune a set that is guaranteed to have perfect recall.
  
  `Set with high recall and low precision -> an intelligent LLM workflow -> Set with high recall and high precision`
  
  The LLM is really good at finding the items that are truly relevant, and we need to have high recall and precision. Now we have all of this relevant context, and instructions to be passed to the execution (diff generation and validation). ([View Highlight](https://read.readwise.io/read/01j2xfaemb5gk28qmcd7z6mqcd))
	- 通过 AST 图空间结合 entity 检索来找到更加关键的 context。