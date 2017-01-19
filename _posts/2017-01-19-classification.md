---
layout: post
title: Classification Problems
subtitle: start to learn in this winter vacation
bigimg: /img/study01.jpg
tags: [Logistic regression, LDA, QDA, KNN]
---

# The Stock Market Data

We will begin by examining some numerical and graphical summaries of
the `Smarket` data, which is part of the `ISLR` library. This data set consists of
percentage returns for the S&P 500 stock index over 1, 250 days, from the
beginning of 2001 until the end of 2005. For each date, we have recorded
the percentage returns for each of the five previous trading days, `Lag1`
through `Lag5`. We have also recorded `Volume` (the number of shares traded
on the previous day, in billions), Today (the percentage return on the date
in question) and `Direction` (whether the market was `Up` or `Down` on this
date).

We can use the following `R codes` to view the data structure.

```javascript
library(ISLR)
names(Smarket)
dim(Smarket)
summary(Smarket)
cor(Smarket[ ,-9])
```

The `cor()` function produces a matrix that contains all of the pairwise
correlations among the predictors in a data set. 

We can also make a plot of the data.

```javascript
attach(Smarket)
plot(Volume)
```
![](/img/study02.png)

As one would expect, the correlations between the lag variables and today’s
returns are close to zero. In other words, there appears to be little
correlation between today’s returns and previous days’ returns. The only
substantial correlation is between `Year` and `Volume`. By plotting the data we
see that `Volume` is increasing over time. In other words, the average number
of shares traded daily increased from 2001 to 2005.

# Logistic Regression

Next, we will fit a logistic regression model in order to predict `Direction`
using `Lag1` through `Lag5` and `Volume`. The `glm()` function fits generalized
linear models, a class of models that includes logistic regression. The syntax
of the `glm()` function is similar to that of `lm()`, except that we must pass in
the argument `family=binomial` in order to tell `R` to run a logistic regression
rather than some other type of generalized linear model.

We can use the following codes to deal with the logistic regression.

```javascript
train <- (Year < 2005)
Smarket.2005 <- Smarket[!train, ]
Direction.2005 <- Direction[!train]

glm.fit <- glm(Direction ~ Lag1 + Lag2 + Lag3 + Lag4 + Lag5 + Volume,
               data = Smarket, family = binomial, subset = train)
glm.probs <- predict(glm.fit, Smarket.2005, type = "response")

glm.pred <- rep('Down', 252)
glm.pred[glm.probs > .5] <- "Up"

table(glm.pred, Direction.2005)
mean(glm.pred == Direction.2005)
```

# Linear Discriminant Analysis

Now we will perform LDA on the `Smarket` data. In `R`, we fit a LDA model
using the `lda()` function, which is part of the `MASS` library. Notice that the
syntax for the `lda()` function is identical to that of `lm()`, and to that of
`glm()` except for the absence of the `family` option. We fit the model using
only the observations before 2005.

```javascript
library(MASS)
lda.fit <- lda(Direction ~ Lag1 + Lag2, data = Smarket, subset = train)
plot(lda.fit)

lda.pred <- predict(lda.fit, Smarket.2005)
lda.class <- lda.pred$class
table(lda.class, Direction.2005)
mean(lda.class == Direction.2005)
```
![](/img/study03.png)

# Quadratic Discriminant Analysis

We will now fit a QDA model to the `Smarket` data. QDA is implemented
in `R` using the `qda()` function, which is also part of the `MASS` library. The
syntax is identical to that of `lda()`.

```javascript
qda.fit <- qda(Direction ~ Lag1 + Lag2, data = Smarket, subset = train)
qda.class <- predict(qda.fit, Smarket.2005)$class
table(qda.class, Direction.2005)
```

# K-Nearest Neighbors

We will now perform KNN using the `knn()` function, which is part of the
`class` library. This function works rather differently from the other modelfitting
functions that we have encountered thus far. Rather than a two-step
approach in which we first fit the model and then we use the model to make
predictions, `knn()` forms predictions using a single command. The function
requires four inputs.

1. A matrix containing the predictors associated with the training data,labeled `train.X` below.
2. A matrix containing the predictors associated with the data for whichwe wish to make predictions, labeled `test.X` below.
3. A vector containing the class labels for the training observations,labeled `train.Direction` below.
4. A value for K, the number of nearest neighbors to be used by theclassifier.

We use the `cbind()` function, short for column bind, to bind the `Lag1` and
`Lag2` variables together into two matrices, one for the training set and the
other for the test set.

```javascript
library(class)
train.X <- cbind(Lag1, Lag2)[train, ]
test.X <- cbind(Lag1, Lag2)[!train, ]
train.Direction <- Direction[train]

set.seed(1)
knn.pred <- knn(train.X, test.X, train.Direction, k = 1)
table(knn.pred, Direction.2005)

knn.pred <- knn(train.X, test.X, train.Direction, k = 3)
table(knn.pred, Direction.2005)
```
