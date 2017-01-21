---
layout: post
title: Linear Model Selection and Regularization
subtitle: An introduction to statistical learning
bigimg: /img/LMSR01.jpg
tags: [model selection, ridge regression, lasso, PCR, PLS]
---

# Lab 1: Subset Selection Methods

***

### Best Subset Selection

Here we apply the best subset selection approach to the `Hitters` data. We
wish to predict a baseball player’s `Salary` on the basis of various statistics
associated with performance in the previous year.

First of all, we note that the `Salary` variable is missing for some of the
players. The `is.na()` function can be used to identify the missing observations.
It returns a vector of the same length as the input vector, with a `TRUE`
for any elements that are missing, and a `FALSE` for non-missing elements.
The `sum()` function can then be used to count all of the missing elements.

```r
library(ISLR)
fix(Hitters)
names(Hitters)
dim(Hitters)
sum(is.na(Hitters$Salary))
```

Hence we see that `Salary` is missing for 59 players. The `na.omit()` function
removes all of the rows that have missing values in any variable.

```r
Hitters <- na.omit(Hitters)
```

The `regsubsets()` function (part of the leaps library) performs best subset selection 
by identifying the best model that contains a given number of predictors, where best 
is quantified using RSS. The syntax is the same as for `lm()`. The `summary()` command 
outputs the best set of variables for each model size.

```r
library(leaps)
regfit.full <- regsubsets(Salary ~ ., Hitters)
summary(regfit.full)
```

An asterisk indicates that a given variable is included in the corresponding
model. For instance, this output indicates that the best two-variable model
contains only `Hits` and `CRBI`. By default, `regsubsets()` only reports results
up to the best eight-variable model. But the `nvmax` option can be used
in order to return as many variables as are desired. Here we fit up to a
nineteen-variable model.

```r
regfit.full <- regsubsets(Salary ~ ., data = Hitters, nvmax = 19)
reg.summary <- summary(regfit.full)
```

The `summary()` function also returns R2, RSS, adjusted R2, Cp, and BIC.
We can examine these to try to select the best overall model.

```r
names(reg.summary)
```

![](/img/LMSR02.png)

For instance, we see that the R2 statistic increases from 32%, when only
one variable is included in the model, to almost 55%, when all variables
are included. As expected, the R2 statistic increases monotonically as more
variables are included.

```r
reg.summary$rsq
```

![](/img/LMSR03.png)

Plotting RSS, adjusted R2, Cp, and BIC for all of the models at once will
help us decide which model to select. Note the `type="l"` option tells `R` to
connect the plotted points with lines.

```r
opar <- par(mfrow = c(2, 2))
plot(reg.summary$rss, xlab = "Number of Variables", ylab = "RSS", type = "l")
plot(reg.summary$adjr2, xlab = "Number of Variables", ylab = "Adjusted RSq", type = "l")
```

The `points()` command works like the `plot()` command, except that it
puts points on a plot that has already been created, instead of creating a
new plot. The `which.max()` function can be used to identify the location of
the maximum point of a vector. We will now plot a red dot to indicate the
model with the largest adjusted R2 statistic.

```r
which.max(reg.summary$adjr2)
points(11, reg.summary$adjr2[11], col = "red", cex = 2, pch = 20)
```

In a similar fashion we can plot the Cp and BIC statistics, and indicate the
models with the smallest statistic using `which.min()`.

```r
plot(reg.summary$cp, xlab = "Number of Variables", ylab = "Cp", type = "l")
which.min(reg.summary$cp)
points(10, reg.summary$cp[10], col = "red", cex = 2, pch = 20)

plot(reg.summary$bic, xlab = "Number of Variables", ylab = "BIC", type = "l")
which.min reg.summary$bic)
points(6, reg.summary$bic[6], col = "red", cex = 2, pch = 20)
```

![](/img/LMSR04.png)

The `regsubsets()` function has a built-in `plot()` command which can
be used to display the selected variables for the best model with a given
number of predictors, ranked according to the BIC, Cp, adjusted R2, or
AIC. To find out more about this function, type `?plot.regsubsets`.

```r
plot(regfit.full, scale = "r2")
plot(regfit.full,scale = "bic")
```

The top row of each plot contains a black square for each variable selected
according to the optimal model associated with that statistic. For instance,
we see that several models share a BIC close to −150. However, the model
with the lowest BIC is the six-variable model that contains only `AtBat`,
`Hits`, `Walks`, `CRBI`, `DivisionW`, and `PutOuts`. We can use the `coef()` function
to see the coefficient estimates associated with this model.

### Forward and Backward Stepwise Selection

We can also use the `regsubsets()` function to perform forward stepwise
or backward stepwise selection, using the argument `method="forward"` or
`method="backward"`.

```r
regfit.fwd <- regsubsets(Salary ~ ., data = Hitters , nvmax = 19, method = "forward")
summary(regfit.fwd)

regfit.bwd <- regsubsets(Salary ~ ., data = Hitters , nvmax = 19, method = "backward")
summary(regfit.bwd)
```

For instance, we see that using forward stepwise selection, the best onevariable
model contains only `CRBI`, and the best two-variable model additionally
includes `Hits`. For this data, the best one-variable through sixvariable
models are each identical for best subset and forward selection.
However, the best seven-variable models identified by forward stepwise selection,
backward stepwise selection, and best subset selection are different.

### Choosing Among Models using the Validation Set Approach and Cross-Validation

We just saw that it is possible to choose among a set of models of different
sizes using Cp, BIC, and adjusted R2. We will now consider how to do this
using the validation set and cross-validation approaches.

In order for these approaches to yield accurate estimates of the test
error, we must use only the training observations to perform all aspects of
model-fitting — including variable selection. Therefore, the determination
of which model of a given size is best must be made using only the training
observations. This point is subtle but important. If the full data set is used
to perform the best subset selection step, the validation set errors and
cross-validation errors that we obtain will not be accurate estimates of the
test error.

In order to use the validation set approach, we begin by splitting the
observations into a training set and a test set. We do this by creating
a random vector, `train`, of elements equal to `TRUE` if the corresponding
observation is in the training set, and `FALSE` otherwise. The vector `test` has
a `TRUE` if the observation is in the test set, and a `FALSE` otherwise. Note the
! in the command to create `test` causes `TRUEs` to be switched to `FALSEs` and
vice versa. We also set a random seed so that the user will obtain the same
training set / test set split.

```r
set.seed(1)
train <- sample(c(TRUE, FALSE), nrow(Hitters), rep = TRUE)
test <- (!train)
```

Now, we apply `regsubsets()` to the training set in order to perform best
subset selection.

```r
regfit.best <- regsubsets(Salary ~ ., data = Hitters[train, ], nvmax = 19)
```

Notice that we subset the `Hitters` data frame directly in the call in order
to access only the training subset of the data, using the expression
`Hitters[train,]`. We now compute the validation set error for the best
model of each model size. We first make a model matrix from the test
data.

```r
test.mat <- model.matrix(Salary ~ ., data = Hitters[test, ])
```

The `model.matrix()` function is used in many regression packages for building
an “X” matrix from data. Now we run a loop, and for each size i, we
extract the coefficients from `regfit.best` for the best model of that size,
multiply them into the appropriate columns of the test model matrix to
form the predictions, and compute the test MSE.

```r
val.errors <- rep(NA, 19)

for(i in 1:19) 
{
  coefi <- coef(regfit.best, id = i)
  pred <- test.mat[ ,names(coefi)] %*% coefi
  val.errors[i] <- mean((Hitters$Salary[test] - pred)^2)
}
```

We find that the best model is the one that contains ten variables.

```r
library(migrittr)
val.errors %>% which.min %>% coef(regfit.best, .)
```

This was a little tedious, partly because there is no `predict()` method
for `regsubsets()`. Since we will be using this function again, we can capture
our steps above and write our own predict method.

```r
predict.regsubsets <- function(object, newdata, id, ...) 
{
  form <- as.formula(object$call[[2]])
  mat <- model.matrix(form, newdata)
  coefi <- coef(object, id = id)
  xvars <- names(coefi)
  mat[ ,xvars] %*% coefi
}
```

Our function pretty much mimics what we did above. The only complex
part is how we extracted the formula used in the call to `regsubsets()`. We
demonstrate how we use this function below, when we do cross-validation.

Finally, we perform best subset selection on the full data set, and select
the best ten-variable model. It is important that we make use of the full
data set in order to obtain more accurate coefficient estimates. Note that
we perform best subset selection on the full data set and select the best tenvariable
model, rather than simply using the variables that were obtained
from the training set, because the best ten-variable model on the full data
set may differ from the corresponding model on the training set.

```r
regfit.best <- regsubsets(Salary ~ ., data = Hitters, nvmax = 19)
coef(regfit.best, 10)
```

In fact, we see that the best ten-variable model on the full data set has a
different set of variables than the best ten-variable model on the training
set.

We now try to choose among the models of different sizes using crossvalidation.
This approach is somewhat involved, as we must perform best
subset selection within each of the k training sets. Despite this, we see that
with its clever subsetting syntax, R makes this job quite easy. First, we
create a vector that allocates each observation to one of k = 10 folds, and
we create a matrix in which we will store the results.

```r
k <- 10
set.seed(1)
folds <- sample(1:k, nrow(Hitters), replace = TRUE)
cv.errors <- matrix (NA, k, 19, dimnames = list(NULL , paste(1:19)))
```

Now we write a for loop that performs cross-validation. In the jth fold, the
elements of `folds` that equal j are in the test set, and the remainder are in
the training set. We make our predictions for each model size (using our
new `predict()` method), compute the test errors on the appropriate subset,
and store them in the appropriate slot in the matrix `cv.errors`.

```r
for(j in 1:k)
{
  best.fit <- regsubsets(Salary ~ ., data = Hitters[folds != j, ], nvmax = 19)
  for(i in 1:19) 
  {
    pred <- predict(best.fit, Hitters[folds == j, ], id = i)
    cv.errors[j, i] <- mean((Hitters$Salary[folds == j] - pred)^2)
  }
}
```

This has given us a 10×19 matrix, of which the (i, j)th element corresponds
to the test MSE for the ith cross-validation fold for the best j-variable
model. We use the `apply()` function to average over the columns of this
matrix in order to obtain a vector for which the jth element is the crossvalidation
error for the j-variable model.

```r
mean.cv.errors <- apply(cv.errors, 2, mean)
mean.cv.errors
par(mfrow = c(1, 1))
plot(mean.cv.errors, type = "b")
```

![](/img/LMSR05.png)

We see that cross-validation selects an eleven-variable model. We now perform
best subset selection on the full data set in order to obtain the elevenvariable
model.

```r
reg.best <- regsubsets(Salary ~ ., data = Hitters, nvmax = 19)
coef(reg.best, 11)
```

# Lab 2: Ridge Regression and the Lasso

We will use the `glmnet` package in order to perform ridge regression and
the lasso. The main function in this package is `glmnet()`, which can be used
to fit ridge regression models, lasso models, and more. This function has
slightly different syntax from other model-fitting functions that we have
encountered thus far in this book. In particular, we must pass in an `x`
matrix as well as a `y` vector, and we do not use the `y ∼ x` syntax. We will
now perform ridge regression and the lasso in order to predict `Salary` on
the `Hitters` data. Before proceeding ensure that the missing values have
been removed from the data.

```r
x <- model.matrix(Salary ~ ., Hitters)[ , -1]
y <- Hitters$Salary
```

The `model.matrix()` function is particularly useful for creating x; not only
does it produce a matrix corresponding to the 19 predictors but it also
automatically transforms any qualitative variables into dummy variables.
The latter property is important because `glmnet()` can only take numerical,
quantitative inputs.

### Ridge Regression

The `glmnet()` function has an `alpha` argument that determines what type
of model is fit. If `alpha=0` then a ridge regression model is fit, and if `alpha=1`
then a lasso model is fit. We first fit a ridge regression model.

```r
library(glmnet)
grid <- 10 ^ seq(10, -2, length = 100)
ridge.mod <- glmnet(x, y, alpha = 0, lambda = grid)
```

Associated with each value of λ is a vector of ridge regression coefficients,
stored in a matrix that can be accessed by `coef()`. In this case, it is a 20×100 matrix, 
with 20 rows (one for each predictor, plus an intercept) and 100 columns (one for each value of λ).

```r
ridge.mod$lambda[50]
coef(ridge.mod)[ , 50]
```

We can use the `predict()` function for a number of purposes. For instance,
we can obtain the ridge regression coefficients for a new value of λ, say 50:

```r
predict(ridge.mod, s = 50, type = "coefficients")[1:20, ]
```

We first set a random seed so that the results obtained will be reproducible.

```r
set.seed(1)
train <- sample(1:nrow(x), nrow(x)/2)
test <- (-train)
y.test <- y[test]
```

Next we fit a ridge regression model on the training set, and evaluate
its MSE on the test set, using λ = 4. Note the use of the `predict()`
function again. This time we get predictions for a test set, by replacing
`type="coefficients"` with the `newx` argument.

```r
ridge.mod <- glmnet(x[train, ], y[train], alpha = 0, lambda = grid, thresh = 1e-12)
ridge.pred <- predict(ridge.mod, s=4, newx = x[test, ])
mean((ridge.pred - y.test)^2)
```

In general, instead of arbitrarily choosing λ = 4, it would be better to
use cross-validation to choose the tuning parameter λ. We can do this using
the built-in cross-validation function, `cv.glmnet()`. By default, the function
performs ten-fold cross-validation, though this can be changed using the
argument `folds`. Note that we set a random seed first so our results will be
reproducible, since the choice of the cross-validation folds is random.

```r
set.seed(1)
cv.out <- cv.glmnet(x[train, ], y[train], alpha = 0)
plot(cv.out)

bestlam <- cv.out$lambda.min
bestlam
```

![](/img/LMSR06.png)

Finally, we refit our ridge regression model on the full data set,
using the value of λ chosen by cross-validation, and examine the coefficient
estimates.

```r
out <- glmnet(x, y, alpha = 0)
predict(out, type = "coefficients", s = bestlam)[1:20, ] 
```

As expected, none of the coefficients are zero — ridge regression does not
perform variable selection!
