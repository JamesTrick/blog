---
title: "Self-Improving Prompts: A Critique Loop for LLM Agents"
date: 2026-07-01T00:55:55Z
summary: "When you cannot update model weights, the prompt is your main lever. Here is how to automate prompt improvement using a critique loop inspired by DSPy's GEPA optimizer."
tags: ["LLM", "prompt engineering", "DSPy", "agents", "machine learning"]
---

A common entrypoint when building LLM powered agents is prompt engineering. You write instructions, test them, tweak them, test again. The cycle repeats until the agent behaves the way you want. Then a new model comes out, or user-behaviour starts to drift from reality. Test. Ship. Break. Update. Repeat.

There has to be a better way.

What if the prompt could improve itself? What if, instead of manually tweaking instructions, we could automate the whole process and let an optimisation loop find better prompts for us?

This post explores exactly that idea. I will walk through how DSPy approaches automatic prompt optimisation, why I cannot use it directly for my PydanticAI agent, and how I adapted its core principles into a critique loop that works with my existing stack.

## The constraint

Here is the situation. I am building an agent that uses closed-source LLMs. This comes with limitations.

* I cannot update or perform post-training on the model weights.
* I cannot change the model architecture.
* I can only change the prompt, instructions, and model hyperparameters.
* Some of the agent logic lives in an external platform I cannot fully observe, so I do not have a complete evaluation picture.

This means the prompt is genuinely the only lever I have. If I want the agent to improve over time, I need a systematic way to improve that prompt.

This is not an unusual constraint. Many teams working with API-based models face the exact same problem. The model is a black box, and the prompt is the only interface.

## Enter DSPy

[DSPy](https://dspy.ai/) by Stanford reframes prompt engineering as a programmatic optimisation problem. Instead of hand-writing prompts, you define the structure of your task and DSPy automatically finds the best instructions, few-shot examples, and reasoning patterns for your LLM.

DSPy is built on three concepts.

### Signatures

At the core are **Signatures**, which are declarative specifications of what your LLM call should do. You define input and output fields as a Python class:

```python
class ClassifyQuestion(Signature):
    question: str = InputField()
    category: str = OutputField()
    confidence: float = OutputField()
```

DSPy turns this into a prompt automatically. The signature is the contract that every optimiser works against. It is a clean abstraction. You declare intent, not prompt text.

### Modules

DSPy wraps signatures in **Modules** that apply specific prompting strategies.

* `Predict` is a straightforward one-shot call.
* `ChainOfThought` injects a reasoning step before the final output.
* `ReAct` interleaves tool-use reasoning with action selection.
* `IfThen` provides conditional routing between branches.

So `ChainOfThought(ClassifyQuestion)` gives you chain-of-thought reasoning over that signature. The module is what gets optimised, not a prompt string.

### Optimisers

This is where DSPy becomes really interesting. DSPy provides algorithms that automatically improve your modules given a training set and a metric function. There are more than a dozen optimisers, including:

* **BootstrapFewShot** generates synthetic examples to include as few-shot demonstrations.
* **COPRO** generates candidate instructions and picks the best through coordinate ascent.
* **MIPROv2** uses Bayesian optimisation over instruction space, informed by your data.

But the optimiser that most directly inspired my approach is **GEPA**.

## GEPA: Genetic Evolution of Prompts via Adaptation

GEPA is DSPy's flagship optimiser. It is the one that got me thinking about how to apply these ideas outside of DSPy itself.

GEPA combines several ideas into one loop:

1. **Instruction mutation** takes your current prompt instructions and generates variants by rewriting, rephrasing, adding constraints, or removing redundancy.
2. **Evaluation** tests each variant against your training set using your metric function.
3. **Feedback-driven reflection** is where GEPA differs from simpler approaches. The metric does not just return a score. It returns text feedback alongside it. For example: `score=0.0, feedback="Don't reference the input season verbatim."`
4. **Instruction rewriting** uses a separate reflection LLM that reads the failures and feedback and produces improved instructions.
5. **Selection** picks the best-performing variant as the new base for the next iteration.

The critical insight is the feedback loop. Metrics return structured critiques, not just numbers. The reflection LLM reads those critiques and uses them to rewrite instructions. This is much more powerful than black-box search because the feedback guides *how* to improve, not just *which* variant won.

GEPA is production proven. Shopify migrated to a small Qwen model with GEPA-optimised prompts and got approximately 75 times cost reduction with 2 times better reliability. Dropbox doubled accuracy while processing 10 to 100 times more data at the same cost.

In code, it looks like this:

```python
reflection_lm = dspy.LM("openai/gpt-5.4")

optimizer = dspy.GEPA(
    metric=haiku_metric,       # your scoring function (returns score + feedback)
    reflection_lm=reflection_lm,  # the LLM that rewrites instructions
    auto="light",              # budget: light, medium, or heavy
    num_threads=2,             # parallelism for API rate limits
)

optimized_program = optimizer.compile(haiku_bot, trainset=train, valset=val)
```

## Why DSPy does not fit my stack

DSPy is excellent, but I cannot practically use it for my PydanticAI agent. Here is why.

**DSPy requires rewriting my agent as a DSPy program.** DSPy owns the full pipeline. Signatures, modules, tool calls, agent routing. My agent is built on PydanticAI, which has a fundamentally different architecture. PydanticAI uses Pydantic models for structured outputs, dependency injection for tools, and a different agent lifecycle. Porting to DSPy would mean rewriting the entire application, not just the prompt.

**PydanticAI is incompatible with DSPy modules.** DSPy modules generate their own prompts internally. PydanticAI agents have their own system prompt, tool schemas, and retry logic. DSPy optimisers expect to control prompt construction end-to-end, which PydanticAI does not expose.

**DSPy assumes a flat I/O contract.** DSPy signatures are clean input to output mappings. My agent is conversational with multi-turn state, tool calls, and side effects. DSPy metric functions expect a single prediction object, not an agent session with interleaved tool calls.

**I do not need to re-architect to get prompt optimisation.** The core insight from DSPy and GEPA, which is that you can automate prompt improvement through a critique loop, is framework agnostic. I can apply the principle without adopting the entire DSPy ecosystem.

## The critique loop

So I built my own version. I call it the critique loop, and it adopts the core principles of GEPA but applies them directly to my PydanticAI agent.

What I take from GEPA:

* The **training loop structure** (evaluate, score, optimise, repeat).
* **Feedback-driven improvement** where critiques guide rewriting, not just scores.
* A **separate optimiser LLM** that reads failures and produces improved prompts.
* **Metric design** as the most important lever. Garbage metric gives garbage optimisation.

What I adapt for my stack:

* I optimise the system prompt and instructions directly, not DSPy signatures.
* The agent under test is my real PydanticAI agent.
* Scoring works against golden test cases, not DSPy Example objects.

The general loop looks like this:

```python
for iteration in n_iters:
    agent = create_agent(prompt)

    for query in queries:
        output = agent.generate(query, prompt)
        loss = critique_agent(output, golden_set)

    optimised = optimiser_agent(total_loss, failures)
    prompt = optimised.prompt
```

This looks like a standard model training loop and also looks like a reinforcement learning pipeline, because in a sense it is. I am doing reinforcement learning on the prompt rather than on model weights.

I track prompt state explicitly:

```python
@dataclass
class PromptState:
    """Store the current prompt and instructions."""
    system_prompt: str
    instructions: list[str]
    version: int = 0
```

## Designing the scoring function

Like any training process, the scoring mechanism is the most critical design decision. The optimiser will do exactly what you tell it to do, and if your metric is wrong, it will optimise toward something useless.

For my agent, I have some known negatives. The agent should not produce Markdown. It should not suggest contacting support. Beyond that, I want to encourage concise, considered responses rather than overly long ones, so I add a small penalty for prompt length.

The scoring function returns both a score and feedback text, mirroring what GEPA does. The feedback is what makes the difference. A simple binary pass or fail gives the optimiser very little to work with. Text feedback tells it what went wrong and how to fix it.

## What to watch out for

There are real risks with this approach, and they are worth stating up front.

### Overfitting to your test set

The optimiser will make the prompt extremely good on your test cases and potentially worse on novel edge cases. You need a held-out validation split that is never used for optimisation. Only use it for evaluation. If validation score drops while training score rises, you have overfit.

### The blind spot problem

If part of your agent logic lives in a system you cannot control or observe, the optimiser may waste effort trying to fix failures it cannot actually resolve. You need to classify failures as fixable via prompt versus requiring changes elsewhere before feeding them to the optimiser.

### Prompt bloat

Without length pressure, self-improving prompts tend to get longer over iterations. The optimiser discovers that adding more instructions rarely hurts test scores, so it keeps adding. An explicit token budget or length penalty helps keep prompts lean.

## Early results

Even with a single iteration, I see quite drastic prompt changes. The optimiser restructures instructions, adds missing constraints, and removes redundancy. I plan to run this over a larger case set to see how the prompt evolves across multiple iterations.

The long-term goal is interesting. I want the agent to develop recognition for cases where it needs more information from the user. Requests for screenshots, organisation access, or additional context should emerge naturally from the optimisation process rather than being manually specified.

## Where this is going

The next step is making this a repeatable process. I want prompt optimisation to become a regular part of the development workflow, not a one-off experiment. Run it weekly against an expanding golden set, track version history, and let the prompt compound improvements over time.

The idea is simple. Instead of manually tweaking prompts whenever behaviour degrades, automate the improvement loop and let the system maintain itself. Then move on to other, more fun things.
