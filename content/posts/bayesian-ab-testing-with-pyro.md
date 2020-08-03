+++
date = 2020-04-06T06:14:20Z
tags = []
draft=true
title = "Bayesian AB Testing with Pyro"

+++
Recently, I’ve been involved in experiment design and measurement - specifically AB Testing.

The main challenges involved with this experiment design is that we were dealing with a largely non-technical audience. Instead of an AB test for a website, it involved lower sample size and costly human interventions.

As a result, my desire and my stakeholder’s desire were to capture small gains quickly and iterate.

Because of this, I chose Bayesian AB Testing as a measurement framework. Literature on Bayesian AB testing is readily available, so I won’t go into a lot of detail explaining how it works. This post, by Dynamic Yield explains it quite well.

Instead, we’ll start with a quote from Dynamic Yield on Bayesian AB testing:

> _“The math behind the Bayesian framework is quite complex so I will not get into it here. In fact, I would argue that the fact that the math is more complicated than can be computed with a simple calculator or Microsoft Excel is a dominant factor in the slow adoption of this method in the industry.”_

Dynamic Yield do also offer a great little calculator here, but this post is also designed to get you up and running with your own Bayesian AB testing using Pyro.

**Introducing Pyro**

Pyro is a probabilistic programming language originally open-sourced by Uber AI. It’s uses and functionality far exceed AB testing but is perfectly suited to this type of problem.

To make this simple, let’s start with a story. Your start-up is trialling methods of welcoming customers to a free trial. You either call them, or email them to try get them to convert successfully to your product.

In this, you want to know what channel is performing better – calling or emailing?

We can summarise the results as below:

| --- | --- | --- |
| Method | Attempts/Reach | Conversions |
| Email | 700 | 73 |
| Phone | 200 | 35 |

Something to notice, is our \`n\`, sample sizes are different between each campaign. This is an advantage that Bayesian AB testing can give you over Frequentist testing.

Using Pyro, we can define our model as follows:

    def ab_model(obs_v1, obs_v2, n_1, n_2):
        prior_v1 = pyro.sample("prior_v1", dist.Beta(2, 2))
        prior_v2 = pyro.sample("prior_v2", dist.Beta(2, 2))
        
        difference = pyro.deterministic("difference", prior_v1 - prior_v2)
        
        with pyro.plate('likelihood_1', 1000):
            likelihood_v1 = pyro.sample("likelihood_v1", dist.Binomial(total_count=n_1, probs=prior_v1), obs=obs_v1)
            likelihood_v2 = pyro.sample("likelihood_v2", dist.Binomial(total_count=n_2, probs=prior_v2), obs=obs_v2)
        return difference

Breaking it down, Pyro's models look like regular python functions.

`prior_v1` is our prior knowledge on the performance of emails. We can encode prior knowledge into this by simply changing the Beta parameters, as illustrated below:

\[Graph of different Betas\]

Differences measures the expected difference between the performance of `prior_`_`v2` when compared to `prior`_`_v1`.

We can now condition our model to find the true values of prior_v1, prior_v2 but more importantly yield those helpful insights.

To do this, we'll use Markov Chain Monte Carlo

    nuts = NUTS(ab_model, adapt_step_size=True)
    
    mcmc = MCMC(nuts, num_samples=3000, warmup_steps=200)
    mcmc.run(torch.tensor(obs_v1, dtype=torch.double),
             torch.tensor(obs_v2, dtype=torch.double),
            torch.tensor(n_1, dtype=torch.double),
            torch.tensor(n_2, dtype=torch.double))

The mechanics behind MCMC is again be 