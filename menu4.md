---
layout: page
title: Summary of Results
permalink: /menu4/
---

# Final Solution

The best model was a simple neural network model. We selected loans where the probability estimates of the network were over 0.9. This yielded returns close to 5% on the test data. If we had more time, there are numerous ways to improve upon this model which are explained in other sections.

# Debt Returns Not Symetrical

Debt returns are not symetrical because profits are limited, but losses are not. Thus, a few bad loans can quickly overcome any good loans. In the histogram of returns, a fat tail in negative returns is prominent.

![image](/image/loan.png)

Lending Club's average return is -2%. That is, if you choose loans at random, you will expect to lose 2% of your money per year. On the other hand, US Treasury bonds are yielding around 2.45%. A Treasury bond is a risk-free investment that, barring a collapse of the US Government, will get rapaid. Thus any investment must be able to return more than the basic 2.45% yield to be worth doing because otherwise you would just invest in Treasury bonds. 

We created numerous simple models to predict superlative loans. Of these, the best model was a neural network model that selected loans where the probability estimates were over 0.9. This yielded returns close to 5% on the test data. This is quite a bit above average and also above the US Treasury baseline.

# Accuracy not a good measure

If given more time, we would have explored different models that optimize for things other than accuracy. Accuracy is not a good measure because a few bad loans can quickly wipe out any profits. Thus, even at 90% accuracy for predicting reliably good loans, you might still lose money. We could have created models that optimize on expected return. In addition, our choice of APY as our outcome variable may not have been ideal. A measure that penalizes bad loans may work better.

# Validation and Hyperparameter Optimization

Although some models were promising and worth futher investigation, we did not have the time to tune for hyperparameters (using separate tuning data set). In addition, we did not perfom cross-validation to ensure our models are consistent. For the neural network, it would be helpful to explore different architectures.