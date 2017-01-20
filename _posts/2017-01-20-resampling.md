---
layout: post
title: Cross-Validation and the Bootstrap
subtitle: Continue et fais mes études tous les jours!
bigimg: /img/cv01.jpg
tags: [resampling methods]
---

# The Validation Set Approach

We explore the use of the validation set approach in order to estimate the
test error rates that result from fitting various linear models on the `Auto`
data set.

Before we begin, we use the `set.seed()` function in order to set a seed for
R’s random number generator, so that the reader of this book will obtain
precisely the same results as those shown below. It is generally a good idea
to set a random seed when performing an analysis such as cross-validation
that contains an element of randomness, so that the results obtained can
be reproduced precisely at a later time.

We begin by using the `sample()` function to split the set of samples into
two halves, by selecting a random subset of 196 observations out of the
original 392 observations. We refer to these observations as the training
set.

```javascript
library(ISLR)
set.seed(1)
train <- sample(392, 196)
```

(Here we use a shortcut in the sample command; see `?sample` for details.)
We then use the `subset` option in `lm()` to fit a linear regression using only
the observations corresponding to the training set.

```javascript
lm.fit <- lm(mpg ~ horsepower, data = Auto, subset = train)
```

We now use the `predict()` function to estimate the response for all 392
observations, and we use the `mean()` function to calculate the MSE of the
196 observations in the validation set. Note that the `-train` index below
selects only the observations that are not in the training set.

```javascript
attach(Auto)
mean((mpg - predict(lm.fit, Auto))[- train]^2)
```

Therefore, the estimated test MSE for the linear regression fit is 26.14. We
can use the `poly()` function to estimate the test error for the polynomial
and cubic regressions.

```javascript
lm.fit2 <- lm(mpg ~ poly(horsepower, 2), data = Auto, subset = train)
mean((mpg - predict(lm.fit2, Auto))[- train]^2)

lm.fit3 <- lm(mpg ~ poly(horsepower, 3), data = Auto, subset = train)
mean((mpg - predict(lm.fit3, Auto))[- train]^2)
```

# Leave-One-Out Cross-Validation

In this lab, we will perform linear
regression using the `glm()` function rather than the `lm()` function because
the latter can be used together with `cv.glm()`. The `cv.glm()` function is
part of the `boot` library.

```javascript
library(boot)
glm.fit <- glm(mpg ~ horsepower, data = Auto)
cv.err <- cv.glm(Auto, glm.fit)
cv.err$delta
```

We can repeat this procedure for increasingly complex polynomial fits.
To automate the process, we use the `for()` function to initiate a for loop
which iteratively fits polynomial regressions for polynomials of order i = 1
to i = 5, computes the associated cross-validation error, and stores it in
the ith element of the vector `cv.error`. We begin by initializing the vector.
This command will likely take a couple of minutes to run.

```javascript
cv.error <- rep(0, 5)

for(i in 1:5) 
{
  glm.fit <- glm(mpg ~ poly(horsepower, i), data = Auto)
  cv.error[i] <- cv.glm(Auto, glm.fit)$delta[1]
}

cv.error
```

# k-Fold Cross-Validation

The `cv.glm()` function can also be used to implement k-fold CV. Below we
use k = 10, a common choice for k, on the `Auto` data set. We once again set
a random seed and initialize a vector in which we will store the CV errors
corresponding to the polynomial fits of orders one to ten.

```javascript
set.seed(17)
cv.error.10 <- rep(0, 10)

for(i in 1:10) 
{
  glm.fit <- glm(mpg ~ poly(horsepower,i), data = Auto)
  cv.error.10[i] <- cv.glm(Auto, glm.fit, K = 10)$delta[1]
}

cv.error.10
```

# The Bootstrap

### Estimating the Accuracy of a Statistic of Interest

One of the great advantages of the bootstrap approach is that it can be
applied in almost all situations. No complicated mathematical calculations
are required. Performing a bootstrap analysis in `R` entails only two steps.
First, we must create a function that computes the statistic of interest.
Second, we use the the `boot()` function, which is part of the `boot` library, to
perform the bootstrap by repeatedly sampling observations from the data
set with replacement.

```javascript
alpha.fn <- function(data, index)
{
  X <- data$X[index]
  Y <- data$Y[index]
  return((var(Y) - cov(X, Y))/(var(X) + var(Y) - 2 * cov(X, Y)))
}

alpha.fn(Portfolio, 1:100)
```

The next command uses the `sample()` function to randomly select 100 observations
from the range 1 to 100, with replacement. This is equivalent
to constructing a new bootstrap data set and recomputing ˆα based on the
new data set.

```javascript
alpha.fn(Portfolio, sample(100, 100, replace = T))
```

We can implement a bootstrap analysis by performing this command many
times, recording all of the corresponding estimates for α, and computing
the resulting standard deviation. However, the `boot()` function automates
this approach. Below we produce R = 1, 000 bootstrap estimates for $$\alpha$$.

```javascript
boot(Portfolio, alpha.fn, R = 1000)
```

![](/img/cv02.png)

### Estimating the Accuracy of a Linear Regression Model

We first create a simple function, `boot.fn()`, which takes in the `Auto` data
set as well as a set of indices for the observations, and returns the intercept
and slope estimates for the linear regression model. We then apply this
function to the full set of 392 observations in order to compute the estimates
of β0 and β1 on the entire data set using the usual linear regression
coefficient estimate formulas from Chapter 3. Note that we do not need the
`{` and `}` at the beginning and end of the function because it is only one line
long.

```javascript
boot.fn <- function(data, index)
{
  return(coef(lm(mpg ~ horsepower, data = data, subset = index)))
}

boot.fn(Auto, 1:392)
boot.fn(Auto, sample(392, 392, replace = T))
```

Next, we use the `boot()` function to compute the standard errors of 1,000
bootstrap estimates for the intercept and slope terms.

```javascript
boot(Auto, boot.fn, 1000)
```

![](/img/cv03.png)
