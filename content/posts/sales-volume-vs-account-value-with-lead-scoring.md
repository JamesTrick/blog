+++
date = 2020-08-17T20:34:24Z
description = "There is often a trade-off between Sales Volume and average Account Value. Lead scoring can be both the cause, and the solution to getting the best of both."
hasMath = true
slug = "sales-volume-vs-account-value"
tags = ["customer success", "sales", "lead-scoring"]
title = "Sales Volume vs Account Value With Lead Scoring"

+++
![](https://res.cloudinary.com/dtexqfeo4/image/upload/v1598215831/blog/chrissie-kremer-Eq9uX_TuE_c-unsplash-min.jpg)

In one of my [recent posts on creating a lead score model](/posts/optimize-sales-for-growth-with-lead-scoring/), I mentioned that lead scoring can decrease Account Value. Whilst lead scoring can bring in higher Sales Volume it may do this at the cost of Account Value. I want to quickly explore this idea a bit more today.

## What is a valued lead?

For this, we want need to define value. For all businesses, this will be a combination of:

* How much your customers pay. (Average Revenue Per Customer for SaaS, Checkout value for a commerce business).
* Frequency. For SaaS, this will more be how long they stay before churning. For Commerce, this will also be how long between purchases.

This is formalised with the notion of Customer Lifetime Value (LTV). Simply put, LTV is a metric that tries to quantify, in dollar terms, how valuable a customer is.

In SaaS, the most common calculation is:

`$\frac{ARPU \cdot Gross Margin}{Churn Rate}$`

From this definition it's clear that there are three levers to increase LTV:

* Increase ARPU
* Increase Gross Margin
* Decrease Churn

## How can lead scoring lead to a decrease in LTV?

Lead scoring has the _potential_, to decrease in LTV. This statement rests on the notion that: It's easier to convert lower value customers than higher value customers. Logically, this makes sense. For example, as a consumer you think less when purchasing a $50 item when compared to a $200 item.

As lead scores aim to estimate a prospects sales readiness, a prospect intending to spend less will likely have a faster sales cycle - which will often result in a higher lead score. If a business follows the lead score, they may spend effort converting this prospect, brining in less value they could have. This, as we've seen with the definition holds or lowers ARPU, dampening LTV.

Lower value sales will still contribute positively to your Monthly Recurring Revenue (MRR) and Gross Sales targets. And, depending on your business' goals, it may be beneficial to focus _more_ on these lower value leads so long as they have a faster close rate than higher value leads. As it will grow your market share, and increase the all important "# of Active Users" on your platform.

More mature businesses (typically), may be more focussed on trying to increase the value of the lead which has a direct effect on LTV. Lead scoring can still be useful, in this case, but there are two considerations:

* In the ideal case, use digital means to convert high-ranked leads. Focus on lower ranked leads with potentially more value.
* Whilst building your model, consider not only conversion but the value.

The first point, is about recognising the potential short-comings of a lead score model and is general advice on optimising your sales efforts. Digital efforts are more likely to work on higher ranked leads. So getting your Sales Reps to focus on lower ranked leads can help increase both Sales Volumes and Account Value.

The second point, is of the technical nature and relates to model building. It encourages teams to think about going beyond the conversion phase and modelling value from prospects as well.

Often, I've seen the best implementation of this is when two models are built, and work independently of each other. One model focused on sales readiness (lead score) and another predicting the Account Value. When the two outputs are overlayed, it gives managers the freedom to scale between Sales Volume and Account Value at any given time.

Of course there are other considerations, such as if you have a Customer Success team who have the ability to upsell customers once converted, or the success of your win-back marketing campaigns. This post isn't to ward you off lead scoring, in fact, it's quite the opposite and shows how flexible lead scoring can be.