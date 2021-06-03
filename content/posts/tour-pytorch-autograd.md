+++
date = 2021-02-01T08:06:57Z
description = "I take a dive into PyTorch autograd to explore what it does and how it relates to training deeplearning models."
hasMath = false
slug = "tour-of-pytorch-autograd"
tags = ["data", "python", "pytorch", "machine learning"]
title = "PyTorch Autograd and Training Models"
+++

![Header image of calculus](https://miro.medium.com/max/3600/1*YBt8A5Nsyf1naLHLS9iHPQ.jpeg)

Recently I’ve found myself wanting to dig a bit deeper into PyTorch to really understand how it works and hopefully figure out how to expand on my skillset to try new things.

To aid this, I’ve been following the Deep Learning with PyTorch book by Eli Stevens, Luca Antiga and Thomas Viehmann. The book itself has been great and highly useful. I recommend it to anyone.

In my observations, one the hardest things for newer users of PyTorch to understand is how to train a model. Especially, say if your first taste of Deep learning was from a very high-level package such as Keras where the syntax is akin to `model.train()`. Having to write a training loop in PyTorch was confusing and hard.

In fact, I didn’t fully understand the logic behind this until I read this Chapter 5 on autograd. I wanted to share my thoughts and a bit of a summary on autograd and how it relates to training neural networks.

## What is Autograd and how is it useful?

PyTorch describes autograd as the automatic differentiation engine that powers neural network training. In short, it does the math of differentiation for you. Autograd isn’t unique to PyTorch, other packages such as Tensorflow and JAX have autograd methods.

Perhaps the best way to start to understand how a deep learning model learns to approximate data is by not actually starting with a deep learning model at all and instead start with a simple linear model.

Let’s say we’re trying to fit a linear model such as: $ y = \alpha + \beta * x $. Yes, I know in reality you’d use scikit-learn or statsmodel, but the idea is to start from sctratch to understand the training loop used for more advanced deep learning methods.

First, we need the model, which can be coded as follows:

```python
def model(x, w, b):
    return b + w * b
```

Next, we need a loss function. Loss functions in reality are very problem specific and we won't delve into it here. 

For this basic case, we’ll use a Mean Squared Loss, in PyTorch it is `nn.MSELoss`. But we’ll code it explicitly as follows:

```python
def loss_fn(y_true, y_hat):
    diff = (y_hat - y_true)**2
    return diff.mean()
```
It's pretty clear now that if want to fit the data, we want to minimise the loss function. We can do that, by updating the weights, `w` and `b` we use in our model.

In order to perform gradient descent, we can take the derivative of the loss function, which would become:

```python
def d_loss_fn(y_true, y_hat):
    d_sq_diffs = 2 * (y_hat - y_true) / y_hat.size(0)
    return d_sq_diffs
```
The above function is derivative of the mean in our loss function. We can then repeat the following for the derivatives of our model with respect to w and b:

```python
def dmodel_dw(t_u, w, b): 
    return t_u

def dmodel_db(t_u, w, b): 
    return 1.0
```
Putting this together, we get a gradient function as follows:

```python
def  grad_fn(t_u, t_c, t_p, b):
    dloss_dtp = dloss_fn(t_p, t_c)
    dloss_dw = dloss_dtp * dmodel_dw(t_u, w, b)
    dloss_db = dloss_dtp * dmodel_db(t_u, w, b)
    return torch.stack([dloss_dw.sum(), dloss_db.sum()])
```

Finally, we can train our model as follows:

```python
def training_loop(n_epochs, learning_rate, params, x, y):
    for epoch in range(n_epochs + 1):
        w, b = params
        y_hat = model(x, w, b)
        loss = loss_fn(y_hat, y)
        grad = grad_fn(t_u, t_c, t_p, b)
        params = params - learning_rate * grad
```

## The power of autograd

Whilst the above example works and will train your model nicely, it's not scalable nor feasible to manually write the derivatives for functions. This is where autograd comes in!

The biggest change we need to do to our code is to specify that our params tensor needs autograd, we do this by the following:

```python
params = torch.tensor([1., 0.], requires_grad=True)
```
If now, we call `params.grad`, we'll get None back. That's because we actually haven't performed any differentiation.

Let's update our training loop and we'll break it down.

```python
def training_loop(n_epochs, learning_rate, params, x, y):
    for epoch in range(n_epochs + 1):
	if params.grad is not None:
	    params.grad.zero_()
	    
        y_hat = model(x, *params)
        loss = loss_fn(y_hat, y)
        loss.backward()

	with torch.no_grad():
	   params = params - learning_rate * params.grad
```
If you're familiar with PyTorch training loops, this should start to look a bit familiar.

Our first action, is to check whether gradients have already been calculated for our `params`. If they have, we set them to zero to avoid accumulating gradients.

The second step is to pass our data into the model and calculate our loss - like we did before. The difference here is we call `.backward()` on our loss. This then performs the differentiation that we manually calculated before!

The final part is where we update our `params` for the next epoch. Take note of the special context manager `with torch.no_grad():` here. This is needed as by default, all tensors with `requires_grad=True` are tracking their computational history and support gradient computation. When updating our `params`, we don't need this or want this.

## Moving to something familiar

We've done a fair already, and we're still being pretty explicit with our training. Of course, PyTorch has many functions many of which you've probably already seen, so now we'll use those and link back to what we've done.

### Optimizers

So far, we've been doing simple gradient descent by updating our params using a learning rate and a gradient, either calculated ourselves or calculated using autograd - nothing too fancy.

PyTorch has a huge range of optimizers we can use in `torch.optim`. To make life more simple, let's use one of them.

```python
params = torch.tensor([1., 0.], requires_grad=True)
optimizer = torch.optim.SGD([params], lr=learning_rate)

def training_loop(n_epochs, optimizer, params, t_u, t_c):
    for epoch in  range(n_epochs + 1):
        optimizer.zero_grad()
        t_p = model(t_u, *params)
        loss = loss_fn(t_p, t_c)
        loss.backward()
        optimizer.step()
return params
```

If you've trained a model in PyTorch before, this example should be similar to what you've done. Also, by following this post, it should be familiar to what we've already done.

First, we clear the gradients of our optimizer. We did this before with the if statement and manually clearing our gradients. 

Second, we pass the inputs and calculate the loss value. Like before, we use the power of autograd to calculate the derivative of our loss function using `.backward()`.

Finally, we allow the optimizer to handle the updating of our params for the next epoch.

PyTorch does a lot of the heavy lifting of needing to update weights manually. But choosing a optimizer is important when training more complex models.

## Summary
autograd enables PyTorch to do what it does, so naturally there is a lot to understand and take in. This post and indeed myself, only scratches the surface of autograd. Surely, this post will be updated and refined over time as I learn more and figure out gaps in the above.

If you're new to PyTorch, hopefully this helps explain the purpose of the training loop, even if the details are fuzzy. If you're more experienced with PyTorch, hopefully this post explains the why. 

Let me know in the comments below your thoughts and experiences.

## References/More Reading

 - [PyTorch Tutorial on autograd](https://pytorch.org/tutorials/beginner/basics/autogradqs_tutorial.html)
 - [Unrelated, but good example of the power of Autograd by Cameron Davidson-Pilon](https://dataorigami.net/blogs/napkin-folding/the-delta-method-and-autograd)
