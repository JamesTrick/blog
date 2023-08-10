---
title: What's next? Next word prediction with PyTorch
date: 2023-08-10T10:13:04Z
image: images/blog/prompt-header.jpg
author: James Malcolm
description: "Let's build a basic next word prediction model using PyTorch and discuss the applications."
tags: ["data", "llms", "pytorch"]
---

Today, I want to take you through a simple next-word prediction model built using [PyTorch](https://pytorch.org/). These models are famous in search applications, Gmail’s Smart Compose feature, which predicts the next words or sentences in emails you want to write.

![](/static/images/google.gif)

Google goes into detail on how they build their smart compose feature in their [research blog post here](https://ai.googleblog.com/2018/05/smart-compose-using-neural-networks-to.html). From this, I want to pull out some key learnings and requirements:
Latency. Latency is important, must generate a response in under 100ms.
They explored a Sequence to sequence style model (Seq2seq) but found it failed the latency test. Instead, they opted for a Recurrent neural network (RNN) model.

You may be wondering, this blog post came out in 2018 - why am I talking about it now?

The answer is twofold:
* Teach people the basics of an RNN model using PyTorch
* Show applications for use that could help mould user behaviour to get better performance of your models further down the track.

## PyTorch Model

There are many ways to tackle a next word prediction model. For this post, I want to build a bi-directional LSTM model. 

LSTM, long term short term memory layers are a type of RNN, that have a more complex structure - comprising of three gates:
* Input gate
* Output gate
* Forget gate
These three gates control how much information is allowed to enter, leave or be forgotten within the layer.

LSTMs are special because they can retain long-term memory and they help avoid the vanishing gradient problem that vanilla RNN layers can fall victim to. I really enjoyed [this post comparing the two layers in relatively simple terms](https://www.linkedin.com/advice/0/how-do-you-choose-between-rnn-lstm-natural-language).

Now, let's take a look at our model:

```python
import torch
import torch.nn as nn
import torch.optim as optim
import torch.nn.functional as F

import pytorch_lightning as pl

class BiLstm(pl.LightningModule):

  def __init__(self, total_words):
    super().__init__()
    self.emb = nn.Embedding(total_words, 150)
    self.lstm = nn.LSTM(150, 150, bidirectional=True, batch_first=True)
    self.dense = nn.Linear(150 * 2, total_words)

  def forward(self, x):
    x = self.emb(x)
    out, (hidden, cell) = self.lstm(x)
    cat = torch.cat((hidden[-2,:,:], hidden[-1,:,:]), dim = 1)
    return self.dense(cat)

  def training_step(self, train_batch, batch_idx):
    x, y = train_batch
    pred = self.forward(x)
    loss = F.cross_entropy(pred, y.long())
    self.log("training_loss", loss)
    return loss

  def configure_optimizers(self):
    optimizer = torch.optim.Adam(self.parameters(), lr=0.01)
    return optimizer
```

To help make training easier, I use [PyTorch Lightning](https://www.pytorchlightning.ai/). This package is wonderful and makes the training loop easier, and the ability to switch between GPUs and CPUs easier.

Let’s break the model down by its layers.

* Embedding layer. Embeddings have come to the forefront with the rise of generative AI. The role that they play here is compressing the potentially mammoth vocabulary size, down to a more manageable dimension of 150 dimensions.
  You can read more about embeddings [here](https://www.tensorflow.org/text/guide/word_embeddings#word_embeddings_2).
* The next layer is our LSTM layers. Note, we make it a bidirectional layer. Meaning the layer will ‘read’ our embeddings from left to right, and right to left. The output size of this is 150.
* Our final layer is our Linear, or fully connected layer. This layer is the ‘prediction’ layer. Note, the input is 150 * 2, this is because of our `bidirectional=True` statement on our LSTM layer. The output of this layer is simply the number of words we have in our vocabulary.

Finally, I want to get into the `predict` function. The code is presented below.

```python
def predict(x, min_words=5):
  seed_text = x
  x = tokenizer.texts_to_sequences([x])[0]
  x = pad_sequences([x], maxlen=max_sequence_len-1, padding='pre')
  x = torch.tensor(x)

  with torch.no_grad():
    word_count = 0

    for word in range(min_words):
      if word_count > 0:
        inputs = tokenizer.texts_to_sequences([seed_text])[0]
        x = pad_sequences([inputs], maxlen=max_sequence_len-1, padding='pre')
        x = torch.tensor(x)
      preds = model(x)
      preds = torch.argmax(preds, axis=1)

      output_word = list(tokenizer.word_index.keys())[list(tokenizer.word_index.values()).index(preds)]
      seed_text += " " + output_word
      word_count += 1
  return seed_text
```

In this function, we have a `seed_text`, this could be the start of the sentence. We have a min_words function, indicating that we always want a minimum of five words to be generated.

This works! We get an output!

But, there are ways to make it better. Perhaps the logical way would be to add a probability parameter, eg. Only predict the next word if its probability is greater than x%. We can do this from the output of the `model(x)` and pass that through a softmax (`F.softmax()`) function - turning the logits into a probability.

Adding in a probabilty parameter also brings us a step closer to the parameters we commonly see on LLMs.

The next logical move would be to incorporate and tokenize so that punctuation remains in the data. This will bring in the chance that the model predicts an end of a sentence, or question mark perhaps - bringing us a step closer to producing higher quality predictions.

## Applications

Although this is a form of supervised learning, I believe there are benefits to exploring these models outside of just learning.

Prediction models can help predict what to say next, encouraging users to give more details on customer support issues for example.

They can help guide customers to the right term to search within e-commerce or job boards.

Ultimately, being comparatively lightweight to generative models, they can have much lower latency and much lower cost while both increasing the customer experience and helping drive business metrics.