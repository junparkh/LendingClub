---
layout: page
title: Model Development
permalink: /menu3/
---
### Contents
{:.no_toc}
*  
{: toc}

## Overall Exploration Strategy

Our overall goal was to create a model that selects a small subset of loans to invest in and maximizes the overall APR of the portfolio. There are many ways to achieve this but for the purposes of this class, we decided to create models based on a consisten design matrix that would allow us to predict whether a loan is worth investing in or not by looking at the APR of the loan.

## Data Preparation 

### Missing Data

Only a few of the features had significant missing data. We had two options when dealing with this problem. Either eliminate the loans, or impute a value. We decided to impute values whenever possible in order to make sure that we were not systematically eliminating loans from our analysis.

* Loan length - if the borrower never repaid any of the loan then what is the loan length? For our purposes, the loan length is set to 1/365.25, thus yielding APR.

* Employment length - missing in 6% of the loans. We assumed 0.

* Credit history - when missing we just assumed 0.

* Annual income - when missing, we assumed the average. This was the most difficult one because a very good argument can be made for imputing 0. If the number is missing, then the person may be unemployed. At the end, we decided by looking at a subsample of data that this may not be a valid assumption and that it warranted further research. Unfortunately, there was insufficient time to explore this fully and come to a more satisfactory solution.

* DTI (Debt-to-Income) - when missing, we assumed the average, following logic similar to annual income.

### Design Matrix

We included in our design matrix only those features that were available to investors on the Lending Clib web site. These features were scaled to numbers from 0 to 1 using MinMaxScaler. In addition, we also tried to standardize them ((x-mean) / stddev) but this did not yield better results. Categorical values such as home ownership and income validation were encoded with one-hot-encoding.

### Objective

We designated APR as our objective. The APR concept is covered earlier. In our analysis we define it as:

APR = ( Total_Paid / Loan_Amount ) ^ Loan_Length_In_Years

Normally, APR is calculated based on the actual payments as they come in because of the time value of money. Money received sooner is better than money received later. However, since we lack this information, this calculation assumes the money comes in evenly over time. As discussed earlier, including the time element is important when comparing various loans.

For regression models, the models attempted to predict APR directly.

For categorical models, we designated a cutoff target APR. We created a binary category: loans that meet the minimum APR and loans that do not.

## Accuracy vs APR

The categorical models were optimized to maximize accuracy. But accuracy is not the best indicator of success because a few bad loans could easily wipe out any profits from good loans. For example, let's say we pick 10 loans: 9 of which return 7%, but one which is a total loss at -100%. That would be 90% accuracy. But still a return of -37%, which is pretty bad.

To account for this problem, we created an `evaluate_strategy` function which will estimate an average APR for a given model. It creates a list of the loans the model predicts will do well and then returns the mean of the APR. In essense, this would be like estimating the total return of investing $1 in each loan. One important metric is also the percent of loans that match the strategy. For example, if the strategy matches 90% of loans, then it is not good for selection. Likewise, if only 0.1% of loans are matched, it may require a lot of work to pick out winning loans.

## Overall structure of our method for determining which loans we should take:



### Loss prevention

We examined a few models to determine whether a loan would make money or lose money. That is, rather than picking a target APR, merely try to predict APR >= 0. In general these were no more successful than predicting a target return.

### Maximise profit

Regression models were unable to accurately predict (based on R^2) a numerical APR. Linear regressions, LR with lasso, random forest regression, ada boost regression all yielded very small R^2 and expected returns either close to 0 or negative. These models were surprisingly unable to make any clear predictions.

Classification models were also not very successful. Many of these models generated large accuracy scores, but fared badly when an expected return was calculated. Many models fared better than the overall average return -0.024. This was unexpectedly bad performance. Unfortunately, we ran out of time before we could fully explore this. One theory is that the model tends to find unusual loans. Usually, these are high returns, but many times, they are also exceptionally bad. Bad loans are far worse than good loans, because you can loose all your money, whereas your gains are limited by the interest rate.

Logistic Regression had a train and test accuracy around 0.7, but yielded an average APR of -0.08. We fit several Random Forest models. The first model with limited nodes and branches had an R^2 of about 0.23 for both test and train data. However, the strategy had an average return of -0.05.

### Portfolio(size) and discrimination
