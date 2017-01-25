---
layout: post
title: Unsupervised Learning
subtitle: The last chapter of the book
bigimg: /img/nl01.jpg
tags: [PCA, clustering]
---

# Lab 1: Principal Components Analysis

In this lab, we perform PCA on the `USArrests` data set, which is part of
the base `R` package. The rows of the data set contain the fifty states, in
alphabetical order.

```r
states <- row.names(USArrests)
states
```

![](/img/nl02.png)

We first briefly examine the data.

```r
library(magrittr)
USArrests %>% apply(2, function(x) c(mean = mean(x),
                                      var = var(x))) %>% round(1)
```

![](/img/nl03.png)

Not surprisingly, the variables also have vastly different variances. The
broad ranges of means and variances among the variables are not surprising:
the `UrbanPop` variable measures the percentage of the population in
each state living in an urban area, which is not a comparable number to
the number of rapes in each state per 100,000 individuals. If we failed to
scale the variables before performing PCA, then most of the principal components
that we observed would be driven by the `Assault` variable, since
it has by far the largest mean and variance. Thus, it is important to standardize
the variables to have mean zero and standard deviation one before
performing PCA.

We now perform principal components analysis using the `prcomp()` function,
which is one of several functions in `R` that perform PCA.

```r
pr.out <- prcomp(USArrests, scale = TRUE)
summary(pr.out)
```

![](/img/nl04.png)

```r
pr.out$rotation
```

![](/img/nl05.png)

# Lab 2: Clustering

***

### K-Means Clustering

The function kmeans() performs K-means clustering in R. We begin with
a simple simulated example in which there truly are two clusters in the
data: the first 25 observations have a mean shift relative to the next 25
observations.

```r
set.seed(2)
x <- matrix(rnorm(50 * 2), ncol = 2)
x[1:25, 1] <- x[1:25, 1] + 3
x[1:25, 2] <- x[1:25, 2] - 4
```

We now perform K-means clustering with K = 2.

```r
km.out <- kmeans(x, 2, nstart = 20)
km.out$cluster
```

![](/img/nl06.png)

The K-means clustering perfectly separated the observations into two clusters
even though we did not supply any group information to `kmeans()`. We
can plot the data, with each observation colored according to its cluster
assignment.

```r
plot(x, col = (km.out$cluster + 1), 
       main = "K-Means Clustering Results with K=2", 
       xlab = "", 
       ylab = "", 
        pch = 20, 
        cex = 2)
```     

![](/img/nl07.png)

To run the `kmeans()` function in `R` with multiple initial cluster assignments,
we use the `nstart` argument. If a value of `nstart` greater than one
is used, then K-means clustering will be performed using multiple random
assignments, and the `kmeans()` function will report only the best results. 
Here we compare using `nstart=1` to `nstart=20`.

```r
library(plyr)
compare <- data.frame(nstart = c(1, 20)) %>% 
                      mlply(kmeans, args = list(x = x, 
                                           centers = 2)) %>% 
                      laply(function(x) x$tot.withinss)      
names(compare) <- c("nstart = 1", "nstart = 20")
compare
```

The result is just like the following table:

| nstart = 1 | nstart = 20 |
|:----------:|:-----------:|
| 128.6066   | 128.6066    |

### Hierarchical Clustering

The `hclust(`) function implements hierarchical clustering in `R`. In the following
example we use the data above to plot the hierarchical
clustering dendrogram using complete, single, and average linkage clustering,
with Euclidean distance as the dissimilarity measure. We begin by
clustering observations using complete linkage. The `dist()` function is used
to compute the 50 × 50 inter-observation Euclidean distance matrix.

```r
hc.complete <- hclust(dist(x), method = "complete")
hc.average <- hclust(dist(x), method = "average")
hc.single <- hclust(dist(x), method = "single")

opar <- par(mfrow = c(1, 3))
plot(hc.complete, main = "Complete Linkage", xlab = "", sub = "", cex = .9)
plot(hc.average, main = "Average Linkage", xlab = "", sub = "", cex = .9)
plot(hc.single, main = "Single Linkage", xlab = "", sub = "", cex = .9)
par(opar)

cutree(hc.complete, 2)
cutree(hc.average, 2)
cutree(hc.single, 2)
```

![](/img/nl08.png)

For this part, the book of <font color='#1E90FF'>an introduction to statistical learning</font> is 
relatively simple, more labs are to be displayed here in my later post.
