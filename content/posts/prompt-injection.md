---
title: LLM Risks - Prompt Injection
date: 2023-06-15T02:13:04Z
image: images/blog/prompt-header.jpg
author: James Malcolm
description: "With the rise of generative AI models, new and emerging security risks. One of the largest and novel risks is Prompt Injection. We take a look a couple of mitigations on how to make AI safer."
tags: ["data", "llms", "generative ai"]
---

Generative AI models are all the rage nowadays. For data people, generative models have been around for several years, but the power and usability of products such as ChatGPT have taken the world by storm.

This emergence has brought in new and emerging security risks with it. One of the largest and most novel risks is prompt injection. Prompt injection attacks can affect all large language and generative AI models.

Although new, prompt injection can be likened to SQL Injection – a well-known cyber-attack exploiting the use of forms on websites.

## What is a prompt?


[Prompts are a set of instructions inputted into a generative model to help obtain a desired result.](https://news.microsoft.com/source/features/ai/the-art-of-the-prompt-how-to-get-the-best-out-of-generative-ai/)

Traditional deep learning models are trained using labelled datasets. This, in a way, is guiding the model to get the desired output. Generative models, on the other hand, are trained to perform multiple tasks, as such take in prompts by users to guide the model to gain a desired response.

An example prompt may look like the following:

*“Classify the text into neutral, negative or positive.
Text: I think the food was okay.
Sentiment:”*

Or for a different prompt that is a conversational bot, you may use:

*“The following is a conversation with an AI research assistant. The assistant tone is technical and scientific.”
Human: Hello, who are you?
AI: Greetings! I am an AI research assistant. How can I help you today?
Human: Can you tell me about the creation of black holes?
AI:….”*

Prompts are incredibly powerful and for a lot of use-cases, they completely remove the need to fine-tune a LLM which saves on upfront cost. The accessibility of prompts is part of the reason why generative AI models are becoming so widely popular. The accessibility of prompts is also why anyone with a laptop could be a bad actor waiting to exploit your models.

So now that we understand what a prompt is, what is a prompt injection attack?
[OWASP defines a prompt injection attack as](https://owasp.org/www-project-top-10-for-large-language-model-applications/descriptions/Prompt_Injection.html), “using carefully crafted prompts that make the model ignore previous instructions or perform unintended actions.”

[Unlike traditional SQL injections it’s widely agreed that there is no solid way to stop prompt injection](https://simonwillison.net/2022/Sep/16/prompt-injection-solutions/). Instead, there are a series of good practice mitigations that one can consider.

### Filter the user input

For most (all) LLM use cases, the prompt itself won’t be revealed to the user. Companies can mitigate against prompt injection by filtering the user input before it is passed to the LLM. This can include:

* Limiting the character count to a sensible amount. For example, if the LLM is used for search, limit it to a couple of words to ensure a prompt injection can’t fit.
* Filter user input for common prompt injection instructions.
Not only can this help with ensuring safety – it’s a good chance to sanitise the user input for PII or any other sensitive information.

This second point could even be extended by yet another AI solution of identifying prompts. There is a [Hugging Face dataset](https://huggingface.co/datasets/deepset/prompt-injections) containing prompt injections seen and mundane inputs.

### Monitor and log

This may sound like a given, but all interactions with the LLM should be logged and monitored. Companies can go a step further than this and assign a unique ID to the user, to help in identifying bad actors.

This monitoring and logging will help identify and ban bad actors but also provide valuable data on how people may be exploiting your models and insight into how to prevent it from happening.

### Assess the risk

A lot of the prompt injections seen to date have been moderately harmless and trivial. So it’s important to gauge the risk of the application you’re running with.

An example of a higher-risk application without the measures given would be customer support. Let’s say, the bad actor inserts a prompt to automatically offer a discount where one wouldn’t be valid.

Without the proper logging involved, the company would be none the wiser that this was a prompt injection attack, and feel obliged to honour the discount.

It’s easy to see in situations like this, logging is important - but also that it’s the last line of defence. Hopefully, the steps earlier in the post prevent this attack from even taking place.

As with everything security-related, measures need to be considered and implemented at each and every level. Prompt injection is an emerging issue and one that is here to stay as companies grapple with the best ways to deal with it.
