+++
date = 2020-09-13T10:13:56Z
description = "Trustpilot is a leading consumer review site, understand how they rank businesses and how you can learn from it."
hasMath = false
draft = false
slug = "trustpilot-scoring"
tags = ["data", "analytics", "business"]
title = "How does Trustpilot scoring work?"

+++
\[Trustpilot\]([https://trustpilot.com/](https://trustpilot.com/ "https://trustpilot.com/")) is a wide-spread consumer review platform. Founded in 2007, they have over 1m reviews posted every month from consumers. Reviewers share their experiences with a company, giving a score, from one to five and a free-text comment.

Companies proudly display the scores on emails, websites, and other mediums as a form of social proof. As such, it’s important to understand how Trustpilot works and how you can influence the scores.

## **How is Trustpilot’s TrustScore calculated?**

TrustScore is a measure of overall satisfaction, and can be seen in a similar light to Net Promoter Score, NPS. Unlike NPS, the score is bounded from 1 to 5 stars.

Trustpilot are relatively open in terms of how the TrustScore is calculated, saying there are three factors that contribute towards a company’s TrustScore:

* Recency of reviews
* Frequency of reviews
* Bayesian Averaging.

Let’s now explore each aspect and how it affects your businesses score.

### Bayesian Averaging

The most important statement is that they use Bayesian averaging - a technique not uncommon for rating products, forum posts, and in this case - companies.

\[Wikipedia defines Bayesian averaging as\]([https://en.wikipedia.org/wiki/Bayesian_average](https://en.wikipedia.org/wiki/Bayesian_average "https://en.wikipedia.org/wiki/Bayesian_average")): “A Bayesian average is a method of estimating the mean of a population using outside information, especially a pre-existing belief, that is factored into the calculation.”

Bayesian Averaging really shines where there are few reviews or samples, as it starts with an initial belief. TrustPilot even state their initial belief, by saying that businesses start with a review score of 7 reviews, 3.5 - a very neutral view of a business.

To understand this, let’s think about a new business whose first review is 5 stars. A simple, frequentist average will simply take the score divided by the number of reviews, giving a ranking of 5 stars despite only having one view.

Bayesian Averaging implies that given a 5 star review, we’ll update the prior belief (7 reviews of 3.5 stars) to reflect this new review. As a result, the TrustScore will increase, but will not go straight to 5 stars - the company will need to work harder than that.

Bayesian averaging results in a much smoother movement over time, with scores not whipsawing between 1 and 5 stars. It also means that for a business' score to go up (or down), there needs to be a consistent period of good (or bad) reviews as scores aren’t affected largely by one off reviews.

### Moving on from Bayesian Averaging

This brings us nicely to the first two points: Frequency and Recency. TrustPilot states that they give less weight to older reviews and by default more weight to newer reviews.

![Example of exponential decay](/static/graphs/exp_decay.png "Example of exponential decay")

Businesses, wanting to improve or even maintain their score need to work hard to continually get frequent reviews to maintain recency in their reviews.

This also has an added benefit for businesses, in that recent reviews have more of an effect on your current score than old ones. 

Although we've explored that Bayesian averaging helps smooth out scoring for businesses with few reviews, exponential decay can help (or not help if reviews are negative) change the score of businesses with many existing reviews at a faster rate than a simple average as it weights recent reviews more highly.

From Trustpilot’s view, it’s a fantastic measure, as it encourages businesses to actively engage with their platform to help drive frequent reviews to maintain a good score. And for consumers, scores are often more representative and more meaningful than a simple average.
