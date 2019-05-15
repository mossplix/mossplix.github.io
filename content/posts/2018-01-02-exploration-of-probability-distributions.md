---
title: "Probability Distributions"
date: "2018-01-02T11:46:37.121Z"
template: "post"
draft: false
slug: "/posts/probability-distributions/"
category: "Data Science"
tags:
  - "probability distributions"
  - "WebPPL"
description: "My probability distribution notes"
---



A probability model is a mathematical model used to fit a random process. Probabilistic models represent general domain knowledge as a probability distribution over all possible states. A model has two parts
     - Random variable 
     - the random variable's Probability Distribution

The random variables can be discrete or continious

Certain probability distributions occur with such regularity in real-life applications that they have been given their own names

I will use WebPPL,  a small probabilistic programming language built on top of a (purely functional) subset of Javascript to explore some of them.



## Gaussian/Normal/Bell Curve Distribution

The normal distribution is the most important distrib- ution in statistics, since it arises naturally in numerous applications. The key reason is that large sums of (small) random variables often turn out to be normally distributed
~~~


viz(Gaussian({mu: 0, sigma: 1}))

~~~

where mu is the mean and sigma the variance

## Binomial Distribution

The bi in binomial is latin for two.  It is used to model a random experiment in which there are only two possible outcomes - success and failure.

 Imagine that you have the seven days of a week, and every day has a sunny element, which is flip(0.2). Now you want an element whose value is the number of sunny days in the week. This can be achieved by using the element Binomial(7, 0.2). The value of this element is the number of trials that come out true, when there are seven total trials and each trial has probability 0.2 of coming out true. You can use it as follows:


In general, Binomial takes two arguments: the number of trials and the probability that each trial comes out true. The definition of Binomial assumes that the trials are independent; the first trial coming out true doesn’t change the probability that the second trial will come out true.



~~~
var bn = function() {
  return binomial(0.2, 7);
};

var bdist = Infer({ model: bn })
viz(bdist)

~~~


Examples of events that can be modelled as bionomial distribution

- The number of heads/tails in a sequence of coin flips
- Vote counts for two different candidates in an election
- The number of male/female employees in a company
- The number of accounts that are in compliance or not in compliance with an accounting procedure
- The number of successful sales calls
- The number of defective products in a production run
- The number of days in a month your company’s computer network experiences a problem

## Bernouli Distribution

This is a special case of Binomial distribution where only one trial is carried out.


## Multinomial  Distribution

This generalisation  of Binomial distribution  whith multiple outcomes.

## Beta Distribution

The beta distribution is conjugate prior to the Bernoulli distribution. This means that if you have a Beta variable and a Bernoulli variable, and the Beta variable represents the probability that the Bernoulli variable comes out true, then conditioning the Beta variable on the outcome of the Bernoulli variable results in another Beta variable.


 It also means that if you have an unknown probability like the bias of a coin that you are estimating by repeated coin flips, then the likelihood induced on the unknown bias by a sequence of coin flips is beta-distributed.

it represents all the possible values of a probability when we don't know what that probability is.

It’s time to look more closely at the two arguments of the Beta element. The first argument, called  , represents an imaginary number of heads you’ve already seen, plus 1. The second, called  , is the same thing for tails. In your example, you’re using Beta(2,5), meaning that you’re imagining having already seen one head and four tails even before learning. (By the way, the reason you add 1 to the number of heads and tails is mathematical convention; it makes the prediction of the next toss simple, as you’ll see in a couple of paragraphs.) This enables you to encode the fact that you have some prior beliefs about the bias even before seeing any data. If you have no prior beliefs, you can use Beta(1,1).

Now, suppose you observe a toss that comes out heads. You increment   by 1 to get the posterior. Similarly, if the toss comes out tails, you increment   by 1. In general, if you observe h heads and t tails, you increment   by h and   by t. So if the prior is Beta(2, 5) and you observe three heads and two tails, the posterior is Beta(5,7). This is why the beta distribution is the conjugate prior of the Bernoulli distribution.
It’s also easy to predict whether the next toss comes out heads when the bias is a beta distribution. The probability of this happening is   / (  +  ), so 5/12 for Beta(5,7).
~~~
var model = function() {
  sample(Beta({a: 6, b: 2}));
};

var dist = Infer({ model: model })
viz(dist)
~~~




## Dirichlet Distribution

The Dirichlet distribution is a generalization of the beta distribution into multiple dimensions. In this case , it is a conjugate prior for the multinomial distribution. i.e when the trial has multiple outcomes.

~~~
var model = function() {
  var x = dirichlet(Vector([.1, .2, .6]));
  return T.sumreduce(T.range(x, 0, 2));
};
var dist = Infer({ model: model })
viz(dist)
~~~




## Uniform Distribution 

 every possible outcome has an equal chance, or likelihood, of occurring

 ~~~
var model = function() {
  return uniform(2,10);
};

var dist = Infer({ model: model })
viz(dist)
~~~


## Pareto/Bradford distribution

In the 1890s, Vilfredo Pareto studied income tax data from several countries. He plotted the number of people earning an income above a certain threshold against the respec- tive threshold on double logarithmic paper and revealed a linear relationship. The corresponding income distribution was more skewed and heavy-tailed than bell-shaped curves: Pareto felt that he had discovered a new type of “universal law” that was the result of underlying economic mechanisms. Since then, Pareto’s discovery has been con- firmed and generalized to the distribution of firm size (Axtell (2001)) and wealth, which also follow Pareto distributions in the upper tail.



## Exponential

The exponential distribution is typically used to model time intervals between “random events”.

- The length of time between telephone calls
- The length of time between arrivals at a service station
- The life time of electronic components, i.e., an inter- failure time

~~~
var model = function() {
  return exponential(2);
};
var dist = Infer({ model: model })
viz(dist)
~~~


## Poisson Distribution


It usually is applicable in situations where random “events” occur at a certain rate over a period of time.

Examples of events that can be modelled as poisson distribution

- The hourly number of customers arriving at a bank
- The daily number of accidents on a particular stretch of highway
- The hourly number of accesses to a particular web server
- The daily number of emergency calls in Dallas
- The number of typos in a book
- The monthly number of employees who had an ab- sence in a large company
- Monthly demands for a particular product

~~~
var model = function() {
  return poisson(4);
};

var dist = Infer({ model: model })
viz(dist)
~~~




## Gamma Distribution

Gamma distribution is a generalization of exponential distribution.

It arises naturally in processes for which the waiting times between Poisson distributed events are relevant.

e.g when we want to know when the rth customer will arrive at the bank

~~~
var model = function() {
  sample(Gamma({shape: 12, scale: 1 / 8}));
};

var dist = Infer({ model: model })
viz(dist)


~~~



## Chi-Square Distribution

 Basically chi-square distribution is the measure of which enables us to find out the degree of degree of discrepancy between the observed and expected frequencies to determine whether the discrepancy between the observed and expected frequencies is due to error of sampling or due to chance.

 he χ2 distribution is used for testing the goodness of fit. It is used for finding association and relation between attributes. It is also used to test the homogeneity of independent estimates of population. Chi-square is computed on the basis of frequencies in a sample and the value of χ2 so obtained is a statistic.



