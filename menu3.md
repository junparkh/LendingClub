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

Our overall goal was to create a model that selects a small subset of loans to invest in and maximizes the overall APR of the portfolio. There are many ways to achieve this but for the purposes of this class, we decided to create models based on a consistent design matrix that would allow us to predict whether a loan is worth investing in or not by looking at the APY of the loan.

## Data Preparation 

### Missing Data

Only a few of the predictors that we had selected had significant missing data. We had two options when dealing with this problem. Either eliminate the loans, or impute a value. We decided to impute values whenever possible in order to make sure that we were not systematically eliminating loans from our analysis.

* Loan length - if the borrower never repaid any of the loan then what is the loan length? For our purposes, the loan length is set to 1/365.25, thus yielding APY.

* Employment length - missing in 6% of the loans. We assumed 0.

* Credit history - when missing we just assumed 0.

* Annual income - when missing, we assumed the average. This was the most difficult one because a very good argument can be made for imputing 0. If the number is missing, then the person may be unemployed. At the end, we decided by looking at a subsample of data that this may not be a valid assumption and that it warranted further research. Unfortunately, there was insufficient time to explore this fully and come to a more satisfactory solution.

* DTI (Debt-to-Income) - when missing, we assumed the average, following logic similar to annual income.

### Outliers

To deal with outliers, we only considered complete loans that were either paid off or charged off. Furthermore, we filterd loans that were shorter than 60 days. These were special situations and we believed they would skew the data. In addition, loans with an APY greater than 1 were also eliminated because these were truly exceptional situtations probably due to bad data.

Still, as shown in the exploratory analysis, there were numerous outliers in the positive end. Any loan with an APY greater that 0.25 is due to both a high interest rate and high fees (late fees, etc) that can sometimes drive the cost of the loan up significantly. We decided to keep these types of loans because they represent highly risky but lucrative loans.

### Design Matrix

We included in our design matrix only those features that were available to investors on the Lending Club web site. We tried to standardize them ((x-mean) / stddev) but this did not improve our results. Categorical values such as home ownership and income validation were encoded with one-hot-encoding.

### Objective

We designated APY as our objective. The APY concept is covered earlier. In our analysis we define it as:

APY = ( Total_Paid / Loan_Amount ) ^ Loan_Length_In_Years

Normally, APY is calculated based on the actual payments as they come in because of the time value of money. Money received sooner is better than money received later. However, since we lack this information, this calculation assumes the money comes in evenly over time. As discussed earlier, including the time element is important when comparing various loans.

For regression models, the models attempted to predict APY directly.

For categorical models, we designated a cutoff target APY. We created a binary category: loans that meet the minimum APY and loans that do not.

One problem with using APY is that it can be negative. This may cause problems in the models because negative values require negative coefficients. In the *Future Exploration* section below we discuss various ways we were planning to deal with this.

### Test / Train split

For our initial model exploration, we split the entire data set into a training set and a test set. 75% of the data was used for training. 25% was used for testing. Our plan was to later tune model hyperparameters by creating three data set: Training, Tuning, Test. As explained later, we ran out of time to do this.

## Accuracy vs APY

The categorical models were optimized to maximize accuracy. But accuracy is not the best indicator of success because a few bad loans could easily wipe out any profits from good loans. For example, let's say we pick 10 loans: 9 of which return 7%, but one which is a total loss at -100%. That would be 90% accuracy. But still a return of -37%, which is pretty bad.

To account for this problem, we created an `evaluate_strategy` function which will estimate an average APR for a given model. It creates a list of the loans the model predicts will do well and then returns the mean of the APY. In essense, this would be like estimating the total return of investing $1 in each loan. One important metric is also the percent of loans that match the strategy. For example, if the strategy matches 90% of loans, then it is not good for selection. Likewise, if only 0.1% of loans are matched, it may require a lot of work to pick out winning loans.

A better solution would be to optimize for average APY instead of accuracy. However, we ran out of time before we were able to try this.

## Overall structure of our method for determining which loans we should take:

Our plan was to create a number of different models to see which ones were most promising and then exploring these in greater detail. Unfortunately, time constraints limited our ability to explore models in depth. 

As a consequence, we were unable to optimize the hyperparameters for promising models. Our plan was to create three data sets (or more as needed): training, tuning, testing. We would then tune the hyperparameters based on the results on the tuning set. Additionally, we had hoped to do cross validation to analyze the stability of our parameters. This is an area for future exploration.

### Loss prevention

We examined a few models to determine whether a loan would make money or lose money. That is, rather than picking a target APY, merely try to predict APY >= 0. In general these were no more successful than trying predict which loans would meet a minimum target return.

### Exploring: maximize profit

To start, we created several baseline models to see if which algorithms, if any, were worth exploring.

Regression models were unable to accurately predict (based on R^2) a numerical APR. Linear regressions, LR with lasso, random forest regression, ada boost regression all yielded very small R^2 and expected returns either close to 0 or negative. These models were unable to make many good predictions.

Classification models were also not very successful. Many of these models generated large accuracy scores, but fared badly when an expected return was calculated. Many models fared only slightly better than the overall average return -0.024. This was unexpectedly bad performance. Unfortunately, we ran out of time before we could fully explore this. One theory is that the models tended to find unusual loans. Usually, these are high returns, but some times, they are also exceptionally bad. Bad loans are far worse than good loans, because you can loose all your money, whereas your gains are limited by the interest rate.

Logistic regression had a train and test accuracy around 0.7, but yielded an average APY of -0.08. 

We fit several random forest models. The first model with limited nodes and branches had an R^2 of about 0.23 for both test and train data. However, the strategy had an average return of -0.05.

Using the same random forest model, rather than looking at the predictions, we looked at the probabilities predicted. We chose only those loans where the probability was greater than 0.9. That stretegy select 8% of returns and yielded an APY of 0.028, which is better than average.

Random forest regression did not yield better results. Adaboost regressor was unable to create a working model.

We created two adaboost classification models. The first did not use a base classifier. The second used random forest. Both were unable to yield positive returns.

Next, we considered the use of polynomial factors. In order to limit the complexity of the models, we chose a subset of the most promising factors and then created polynomial factors of order 2. A logistic regression yielded R^2 of 0.7 for both train and test. However, the strategy yielded a loss.

A single decision tree, linear discriminant analysis and quadratic discriminant analysis likewise did not yield good results.

A neural network model did not yield good results on its own. However, when we looked at the probabilities the results were more promising. A strategy of picking stocks where the probability estimates were over 0.9 yielded returns of 0.05 on the test data, which is significantly higher than average and the best results so far.

Lastly, we created a stacking model combining probability estimates of the logistic, random forest and neural network models created earlier. A strategy of picking only those loans where all three models agreed with probability of more than 0.8 yieled returns of 0.02.

Thus, the best model was a neural network model that selected loans where the probability estimates were over 0.9. This yielded returns close to 0.05 on the test data.

## Future Exploration

As mentioned earlier, we need to perform further optimization of hyperparameters for the most promising models by creating a separate tuning data set. In addition, cross validation needs to be performed to ensure reliable results.

One thing that stood out was that most models did somewhat worse than the average return. Even models where the R^2 and accuracy were similar for both the training and testing set would often perform worse than average. This was counter-intuitive and suggest serious problem with either the data or the model. Unfortunately, we were unable to fully explore the reasons for this. One posibility is that the train/test split needs to be perfomed differently. Another is overfitting.

Beacause APY can be negative, which may confuse the models, our plan was to create models using 1+APR to eliminate negative numbers. Another plan was using the log of the APY, as in "1/log(1+APY)", to penalize negative APYs. Or even 1+log(1/(1 + APR)), which would have smaller numbers for better loan choices, while still penalizing bad loans. These sorts of modifications would have created significantly different models that may have worked better. The biggest problem with the models was that a few bad loans dispropotionately effected the return.

Another question that we did not have time to investigate is how large our investment portfolio must be to reduce volatility. Even if predictions are on average good, it's possible that the average only holds over very large numbers of loans. If you have a smaller loan portfolio, the returns might be more volatile and unpredictable. Cross validation of the models using different sizes of randomized loan portfolios would help address this question.

### Portfolio(size) and discrimination

Although we did collect data on demographics, we were unable to join to our data due to time constraints. Our plan was to see if our models tended to pick loans in locations that were primarily non-minority and then explore why the models behaved as they did. Unfortunately, time constraints did not allow this. Even though this would not be proof of discrimination, it might suggest areas of further exploration.
