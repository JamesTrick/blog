---
title: What's next? Next word prediction with PyTorch
date: 2023-11-08T00:13:04Z
image: images/blog/prompt-header.jpg
author: James Malcolm
description: Let's build a basic next word prediction model using PyTorch and discuss the applications.
tags:
  - data
  - llms
  - pytorch
---
Today, I want to take you through a simple next-word prediction model built using [PyTorch](https://pytorch.org/). These next word prediction models are a form of language modelling, and the knowledge learnt here forms the basis for larger large language models.

Specifically, we draw on research published by Google for Gmail’s Smart Compose feature, which predicts the next words or sentences in emails you want to write.

![](/static/images/google.gif)

Google goes into detail on how they build their smart compose feature in their [research blog post here][1]. From this, I want to pull out some key learnings and requirements:

* Latency. Latency is important, must generate a response in under 100ms.
* They explored a Sequence to sequence style model (Seq2seq) but found it failed the latency test. Instead, they opted for a Recurrent neural network (RNN) model.

You may be wondering, this blog post came out in 2018 - why am I talking about it now? Yes, almost certainly Google has improved their Smart Compose model and evolved from this architecture. Google also hint at that within their blog when they say ""

But, I really enjoy the application of RNN and the utility of the Smart Compose model. On top of this, I believe understanding RNNs and having the ability to build these models will put anyone on a good footing for more advanced models such as [LLMs](/posts/llm-memory), which I explore in other posts.

## Look into RNNs

[Recurrent neural networks (RNN)][2] are class of neural network that allow for previous outputs to be used as inputs. That is to say as the sequence is inputs is passed through the model, the weights of previous inputs are used as inputs also.

This ability to remember previous imagine is akin to how we humans read. We read in a single directional, typically, and we recall what previous words have been used to understand the meaning of a sentence, then paragraph.

Traditional networks, don't do this. Once the are scan a given input they move on, forgetting all that the previous inputs.

RNN's ability to use previous inputs naturally makes RNN models suited for sequential data such as text, time-series, audio, etc. When compared to say, convolutional layers the RNN's ability to 'remember' information is what makes it valuable.

One type of RNN architectures is the LSTM. LSTM stands for: Long Short Term Memory. As the name the suggests, LSTMs are capable of learning [long term dependencies][3],

LSTMs are seen as an improvement over vanilla RNN layers due to them being designed to overcome vanishing gradient problem - a problem you may while training an LLM model.

Both, LSTM and Gated Recurrent Units (GRU, another variation on an RNN) have a more complex structure than a RNN, these comprise - comprising of three gates:

* Input gate
* Output gate
* Forget gate.

These three gates represent sigmoid layers, which control how much information is allowed to enter, leave or be forgotten within the layer.

This is an extremely [good post][3] if you want a full in-depth understanding of LSTM layers.

## PyTorch Model

Now, let's take a look at our model:

```python
class Model(nn.Module):
	def __init__(self, embedding_dim, hidden_dim, vocab_size):
		super(Model, self).__init__()
		
		self.hidden_dim = hidden_dim
		self.word_embeddings = nn.Embedding(vocab_size, embedding_dim)
		
		# The LSTM takes word embeddings as inputs, and outputs hidden states
		# with dimensionality hidden_dim.
		self.lstm = nn.LSTM(embedding_dim, hidden_dim)
		
		# The linear layer that maps from hidden state space to vocab size
		self.hidden2tag = nn.Linear(hidden_dim, vocab_size)
	
	def forward(self, sentence):
		embeds = self.word_embeddings(sentence)
		lstm_out, _ = self.lstm(embeds)
		tag_space = self.hidden2tag(lstm_out)
		return tag_space
```

Let’s break the model down by its layers.

#### Embedding layer

Embeddings have come to the forefront with the rise of generative AI. The role that they play here is compressing the potentially mammoth vocabulary size (`vocab_size`), down to a more manageable dimension of `embed_size` dimensions.

This embedding layer will help play a role within associated similar words close together within the `embed_size` vector space.  You can read more about embeddings [here](https://www.tensorflow.org/text/guide/word_embeddings#word_embeddings_2).

#### LSTM Layers

The next layer is our LSTM layer, this is the PyTorch implementation of what we described above. It takes the output of our embedding layer (`embed_size`), and outputs a tensor with `hidden_dim`.

#### Linear Layer

This dense linear layer takes the output from our LSTM layer, and outputs it a size `vocab_size` so we can apply a softmax function over it to get the probabilities for each word in `vocab_size` to get the highest ranked word.

## Predicting the next word.

Finally, I want to get into the `predict` function. The code is presented below.

```python
	def predict(prompt: str, max_seq_len: int, temperature: float, model, tokenizer, vocab, device, seed=None):
	if seed is not None:
		torch.manual_seed(seed)
		
	model.eval()
	tokens = tokenizer(prompt)
	indices = [vocab[t] for t in tokens]
	batch_size = 1
	
	with torch.no_grad():
		for i in range(max_seq_len):
			src = torch.LongTensor([indices]).to(device)
			prediction = model(src)
			probs = torch.softmax(prediction[:, -1] / temperature, dim=-1)
			prediction = torch.multinomial(probs, num_samples=1).item() # take one sample from the distribution
			while prediction == vocab['<unk>']:
				prediction = torch.multinomial(probs, num_samples=1).item()
			
			if prediction == vocab['<eos>']:
				break
			indices.append(prediction)
			
		itos = vocab.get_itos()
		
		tokens = [itos[i] for i in indices]
		
		return tokens
```

In this function, we have a `prompt`, this could be the start of the sentence. We have a `max_seq_len` argument that limits the number of words generated. That is to say, this function will keep predicting next words until `max_seq_len` is met or, the model itself predicts `<eos>`, indicating that text has finished.

We apply a temperature argument here. Temperature is a common parameter within generation models and works to 'temper' the models confidence on most likely predicted output. As temperature increases, this in a way increases the probability that lower predicted tokens will be selected by the next line where in which we sample the Multinomial distribution. [This blog has a brilliant visualisation of this][5].

## Grounding the model with context

One way to improve the model would be to try provide the model context as to what it's predicting. For example, writing an email to a friend would be very different to writing an email enquiring about a job.

Our two resources on the Smart compose model come from the blog post mentioned above, and the [associated research paper][4], in which the paper goes into more detail.

Google attempt to do just this! They try to ground the model with context by embedding the Subject line, and previous messages into the model, and concatenating this embedding with the input to the model. The helps model predict a relevant sentence given the context that the email is being written in.

![](https://2.bp.blogspot.com/-ilOCekdQP0Y/WvxdAt6fPZI/AAAAAAAACvE/2_bZTVZt2D8iwSeiKx1rB2rpTVbr_v9KQCLcBGAs/s640/model3.png)

## Applications

Although this is a form of supervised learning, I believe there are benefits to exploring these models outside of just learning.

Prediction models can help predict what to say next, encouraging users to give more details on customer support issues for example.

They can help guide customers to the right term to search within e-commerce or job boards.

Ultimately, being comparatively lightweight to generative models, they can have much lower latency and much lower cost while both increasing the customer experience and helping drive business metrics.


[1]:  https://blog.research.google/2018/05/smart-compose-using-neural-networks-to.html
[2]: https://stanford.edu/~shervine/teaching/cs-230/cheatsheet-recurrent-neural-networks
[3]: https://colah.github.io/posts/2015-08-Understanding-LSTMs/
[4]: https://arxiv.org/pdf/1906.00080.pdf
[5]: https://lukesalamone.github.io/posts/what-is-temperature/
