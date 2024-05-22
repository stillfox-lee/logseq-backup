title:: Here Is a (Brief) Taxono...
author:: [[@cwolferesearch on Twitter]]
url:: https://twitter.com/cwolferesearch/status/1787553640703520832

![](https://pbs.twimg.com/profile_images/1715212547215802368/tqxfSqh3.jpg)

- > Here is a (brief) taxonomy of the three advanced prompt engineering techniques that are most commonly used/referenced…
  
  Disclaimer: Basic prompting techniques (e.g., zero/few-shot or instruction prompting) are highly effective, but sometimes more complex prompts can be useful for solving difficult problems (e.g., math/coding or multi-step reasoning problems). Because LLMs naturally struggle with these problems (reasoning capabilities do not improve monotonically with model scale) most research on prompt engineering is focused upon improving reasoning and complex problem solving capabilities. Simple prompts will work for solving most other problems.
  
  Chain of Thought (CoT) prompting [1] elicits reasoning capabilities in LLMs by inserting a chain of thought (i.e., a series of intermediate reasoning steps) into exemplars within a model’s prompt. By augmenting each exemplar with a chain of thought, the model learns (via in-context learning) to generate a similar chain of thought prior to outputting an answer. We see in [1] that explicitly explaining the underlying reasoning process for solving a problem actually makes the model more effective at reasoning.
  
  CoT variants: Due to the effectiveness and popularity of CoT prompting, several extensions have been proposed:
  
  \- Zero-Shot CoT [2]: eliminates few-shot exemplars and instead encourages the model to generate a problem-solving rationale by appending the words “Let’s think step by step.” to the end of the prompt.
- > Here are links to all of the papers for the techniques referenced above!
  
  [1] https://t.co/v45j0xwfgz
  [2] https://t.co/NPMzb6f8OE
  [3] https://t.co/R1rJj9mCaM
  [4] https://t.co/7epPyigq6v
  [5] https://t.co/7Jo9L0TesK 
  [6] https://t.co/qVvVLikRmY
  [7] https://t.co/yfnlqOyMan ([View Tweet](https://twitter.com/cwolferesearch/status/1787554287263871086))
- > For more details on these advanced prompting techniques and more recent research on prompt engineering, check out my latest overview! It covers everything from basic concepts for LLMs to a taxonomy of recent research topics! 
  
  ![](https://pbs.twimg.com/media/GM6rdoiW8AAZR5s.jpg) ([View Tweet](https://twitter.com/cwolferesearch/status/1787554635818930308))