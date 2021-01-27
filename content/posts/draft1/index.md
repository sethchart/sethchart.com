---
title: "Difference Of Means"
date: 2020-09-23T08:21:08-04:00
draft: true
math: true
---

## The Problem

Suppose that you are handed a spreadsheet containing a random sample of the employees at your company along with their salary and gender. You are asked to answer the following question.

> Is there a gender wage gap at your company?

 For the sake of exposition we treat gender as a binary variable taking values Male and Female.

## Recasting the Problem

We are being asked to compare the salaries of two groups Female employees and Male employees. It is reasonable to aggregate salaries for each group using the mean since this statistic describes the salary that each member of the group would receive if the total salary of their group were redistributed equally and it is sensitive to extreme salaries. It might also be reasonable to aggregate salaries using the median, however this would not be sensitive to extremely large or small salaries, which are of interest when considering the existence of disparities between the groups.

These considerations lead us to recast the original question as the more mathematically precise question below.

> Is there as significant difference between the mean salary of the Female group and the mean salary of the Male group?

### Standardizing the Difference

When investigating any statistic it is useful to standardize the spread so that values can be interpreted easily without relying unnecessarily on the context. In our case the context specific values are measured in dollars and on a scale that depends on the compensation that your company provides. It would be easier to leverage our statistical intuition if values were measured in unitless standard deviations from the mean. If we were interested in inspecting salaries directly, it would be appropriate to convert salaries to z-scores. However, we are interested in examining the difference in means between the two groups. In this case it is appropriate to convert the difference in means to a t-score. A t-score is constructed in much the same way as a z-score, the statistic measured in context specific units is divided by an unbiased estimate of the population standard deviation, thereby converting to unitless standard deviations. 

For a difference of means the appropriate unbiased estimate of the population standard deviation is the pooled standard deviation. For the reader who enjoys mathematical formulae, we describe the pooled standard deviation below.

#### Pooled Standard Deviation

### Defining Significance

In the restated question above, one word was left undefined. That word is "significant". One very common way to define what we mean by significant is as follows. 
 
 1. Define a threshold probability that captures our idea of an improbable event. 
 2. Find the probability that we would see a standardized difference as large or larger than the one we have observed by random chance.
 3. If the probability from Step 2 is smaller than the probability from Step 1, then we say that the difference is significant. 

Most of the difficulty in this plan is hiding in Step 2. In order to complete Step 2, we will need to make some assumptions about our data and ensure that those assumptions are justified.

### Computing Probabilities

One way or another we need a probability distribution for our standardized difference that will allow us to compute the probability of getting a value as large or larger than the one that we have by random chance.

When we say by random chance we really mean:

> Assuming that there is no difference between the mean wages of Female employees and Male employees.

This is our null hypothesis. In the end we wan to show that, if the assumption above holds, it is improbable that we would see a wage gap as large or larger than the one that we have observed.

Alright, so we are assuming that the true expected value of our standardized difference is zero. That means that our probability distribution should have mean zero.

We also have also standardized the difference of means so that they have standard deviation 1. Therefore, our probability distribution should have standard deviation 1.

#### The t-distribution


