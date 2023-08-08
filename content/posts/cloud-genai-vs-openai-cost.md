---
title: Counting Pennies - Deploy or buy GenAI?
date: 2023-08-07T02:13:04Z
image: images/blog/prompt-header.jpg
author: James Malcolm
description: "In this post, we take a look at the cost of deploying your generative AI on AWS and Google cloud, and compare it to that of OpenAI's pricing model"
tags: ["data", "llms", "generative ai"]
---

In this post, we explore the cost of deploying or buying your generative AI. Specifically, I want to focus on the computing cost - not the additional costs which contribute to the total cost of ownership.

In this, I want to explore four options, these are:

1. Managed: Use OpenAI directly
2. Self-managed: Deploy using AWS
3. Self-managed: Deploy using Google Cloud

## Assumptions

For the best possible comparison, we compare OpenAI and Cohere to deploying and managing LLaMA 2 - perhaps the best competing commercially usable open-source model out there.

## AWS vs Google Cloud

To begin the analysis we first need to understand what we’re dealing with in terms of cost between the cloud providers.

### AWS

For the cost of this, we draw from [Phillip Schmid’s blog post](https://www.philschmid.de/sagemaker-llama-llm) which suggests you need the [p4d.24xlarge instance](https://aws.amazon.com/ec2/instance-types/p4/). For this - you’ll need to increase your service quotas. This instance boasts:
* 98 vCPUs
* 1152 GB of RAM
* 8 NVIDIA A100 GPUs
All this compute comes at a sizeable cost, running USD 32.773 per hour, or USD 23,923 per month within the us-east-1 region.

The typical deployment of LLaMA on AWS would be using the [Sagemaker](https://aws.amazon.com/sagemaker/) and the Hugging Face DLC. Note, however, spot instances cannot be used for endpoints within Sagemaker. So a containerised service like ECS could help in bringing down the costs.

### Google Cloud

Google Cloud also offers the Nvidia A100 instances, and the most similar instance to run LLaMA is the [a2-ultragpu-8g](https://cloud.google.com/compute/docs/gpus#a100-80gb). This instance packs:

* 96 vCPUs
* 1360 GB of RAM
* 8 NVIDIA A100 GPUs
Similar to AWS, perhaps the easiest way to deploy 
In this case, Google Cloud is more expensive than AWS running at USD 40.55 per hour, or USD 29,601.78 per month within the us-central1 region.

Like AWS Sagemaker, [Vertex AI](https://cloud.google.com/vertex-ai) is an option to deploy your model, it has a very similar deployment pattern to that of Sagemaker. Unforetuenly, like Sagemaker, endpoints don’t support preemptible (spot) instances. Using a service such as Google Kubernetes Engine (GKE) would support GPUs, preemptible instances and scale to zero if desired.

## OpenAI

Although we look at [OpenAI](https://openai.com/), OpenAI’s pricing style has been adopted by most of the vendors selling direct, API access to their models.

OpenAI charges by token. OpenAI says, [“Tokens are pieces of words used for natural language processing. For English text, 1 token is approximately 4 characters or 0.75 words.”](https://openai.com/pricing). Alternatives to OpenAI such as Cohere, Google Cloud’s PaLM API (not to be confused with deploying your own), all charge token-based.

OpenAI charges for both input tokens and the output tokens of an interaction. Currently, as of 7 August, the price sits at $0.03 / 1K input tokens, and $0.06 / 1K output tokens for their GPT4 model.

## Tallying the cost

Now, let’s bring this all together. Since we are exploring different billing models, we need to make some assumptions:
The average message or interaction contains words, or 200 tokens and an additional 100 tokens are used for prompting. So, 300 tokens for the input. Let’s say, you also get 300 tokens back as output tokens. Giving us a cost per message of: $0.03.
The length of the message is largely irrelevant for the cloud deployments - and this is what helps them become economical. But, as the number of queries increases - the more demand you’ll have for multiple instances. Ensure you can scale down easily as well! For this, I didn't explore the point at which you may need scale  to additional instances.

| # of Exchanges | OpenAI (Total Cost) | AWS (Average Cost Per) | Google Cloud (Average Cost Per) |
|---------------:|---------------------|------------------------|---------------------------------|
|       	5000 |          	$90.00 |              	$4.78 |                       	$5.92 |
|     	50,000 |         	$900.00 |              	$0.48 |                       	$0.59 |
|    	100,000 |       	$1,800.00 |              	$0.24 |                       	$0.30 |
|    	150,000 |       	$2,700.00 |              	$0.16 |                       	$0.20 |
|    	200,000 |       	$3,600.00 |              	$0.12 |                       	$0.15 |
|    	250,000 |       	$4,500.00 |              	$0.10 |                       	$0.12 |


As we can see from the table above, OpenAI wins in quite a nice fashion. So, it begs the question of why would you want to deploy your LLM.

Control being the big one, I refer to this as being able to control where your data goes to, but also having little ‘model-lockin’, and having the ability to switch out models to less resource-intensive ones.

For workloads that are token-heavy, the cloud does start to make more sense.

These can either be workloads where input lengths are long, a lot of context is given or perhaps conversational history. Generally, workloads that have a lot of tokens help lower the break-even point of the cloud deployment. 

In this situation, you’d be paying more to OpenAI or any token-based pricing provider. Whereas you pay for the compute, not the tokens with a cloud provider, so the break-even point becomes a lot lower.

And of course, there are ways to make compute cheaper, either through spot instances (although requiring a different service to Vertex AI and Sagemaker) or savings plans.

## Moore’s Law: Where will the cost go?

![](/static/images/moore-law.jpg)

It wouldn’t be a cost of technology post without mentioning Moore’s law. Moore’s law is an observation that computing power doubles every two years, whereas the cost is halved over that time frame.

The trouble today is that computing demands to run the latest and greatest LLMs outstrip supply. As we've seen with the LLaMA model you need one of AWS's largest models to run. Where a couple of years ago you could get away with running game-changing [BERT](https://www.techtarget.com/searchenterpriseai/definition/BERT-language-model) on a much smaller instance.

Although the cost of these GPUs will likely come down over the next few years, I believe the cost of keeping up with the latest algorithms will keep increasing for the foreseeable future.

## Conclusion

Many factors should influence the buy vs build decision of LLMs. Today, we’ve only explored one of them - cost. In future posts, I’ll dig a little deeper into other considerations you may face.