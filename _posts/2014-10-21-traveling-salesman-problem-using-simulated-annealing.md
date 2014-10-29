---
layout: post
title: Solving Traveling Salesman Problem Using Simulated Annealing
categories: algorithm
comments: true
excerpt:
  Find the minimum cost to visit all cities. This is a classical algorithmic problem, but have you tried solving it using R?
tags: [snippets, R, rmarkdown]
---

The generic Simulated Annealing algorithm, adapted from [Yong Wang](https://www.stat.auckland.ac.nz/~yongwang/)'s slide:

{% gist avansp/aad59611a6bb13f19068 SimulatedAnnealing.R %}


Let $$k$$ random points representing cities generated from uniform random distributions on $$(0,1)^2$$. Distances between cities are measured using the Euclidean distance.

{% highlight r %}
k <- 50             # SET here the number of cities
cities <- cbind(runif(k,0,1), runif(k,0,1))
{% endhighlight %}

Calculate the distance matrix:

{% highlight r %}
dist.cities <- dist(cities, method='euclidean')
{% endhighlight %}

Given a tour from $$c = (c_1, c_2, \ldots, c_k, c_1)$$, define the total cost as the sum:

{% highlight r %}
cost.tour <- function(tour,  # give the tour from c(1) to c(k)
                      d)     # the distance matrix
{
  if( is.null(k <- attr(d,"Size")) ) stop('d must be a distance matrix')
  if( length(setdiff(unique(tour), 1:k)) > 0 ) stop('invalid tour')
  tour.xy <- cbind(tour, c(tour[-1], tour[1]))
  sum(as.matrix(d)[tour.xy])
}
{% endhighlight %}

Let's check it with initial random tour:

{% highlight r %}
plot.tour <- function(tour, cities, d, ...)
{
  plot(cities, pch=16, asp=1, xlab='x', ylab='y',
       main = sprintf("D = %.2f", cost.tour(tour, d)))
  lines( cities[c(tour, tour[1]),], col="red" )
}
init.tour <- sample(1:k)
plot.tour(init.tour, cities, dist.cities)
{% endhighlight %}

![center](/../images/2014-10-21-traveling-salesman-problem-using-simulated-annealing/unnamed-chunk-6.png)

The Simulated Annealing solution to the TSP is performed by choosing randomly two cities $$i$$ and $$j$$. The block between $$c_i$$ to $$c_j$$ is then reversed. For example, if the current tour for $$i<j$$ is

$$c = (c_1,\ldots,c_{i-1},c_i,c_{i+1},\ldots,c_{j-1},c_j,c_{j+1},\ldots,c_k)$$

then the next tour is

$$
\begin{align*}
c &= (c_j,c_{j-1},\ldots,c_{i+1},c_{i},c_{j+1},\ldots,c_{k},c_1,\ldots,c_{i-1}) \\
  &= (c_1,\ldots,c_{i-1},c_j,c_{j-1},\ldots,c_{i+1},c_{i},c_{j+1},\ldots,c_{k})
\end{align*}
$$

For $$i>j$$, if the current tour is

$$c = (c_1,\ldots,c_{j-1},c_j,c_{j+1},\ldots,c_{i-1},c_i,c_{i+1},\ldots,c_k)$$

the solution is

$$
\begin{align*}
c &= (c_j,c_{j-1},\ldots,c_1,c_k,\ldots,c_{i+1},c_i,c_{j+1},\ldots,c_{i-1}) \\
  &= (c_1,c_k,\ldots,c_{i+1},c_i,c_{j+1},\ldots,c_{i-1},c_j,c_{j-1},\ldots,c_2)
\end{align*}
$$

*Note that first the $$j$$th city is brought at the first city in the tour in the first solution. The second solution is just to ensure that the first city to start is always at $$c_1$$ though the cost of TSP will be identical no matter where you start iff the route is equivalent.*

This is going to be the `walk.fun` function:

{% highlight r %}
walk.tsp <- function(tour)
{
  k <- length(tour)

  ## get any two distinct cities
  ij <- sample(k, size=2, replace=FALSE)
  next.tour <- tour
  next.tour[c(ij[1]:ij[2])] <- rev(next.tour[c(ij[1]:ij[2])])
  next.tour
}
{% endhighlight %}

Let's try the SA algorithm (**don't forget to invert the cost!!**):

{% highlight r %}
tsp.basic <- SA(init.tour,
                cost.fun = function(x) -cost.tour(x,dist.cities),
                walk.fun = walk.tsp,
                n = 10000)
{% endhighlight %}



{% highlight text %}
## Passed iteration 1000
## Passed iteration 2000
## Passed iteration 3000
## Passed iteration 4000
## Passed iteration 5000
## Passed iteration 6000
## Passed iteration 7000
## Passed iteration 8000
## Passed iteration 9000
## Passed iteration 10000
{% endhighlight %}

The cost plot

{% highlight r %}
plot(-tsp.basic$cost, type='l', xlab="Iteration", ylab="Traveling Cost")
{% endhighlight %}

![center](/../images/2014-10-21-traveling-salesman-problem-using-simulated-annealing/unnamed-chunk-9.png)

The four representative routes:

{% highlight r %}
par(mfrow=c(2,2))
iter <- seq(1,length(tsp.basic$cost),len=4)
plot.tour(tsp.basic$chain[iter[1],], cities, dist.cities)
plot.tour(tsp.basic$chain[iter[2],], cities, dist.cities)
plot.tour(tsp.basic$chain[iter[3],], cities, dist.cities)
plot.tour(tsp.basic$chain[iter[4],], cities, dist.cities)
{% endhighlight %}

![center](/../images/2014-10-21-traveling-salesman-problem-using-simulated-annealing/unnamed-chunk-10.png)
