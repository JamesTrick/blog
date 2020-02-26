+++
date = 2020-02-26T02:13:04Z
tags = ["sales", "customer success", "stocks", "research"]
title = "Balancing Research and Sales Spend"

+++
Sales and Marketing (S&M) and Research and Development (R&D) often compete when it comes to company budgeting.

For budget managers and shareholders alike it’s interesting and beneficial to compare companies spend in this area to get insight into strategy and design budgets.

A good start would be to look at how some of the largest US listed Software companies split their spending between S&M and R&D.

This plot below illustrates that there is a large range of spending choices, right from Square through to Splunk.

![](/static/graphs/Top_Companies-1.png)

Notably, both PayPal and Square are companies that rely on being integrated as payment services by 3rd parties, which then allows them to earn a fee per transaction. This technique evidently requires less sales spend to produce high revenues.  
  
This does allow one an insight into company strategy, with ServiceNow and Zendesk spending more on R&D as a proportion of Revenue compared to that of incumbent Salesforce. There are several reasons for this, undoubtedly including Salesforce’s history and appetite for acquisitions.

## **What is the optimum spend?**

If we were to run a regression against incremental change in Revenue only using incremental change in R&D and S&M spend,we get an impressive R^2 of 0.57, meaning that 57% of variance in revenue can be explained solely by S&M and R&D expenditure.

There’s no real doubt that R&D and S&M are key drivers for increasing revenue, both proved by both the model and intuition.

However, one can hypothesise that there might be a stage in a company’s life where one is more important than others.

## **Sales and Marketing spend By Company Size**

As we're comparing companies in same industry it's fair to use Total Assets as a proxy for company size. Given this, when visualising Sales and Marketing spend against Total Assets, we can see that there is a weak negative relationship.

![](/static/graphs/sm_expense-1.png)

Despite the relationship being weak, we could infer that larger companies may spend less on Sales and Marketing. But there’s no big difference that I was looking for. Maybe Henry Ford was right when [he famously once said](https://www.briansorce.com/10-marketing-quotes-henry-ford/): “A man who stops advertising to save money is like a man who stops a clock to save time.”

## **Research and Development spend by Company Size**

Equally, on the R&D spend we see a weak relationship between R&D Spend and Assets, however this trend is positive and a slightly better fit.

![](/static/graphs/rd_figure-1.png)

By visualising data such as this, we can draw conclusions that smaller companies favour Sales and Marketing spend, perhaps in an attempt to increase their size or due to lack of resources.

As discovered, exploring the split between S&M and R&D expenses is an interesting exercise and helpful in accessing company strategy. From this analysis we drew some insight into the relationship of company size and expenditure.