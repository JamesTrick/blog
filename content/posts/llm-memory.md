+++
date = 2023-10-24T07:00:00Z
description = "Learn about the role of memory in LLM applications and how to handle multiple users and sessions for chat applications."
hasMath = false
slug = "llm-memory"
tags = ["data", "llms", "generative ai"]
title = "Handling multiple interactions with Langchain"
+++

![](/static/images/llm-memory.jpg)

There are many tutorials on getting started with Langchain and LLMs to create simple chat applications.

I want to go slightly beyond this post and go into a bit of detail on the role of memory has in chat applications, and lastly touch on how you can scale your application across multiple sessions and multiple users. 

---
## What is Langchain?

[Langchain](https://www.langchain.com/) is an open-source python package that helps in creating LLM solutions. It has four main modules:

* Models. Models allow for easy swapping in and out from a wide range of models, from [OpenAI](https://openai.com/gpt-4), [AWS Bedrock](https://aws.amazon.com/bedrock/) to open source models such as [LLaMA 2](https://huggingface.co/blog/llama2).
* Retrieval. Retrieval focuses on vectorstore wrappers from vector databases such as Pinecone, as well as handling text-embedding - all steps needed for Retrieval Augmented Generation (RAG)
* Memory. Memory is the large focus of this post, Langchain has multiple wrappers to make storage of, and use of memory easier.
* Chains. Chains as hinted by the name, bring all the parts above togerther into a chain.
---

## Why is memory needed?

Interactions with LLMs are stateless.

That is, out of the box, the previous interactions that a user has have no influence on current or future interactions. For example, take this interaction with a stock-standard LLM.

> *User:* Hi, my name is James.    
> **AI:** Hi James, nice to meet you.    
> *User*: What is my name?    
> **AI:** I’m sorry, I don’t know the answer to this.

You’ll note that although in the first message, I state my name is James. In the second interaction, we ask the LLM what my name is. The bot simply doesn’t answer. This is because the LLM is stateless, i.e. it has no memory of its own.

### Manually adding memory

A method to simulate memory is by modifying the prompt. In a general sense, your prompt would like like:

```
You are a helpful travel assistant helping customers book flights. Using the following conversation history, answer the following question.
{history}

Question:
{question}
```

The exact wording of your prompt will depend on the underlying LLM model you choose. For example, Claude would benefit greatly by using something along the lines of:

```
Human: You are a helpful travel assistant helping customers book flights. Use the following conversation to help answer the latest question:

Human: Hi, I’m James.

Assistant: Hi James, nice to meet you. How can I help you?

Human: What is my name?

Assistant:
```

Note, the Human: Assistant: formatting? This is because Claude specifically is trained using reinforcement learning with human feedback, and this is the formatting they used for their data. To get best results, Claude themselves [suggest this formatting](https://docs.anthropic.com/claude/docs/introduction-to-prompt-design):

---
### This post is part of my wider LLM series.

* [Counting Pennies - Deploy or buy GenAI?](/posts/cloud-genai-vs-openai-cost/)
* [LLM Risks - Prompt Injection](/posts/prompt-injection/)

Or a full list of posts, [available here](https://jamesmalcolm.me/tags/llms/).

---
## This is all quite… manual. How do we make it easier?

Now that we understand the basic idea behind adding memory, now is a good chance to introduce Langchain and the benefit of langchain for memory.

Langchain, thankfully as a solution for memory, various solutions in fact. These solutions sit within the Memory section of langchain and include:

* ConversationBuffer
* ConversationBufferWindow
* ConversionSummary

And a full list is available [here](https://python.langchain.com/docs/modules/memory/types/). But what do these classes actually do?

Let’s start with the most simple, ConversationBuffer. ConversationBuffer allows you to store messages in a dictionary format, which are then passed into the prompt. This is extremely similar to the example above.

### Cost increases with the input size

Nearly all LLMs I know of charge by input token and output tokens used. Note, that the input tokens include your customers message/input but also your prompt.

Since the current solution for LLM memory is injecting the history into a prompt, we can expect (rapidly) increasing costs.

To help minimise costs, the next two Langchain Memory classes come into play.

ConversationBufferWindow is similar to ConversaionBuffer aside from it only keeps the last `k` interactions in memory. This is helpful if old messages are no longer helpful, but can lead to quite abrupt memory loss.

ConversationSummaryMemory works by using a LLM to summarise the history. This can lead to a lower input token cost due to a more condensed history, but naturally you are likely charged for the summary.

With both these additional solutions, you are losing detail of previous interactions that could lead to lower quality future responses. On the flip side, you are working to increase latency and decrease cost. Experimentation is key here on what works best for your solution.

## What about multiple users, or across multiple sessions?

Now that we have an understanding of the role of memory and how it passed into a prompt, we now want to consider how to handle multiple users or multiple sessions for a user.

I think at this stage, people start to think of VectorDBs and retrieving content, similar to a RAG architecture.

But, I want to steer you more towards a traditional software pattern. Langchain aside for a bit, remember the basis of the prompt we need complete:

```
You are a helpful travel assistant helping customers book flights. Using the following conversation history, answer the following question.
{history}

Question:
{question}
```

Our goal here is to populate the `history` variable. To do this across different time periods and different users, we need a persistent database to store the chat history with a unique identifier for the user, and ideally the session.

This way, you can retrieve the conversational history that a user has from your database of choice. Once we have this knowledge, you can then inject this into the prompt template either manually, or using langchain.

By using a persistent store, it enables developers to deploy chat applications in serverless environments such as AWS Lambdas, whilst maintaining a natural conversational flow. Or, it enables chat applications to become highly personalised, by remembering previous interactions users have had.