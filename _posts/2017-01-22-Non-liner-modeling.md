---
layout: post
title: Non-Linear Modeling
subtitle: Bon courage
bigimg: /img/nlm01.jpeg
tags: [polynomial regression, splines, GAM]
---

non-linear fitting procedures discussed can be easily implemented in `R`. We
begin by loading the `ISLR` library, which contains the data.

```r
library(ISLR)
attach(Wage)
```

# Polynomial Regression

We first fit the model using the following command:

```r
fit <- lm(wage ~ poly(age, 4), data = Wage)
coef(summary(fit))
```

![](/img/nlm02.png)

This syntax fits a linear model, using the `lm()` function, in order to predict
`wage` using a fourth-degree polynomial in `age`: `poly(age,4)`. The `poly()` command
allows us to avoid having to write out a long formula with powers
of `age`. The function returns a matrix whose columns are a basis of or-
thogonal polynomials, which essentially means that each column is a linear
combination of the variables `age`, `age^2`, `age^3` and `age^4`.

However, we can also use `poly()` to obtain `age`, `age^2`, `age^3` and `age^4`
directly, if we prefer. We can do this by using the `raw=TRUE` argument to
the `poly()` function. Later we see that this does not affect the model in a
meaningful way — though the choice of basis clearly affects the coefficient
estimates, it does not affect the fitted values obtained.

```r
fit2 <- lm(wage ~ poly(age, 4, raw = T), data = Wage)
coef(summary(fit2))
```

![](/img/nlm03.png)

We now create a grid of values for `age` at which we want predictions, and
then call the generic `predict()` function, specifying that we want standard
errors as well.

```r
agelims <- range(age)
age.grid <- seq(from = agelims[1], to = agelims[2])

preds <- predict(fit, newdata = list(age = age.grid), se = TRUE)
se.bands <- cbind(preds$fit + 2 * preds$se.fit, preds$fit - 2 * preds$se.fit)
```

Finally, we plot the data and add the fit from the degree-4 polynomial.

```r
opar <- par(mfrow = c(1, 2), mar = c(4.5, 4.5, 1, 1), oma = c(0 ,0 ,4 ,0))
plot(age, wage, xlim = agelims, cex = .5, col = "darkgrey")
title("Degree-4 Polynomial", outer = T)
lines(age.grid, preds$fit, lwd = 2, col = "blue")
matlines(age.grid, se.bands, lwd = 1, col = "blue", lty = 3)
```

Here the `mar` and `oma` arguments to `par()` allow us to control the margins
of the plot, and the `title()` function creates a figure title that spans both
subplots.

In performing a polynomial regression we must decide on the degree of
the polynomial to use. One way to do this is by using hypothesis tests. We
now fit models ranging from linear to a degree-5 polynomial and seek to
determine the simplest model which is sufficient to explain the relationship
between `wage` and `age`. We use the `anova()` function, which performs an
analysis of variance (ANOVA, using an F-test) in order to test the null
hypothesis that a model M1 is sufficient to explain the data against the
alternative hypothesis that a more complex modelM2 is required. In order
to use the `anova()` function, M1 and M2 must be nested models: the
predictors in M1 must be a subset of the predictors in M2. In this case,
we fit five different models and sequentially compare the simpler model to
the more complex model.

```r
fit.1 <- lm(wage ~ age, data = Wage)
fit.2 <- lm(wage ~ poly(age, 2), data = Wage)
fit.3 <- lm(wage ~ poly(age, 3), data = Wage)
fit.4 <- lm(wage ~ poly(age, 4), data = Wage)
fit.5 <- lm(wage ~ poly(age, 5), data = Wage)
anova(fit.1, fit.2, fit.3, fit.4, fit.5)
```

![](/img/nlm04.png)

However, the ANOVA method works whether or not we used orthogonal
polynomials; it also works when we have other terms in the model as well.
For example, we can use `anova()` to compare these three models:

```r
fit.1 <- lm(wage ~ education + age, data = Wage)
fit.2 <- lm(wage ~ education + poly(age, 2), data = Wage)
fit.3 <- lm(wage ~ education + poly(age, 3), data = Wage)
anova(fit.1, fit.2, fit.3)
```

Next we consider the task of predicting whether an individual earns more
than $250, 000 per year. We proceed much as before, except that first we
create the appropriate response vector, and then apply the `glm()` function
using `family="binomial"` in order to fit a polynomial logistic regression
model.

```r
fit <- glm(I(wage > 250) ~ poly(age, 4), data = Wage, family = binomial)
```

Note that we again use the wrapper `I()` to create this binary response
variable on the fly. The expression `wage>250` evaluates to a logical variable
containing `TRUEs` and `FALSEs`, which `glm()` coerces to binary by setting the
`TRUEs` to 1 and the `FALSEs` to 0.

Once again, we make predictions using the `predict()` function.

```r
preds <- predict(fit, newdata = list(age = age.grid), se = T)
pfit <- exp(preds$fit) / (1 + exp(preds$fit))

se.bands.logit <- cbind(preds$fit + 2 * preds$se.fit, preds$fit - 2 * preds$se.fit)
se.bands <- exp(se.bands.logit) / (1 + exp(se.bands.logit))
```
Note that we could have directly computed the probabilities by selecting
the `type="response"` option in the `predict()` function.

```r
preds <- predict(fit, newdata = list(age = age.grid), se = T, type = "response")
```

Finally, we can make plots as follows:

```r
plot(age, I(wage > 250), xlim = agelims, type = "n", ylim = c(0, .2))
points(jitter(age), I((wage > 250) / 5), cex = .5, pch = "|", col = "darkgrey")
lines(age.grid, pfit, lwd = 2, col = "blue")
matlines(age.grid, se.bands, lwd = 1, col = "blue", lty = 3)
par(opar)
```

![](/img/nlm05.png)

# Splines

In order to fit regression splines in `R`, we use the `splines` library. The `bs()` 
function generates the entire matrix of basis functions for splines with the 
specified set of knots. By default, cubic splines are produced. Fitting `wage` to 
`age` using a regression spline is simple:

```r
library(splines)
fit <- lm(wage ~ bs(age, knots = c(25, 40, 60) ), data = Wage)
pred <- predict(fit, newdata = list(age = age.grid), se = T)

plot(age, wage, col = "gray")
lines(age.grid, pred$fit, lwd = 2)
lines(age.grid, pred$fit + 2 * pred$se, lty = "dashed")
lines(age.grid, pred$fit - 2 * pred$se, lty = "dashed")
```

![](/img/nlm06.png)

In order to instead fit a natural spline, we use the `ns()` function. Here
we fit a natural spline with four degrees of freedom.

```r
fit2 <- lm(wage ~ ns(age, df = 4), data = Wage)
pred2 <- predict(fit2, newdata = list(age = age.grid), se = T)
lines(age.grid, pred2$fit, col = "red", lwd = 2)
```

![](/img/nlm07.png)

As with the `bs()` function, we could instead specify the knots directly using
the `knots` option.

In order to fit a smoothing spline, we use the `smooth.spline()` function.

```r
plot(age, wage, xlim = agelims, cex = .5, col = "darkgrey")
title("Smoothing Spline")
fit <- smooth.spline(age, wage, df = 16)
fit2 <- smooth.spline(age, wage, cv = TRUE)
fit2$df

lines(fit, col = "red", lwd = 2)
lines(fit2, col = "blue", lwd = 2)
legend("topright", legend = c("16 DF", "6.8 DF"), col = c("red", "blue"), 
                      lty = 1, lwd = 2, cex = .8)
```

![](/img/nlm08.png)

Notice that in the first call to `smooth.spline()`, we specified `df=16`. The
function then determines which value of λ leads to 16 degrees of freedom. In
the second call to `smooth.spline()`, we select the smoothness level by crossvalidation;
this results in a value of λ that yields 6.8 degrees of freedom.

In order to perform local regression, we use the `loess()` function.

```r
plot(age, wage, xlim = agelims, cex = .5, col = "darkgrey")
title("Local Regression")
fit <- loess(wage ~ age, span = .2, data = Wage)
fit2 <- loess(wage ~ age, span = .5, data = Wage)
lines(age.grid, predict(fit, data.frame(age = age.grid)), col = "red", lwd = 2)
lines(age.grid, predict(fit2, data.frame(age = age.grid)), col = "blue", lwd = 2)
legend("topright", legend = c("Span=0.2", "Span=0.5") ,
                      col = c("red", "blue"), lty = 1, lwd = 2, cex = .8)
```

![](/img/nlm09.png)

Here we have performed local linear regression using spans of 0.2 and 0.5:
that is, each neighborhood consists of 20% or 50% of the observations. The
larger the span, the smoother the fit. The `locfit` library can also be used
for fitting local regression models in `R`.

# GAMs

We now fit a GAM to predict `wage` using natural spline functions of `year`
and `age`, treating `education` as a qualitative predictor. Since
this is just a big linear regression model using an appropriate choice of
basis functions, we can simply do this using the `lm()` function.

```r
gam1 <- lm(wage ~ ns(year, 4) + ns(age, 5) + education, data = Wage)
```

In order to fit more general sorts of GAMs, using smoothing splines
or other components that cannot be expressed in terms of basis functions
and then fit using least squares regression, we will need to use the `gam`
library in `R`.

```r
library(gam)
gam.m3 <- gam(wage ~ s(year, 4) + s(age, 5) + education, data = Wage)
```

In order to produce figures, we simply call the `plot()` function:

```r
opar <- par(mfrow = c(1, 3))
plot(gam.m3, se = TRUE, col = "blue")
```

![](/img/nlm10.png)

```r
plot.gam(gam1, se = TRUE, col = "red")
```

![](/img/nlm11.png)

In these plots, the function of `year` looks rather linear. We can perform a
series of ANOVA tests in order to determine which of these three models is
best: a GAM that excludes `year` (M1), a GAM that uses a linear function
of `year` (M2), or a GAM that uses a spline function of `year` (M3).

```r
gam.m1 <- gam(wage ~ s(age, 5) + education, data = Wage)
gam.m2 <- gam(wage ~ year + s(age, 5) + education, data = Wage)
anova(gam.m1, gam.m2, gam.m3, test = "F")
```

![](/img/nlm12.png)

We can also use local regression fits as building blocks in a GAM, using
the `lo()` function.

```r
gam.lo <- gam(wage ~ s(year, df = 4) + lo(age, span = 0.7) + education, data = Wage)
plot.gam(gam.lo, se = TRUE, col = "green")
```

Here we have used local regression for the `age` term, with a span of 0.7.
We can also use the `lo()` function to create interactions before calling the
`gam()` function. For example,

```r
gam.lo.i <- gam(wage ~ lo(year, age, span = 0.5) + education, data = Wage)
```

