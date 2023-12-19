+++
date = 2023-12-19T00:00:00Z
description = "Trustpilot is a leading consumer review site, understand how they rank businesses and how you can learn from it."
hasMath = false
slug = "trustpilot-scoring"
tags = ["data", "analytics", "business"]
title = "How does Trustpilot scoring work?"
+++

[Trustpilot](https://trustpilot.com) is a wide-spread consumer review platform. Founded in 2007, they have over 1m reviews posted every month from consumers. Reviewers share their experiences with a company, giving a score, from one to five and a free-text comment.

Companies proudly display the scores on emails, websites, and other mediums as a form of social proof. As such, it’s important to understand how Trustpilot works and how you can influence the scores.

## What is Trustpilot’s TrustScore?

Trustpilot’s public review forum is most similar to that of the Net Promoter Score (NPS) - but in the public eye.

When consumers are leaving a review, they are have to leave a numerical star rating, and are encouraged to leave written feedback in the free-text box. This free form text is visible to other consumers on the company’s website.

To aggregate these scores, Trustpilot distils this information into a relevant score called TrustScore.

## How is Trustpilot’s TrustScore calculated?

TrustScore is a measure of overall satisfaction, and can be seen in a similar light to Net Promoter Score, NPS. Unlike NPS which is ranked from -100 to 100, TrustScore is bounded between 1 to 5 stars.

Trustpilot are very open in terms of [how the TrustScore is calculated](https://support.trustpilot.com/hc/en-us/articles/201748946-TrustScore-and-star-rating-explained), saying there are three factors that contribute towards a company’s TrustScore:

* Recency of reviews
* Frequency of reviews
* Bayesian Averaging.

Let’s now explore each aspect and how it affects your businesses score.

### Recency of reviews

Trustpilot states that they give more weight to newer reviews than older reviews. This is likely done by an exponential decay function that downweights reviews values as time goes on.

This is a [common extension of review systems](https://www.evanmiller.org/bayesian-average-ratings.html), and is more helpful for consumers as they want a review that reflects the current state of the business.

### Frequency of reviews

This second point is tied in closely to that of the first. The more frequent your reviews are, the more accurate the score will be. For businesses with less than 10,000 reviews the TrustScore will be re-calculated after every review. For those with over 10,000 reviews, the score will be re-calculated on a daily basis.

### Bayesian Averaging

The most important statement is that they use Bayesian averaging - a technique not uncommon for rating products, forum posts, and in this case - companies.

To best understand bayesian averaging, let’s illustrate two businesses:

* Business A has 9,700 reviews and a TrustScore of 4.4
* Business B has 2 reviews and a TrustScore of 5.

Which business would you trust more? Despite Business B having higher score, you might lean to toward trusting Business A because of the volume of reviews, and consider Business B to be abnormally high.

This concept is central to bayesian averaging, of which [Wikipedia defines as](https://en.wikipedia.org/wiki/Bayesian_average): “A Bayesian average is a method of estimating the mean of a population using outside information, especially a pre-existing belief, that is factored into the calculation.”

Bayesian averaging really shines for consumers evaluating businesses with few reviews. Trustpilot say, that all new businesses start off with a review score of 7 reviews, 3.5 - a very neutral view of a business. This is the pre-existing belief, that the business is neither bad nor great.

Bayesian Averaging implies that given a 5 star review, we’ll update the prior belief (7 reviews of 3.5 stars) to reflect this new review. As a result, the TrustScore will increase, but will not go straight to 5 stars - the company will need to work harder than that.

Bayesian averaging results in a much smoother movement over time, with scores not whipsawing between 1 and 5 stars. It also means that for a business’ score to go up (or down), there needs to be a consistent period of good (or bad) reviews as scores aren’t affected largely by one off reviews.

### How frequency and recency tie into bayesian averaging

By now, you’ll probably notice the best way to influence a TrustScore is to gather reviews - this is exactly the behaviour that bayesian averaging has and the exponential decay has.

Businesses, wanting to improve or even maintain their score need to work hard to continually get frequent reviews to maintain recency in their reviews.

This also has an added benefit for businesses, in that recent reviews have more of an effect on your current score than old ones. You can learn more [about how to hack your Trustscore in this follow-up post](/posts/improving-trustpilot-score/).

From Trustpilot’s view, it’s a fantastic measure, as it encourages businesses to actively engage with their platform to help drive frequent reviews to maintain a good score. And for consumers, scores are often more representative and more meaningful than a simple average.
