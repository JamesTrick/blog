+++
date = 2020-02-24T05:17:55Z
draft = true
tags = ["natural language processing", "python"]
title = "Classifying German Text with PyTorch"

+++
Welcome to this first article on applying modern NLP techniques using [PyTorch](https://pytorch.org/) and [torchtext](https://pytorch.org/text/) to the wonderful German language.

This article is going to be largely split in three parts, covering:

* Data Preparation and loading it successfully into torchtext.
* Applying a FastText style model to the German Dataset.
* Applying a Bidirectional RNN to the German Dataset.

### The Intuition

### The Data

### Loading into Torchtext

Our primary goal is to use a Neural Network built using PyTorch. torchtext is a library built by some of the core PyTorch team, focusing on data preparation and loading of natural language data. It’s similar to Tensorflow’s, [TF.text](https://www.tensorflow.org/tutorials/tensorflow_text/intro).

There are plenty of tutorials on using torchtext around, so this post isn’t going to step through the absolute basics — instead, this post is going to highlight the differences of loading a foreign language to English.

_This tutorial is intended for German, but the principles should apply for any language._

The step is to define the fields needed. We can do this as below:

    import spacy
    
    from torchtext import data
    
    # Must first download the German model
    # Can do this, by running `python -m spacy download de_core_news_sm`
    nlp = spacy.load("de_core_news_sm")
    
    def tokenizer(x):
      return [w.text for w in nlp.tokenizer(x)]
    
    TEXT = data.Field(
      tokenize=tokenizer
    )
    
    LABEL = data.LabelField()

Important things to note are:

* We define our own tokenizer using [Spacy](https://spacy.io/). Torchtext does have a tokenizer_language field where you can specify the spacy language to use. However by defining your own tokenizer, it makes it easier to make changes further down the track.