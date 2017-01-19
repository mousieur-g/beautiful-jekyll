---
layout: post
title: Classification Problems
subtitle: start to learn in this winter vacation
bigimg: /img/study01.jpg
tags: [Logistic regression, LDA, QDA, KNN]
---

# The Stock Market Data

> We will begin by examining some numerical and graphical summaries of
the `Smarket` data, which is part of the `ISLR` library. This data set consists of
percentage returns for the S&P 500 stock index over 1, 250 days, from the
beginning of 2001 until the end of 2005. For each date, we have recorded
the percentage returns for each of the five previous trading days, `Lag1`
through `Lag5`. We have also recorded `Volume` (the number of shares traded
on the previous day, in billions), Today (the percentage return on the date
in question) and `Direction` (whether the market was `Up` or `Down` on this
date).

> We can use the following R codes to view the data structure

```javascript
library(ISLR)
names(Smarket)
```
