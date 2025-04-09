title:: 📝 Large Language Models Are Bad at Refactoring Code
author:: [[sweep.dev]]
url:: https://docs.sweep.dev/blogs/refactor-python

![](https://docs.sweep.dev/og_image.png)

- > One trick we found that helps with this is to add "anchors" to the chain-of-thought(CoT) prompting. Instead of the classic `"Think step-by-step to solve the problem"`, our CoT prompt looks like this:
  
    <contextual_request_analysis>
    Analyze the user request and outline the first and last few lines of code that should be extracted.
    </contextual_request_analysis>
  
  and then our GPT4 response will contain this line:
  
    This section starts with the line `documents = []` and ends with the line `ids.append(f"{gh_file_path}:{snippet.start}:{snippet.end}")`
  
  This helps *a lot*. Our hypothesis is that when starting to extract the span, the transformer(underlying model architecture of GPT4) attends to and anchors the generation to the start `documents = []`, and then does the same for the end `ids.append(f"{gh_file_path}:{snippet.start}:{snippet.end}")`. ([View Highlight](https://read.readwise.io/read/01j2xbrywm79ae9cnmr12445bk))
	- 使用tag作为指令的锚点，Prompt 的效果会比较好。