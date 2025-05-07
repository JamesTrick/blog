---
title: Learning and applying Deepseek techniques
date: 2025-02-01T02:13:04Z
image: images/blog/deepseek-header.avif
author: James Malcolm
description: Explore what makes Deepseek unique and how to apply the same techniques to smaller open source models such as Google's Gemma.
tags:
  - data
  - llms
  - generative
  - ai
---

In January 2025, Deepseek made headlines with the release of their Deepseek R1 models and a suite of smaller models distilled from the larger R1 variant. The announcement sent shockwaves through the market—shaking NASDAQ and causing NVIDIA shares to drop nearly 20% in a single day. Although the performance of these models wasn’t the only factor, Deepseek’s innovation called into question the competitive advantage long held by US-based AI giants.

## The Motivation

One of the key innovations introduced by Deepseek was the novel order in which they train their models. Traditionally, the training process involves three main steps:

1. **Pre-training:**  
    The model is trained on vast amounts of text data to predict the next word in a sequence. (For more details, check out [my post on building a similar model with PyTorch](posts/next-word-prediction-pytorch/).)
    
2. **Supervised Fine-Tuning:**  
    After pre-training, the model undergoes fine-tuning with supervised data. This step typically transforms the base model into an instruct or chat-based model.
    
3. **Preference Optimization:**  
    Finally, a form of reinforcement learning—often combined with human feedback (RLHF)—is applied to further align the model’s outputs with user expectations.
    

Deepseek flipped this script by applying reinforcement learning directly after pre-training. They introduced a novel reinforcement learning technique called **GRPO** (detailed in their [DeepseekMath paper, February 2024](https://arxiv.org/abs/2402.03300)). Unlike the traditional Proximal Policy Optimization (PPO) approach—which requires a separate 'critic' model—GRPO averages outputs directly from the actor model. This self-supervising mechanism eliminates the need for a reference or value model and bypasses the requirement for supervised data.

One of the most fascinating discoveries with the R1-Zero model was its emerging ability to reason. During reinforcement learning, one of the reward functions incentivized the model to include reasoning segments by rewarding outputs that contained text wrapped in `<thinking> ... </thinking>` tags.

Although Deepseek released an entire suite of models (including distilled variants), it’s important to note that the smaller distilled models did not undergo the same reinforcement learning as the larger R1-based models. Instead, these models were fine-tuned on samples from the R1-Zero model using supervised techniques. As a result, they only mimic the appearance of the unique `<thinking>` phase without fully developing the underlying reasoning capabilities.

In this post, we’re going to replicate the Deepseek R1 approach: applying reinforcement learning techniques similar to GRPO on smaller, more manageable LLMs.

## The Approach

### Understanding Reward Functions

The original Deepseek paper introduces two key reward functions:

- **Format Rewards:**  
    This reward assesses whether the LLM’s response contains the proper tags (e.g., `<thinking> </thinking>` and `<answer> </answer>`). Depending on your specific use case, you might tweak or expand these rules. For instance, some approaches—like MLX's—reward the presence of a single tag even if its corresponding closing tag is missing, though to a lesser extent.
    
- **Accuracy Rewards:**  
    This reward determines if the answer is verifiably correct. In a math problem, for example, the solution process can be verified even if the final answer isn’t entirely correct. This mirrors the educational approach in high school mathematics, where students earn marks for demonstrating a sound problem-solving process, regardless of the final answer’s accuracy.
    

The interplay between these two reward functions is crucial. They teach the model not just to provide an answer but to reason through a problem, ensuring that its internal thought process is both logical and accurate.

### Leveraging GRPOTrainer from HuggingFace

The HuggingFace team quickly embraced Deepseek’s innovation by implementing a `GRPOTrainer` in their `trl` library. The trainer allows you to combine multiple reward functions. For example:

python

CopyEdit

`trainer = GRPOTrainer(     reward_funcs=[accuracy_func, format_func, length_func, ...],     # additional parameters )`

A few important notes regarding the reward functions:

- **Input Format:**  
    Reward functions accept prompts, completions, and any extra variables. If your dataset contains additional columns, be sure to handle them via the `**kwargs` parameter.
    
- **Dataset Formats:**  
    The expected format of prompts and completions will vary depending on your dataset. For datasets in the [standard format](https://huggingface.co/docs/trl/main/en/dataset_formats#standard), these will be lists of strings. In a [conversational format](https://huggingface.co/docs/trl/main/en/dataset_formats#conversational), they will be lists of message dictionaries. More details can be found in the [HuggingFace documentation](https://huggingface.co/docs/trl/main/en/grpo_trainer#using-a-custom-reward-function).
    

## Let’s Train

With the theoretical foundation in place, the next step is to apply these Deepseek techniques to real data. In the following sections, we’ll walk through the practical steps to set up your environment, define custom reward functions, and initiate training using GRPOTrainer.

Stay tuned as we dive deeper into the implementation details and share insights from our experiments.

```python
import re  
import logging  
import os  
  
from trl import GRPOConfig, GRPOTrainer  
from transformers import TrainingArguments  
from peft import LoraConfig  
from datasets import load_dataset  
  
logger = logging.getLogger(__name__)    
  
MODEL_ID = "google/gemma-2-2b-it"  
  
  
#  Extracted from Deepseek paper  
SYSTEM_PROMPT = """  
<start_of_turn>user  
A conversation between User and Assistant. The user asks a question, and the Assistant solves it.  
The assistant first thinks about the reasoning process in the mind and then provides the user  
with the answer. The reasoning process and answer are enclosed within <think> </think> and  
<answer> </answer> tags, respectively, i.e., <think> reasoning process here </think>  
<answer> answer here </answer>. User: {}.  
<end_of_turn>  
<start_of_turn>model:  
"""  
  
  
def format_rewards(prompts, completions, **kwargs):  
    """We want to ensure that the model is thinking, and returns the answer within the <answer> tags."""  
    pattern = r"^<think>.*?</think><answer>.*?</answer>$"  
    # completion_contents = [completion[0]["content"] for completion in completions]  
    matches = [re.match(pattern, content.strip()) for content in completions]  
    return [1.0 if match else 0.0 for match in matches]  
  
  
def accuracy_rewards(prompts, completions, ground_truth, **kwargs):  
    """Is the response correct/accurate"""  
    rewards = list()  
    for response, truth in zip(completions, ground_truth):  
        if response == truth:  
            rewards.append(1.0)  
        else:  
            rewards.append(0.0)  
    return rewards  
  
  
def make_conversation(sample):  
    return {  
        'prompt': SYSTEM_PROMPT.format(sample['Question'])  
    }  
  
  
def prepare_dataset(dataset_id='FreedomIntelligence/medical-o1-reasoning-SFT', test_split=1):  
    dataset = load_dataset(dataset_id, 'en')  
    logger.info("Dataset loaded")  
    dataset = dataset['train']  
    dataset = dataset.map(make_conversation)  
    dataset = dataset.rename_column("Response", "ground_truth")  
    dataset = dataset.remove_columns(["Complex_CoT", "Question"])  
    return dataset  
  
  
def main():  
    dataset = prepare_dataset()  
    dataset = dataset.train_test_split(test_size=0.2)  
    logger.info("Dataset prepared")  
  
    reward_funcs = [  
        accuracy_rewards,  
        format_rewards,  
    ]  
    logger.info("Starting training")  
  
    training_args = GRPOConfig(  
        output_dir="./grpo",  
        per_device_train_batch_size=1,  
        gradient_accumulation_steps=4,  
        num_train_epochs=1,  
        learning_rate=1e-5,  
        logging_steps=1,  
    )  
  
    trainer = GRPOTrainer(  
        model=MODEL_ID,  
        reward_funcs=reward_funcs,  
        train_dataset=dataset['train'],  
        eval_dataset=dataset['test'],  
        peft_config=LoraConfig(task_type="CAUSAL_LM"),  
        args=training_args  
    )  
  
    trainer.train()  
  
  
if __name__ == "__main__":  
    main()

```