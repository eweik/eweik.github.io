---
layout: post
title: Gaussian Process Regression
data: 2018-08-01 12:00:00 
category: 
- introduction
tags: 
- introduction
---

An introduction to gaussian process regression.

In this post, I'm going to try and introduce, in a friendly manner, gaussian process regression (GPR). Gaussian process regression is a powerful all-purpose regression method. Some of its applications are in Brownian motion, which describes the motion of particle in a fluid, and in geostatistics, where we're given an incomplete set of 2D points that's supposed to span some space and GPR is used to estimate the gaps between our observations to fill in the rest of the terrain. 

Some concepts you should know about are the multivariate Gaussian distribution and Bayesian regression. I’ll review them here, but it will only be brief.

<br>

The main mathematical structure behind GPR is the **multivariate Gaussian distribution**, which is a generalization of the Gaussian (Normal) distribution to multiple dimensions. Two important properties of multivariate Gaussians to know for GPR is the _conditioning property_ and the _additive property_. I'll first show the conditioning property. If we write our random vector $$ x \sim \mathcal{N}( \mu, \Sigma ) \in {\rm I\!R}^{n} $$ as

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
$$ x = \begin{bmatrix} x_a \\ x_b \end{bmatrix} $$ where $$ x_a = \begin{bmatrix} x_1 \\ . \\ . \\ x_k \end{bmatrix} $$ and $$ x_b = \begin{bmatrix} x_{k+1} \\ . \\ . \\ x_n \end{bmatrix} $$, then

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
  $$ \mu = \begin{bmatrix} \mu_a \\ \mu_b \end{bmatrix}$$ and $$\Sigma =  \begin{bmatrix} \Sigma_{aa} & \Sigma_{ab} \\ \Sigma_{ba} & \Sigma_{bb} \end{bmatrix} $$.

The conditioning property says that the distribution of $$x_a$$ given (conditional on) $$x_b$$ is also multivariate Gaussian! It's distribution is given by

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
$$x_a | x_b \sim \mathcal{N}( \mu_a + \Sigma_{ab} \Sigma_{bb}^{-1} (x_b - \mu_b),$$ $$\Sigma_{aa} - \Sigma_{ab}\Sigma_{bb}^{-1}\Sigma_{ba})$$,

The additive property says that if $$ x \sim \mathcal{N}( \mu_x, \Sigma_x ) $$ and $$ y \sim \mathcal{N}( \mu_y, \Sigma_y ), $$ then 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
$$ x + y \sim \mathcal{N}( \mu_x + \mu_y, \Sigma_x + \Sigma_y ) $$

These are a very intuitive results (although the formula for the conditional distribution is cumbersome) that will be very useful when we talk about GPR more.

<br>

The next concept that's important is **Bayesian linear regression**. Without getting too involved in the details of Bayesian inference, some important points about it is that you can _use your prior knowledge_ of the dataset to help your predictions and you _get a distribution on the predicted value_! This is in contrast to other methods which return a single value with no account of the possible uncertainty in that value (these other methods use _Maximum Likelihood estimation_, which normal linear regression does). Bayesian linear regression uses _Maximum a Posteriori estimation_ for the full distribution of the output, called the posterior distribution. Figure 1 shows the difference between these two methods.

<p align="center">
    <img src="//raw.githubusercontent.com/eweik/eweik.github.io/master/images/gaussian-process-regression/regression1.png" width="600">
</p>
_Figure 1_: Bayesian and Classical (Frequentist) predictions on linear observations. Both are very similar, but one advantage of the Bayesian approach is that it gives the distribution of our prediction. In the figure, the shaded region represents 2 standard deviations above and below the prediction.

<br>

# Gaussian Processes
Before we get to Gaussian Process regression, I'm going to introduce Gaussian processes (GPs).

**Gaussian processes** are formally defined a set of random variables $$ \{ f(x) : x \in X \} $$, indexed by elements $$ x $$ (normally time or space) from some index set $$ X $$, such that any finite subset of this set $$ \{ f(x_1),...,f(x_n) \} $$ is multivariate Gaussian distributed.

<br>

These sets can be infinitely sized (think of the index variable - time or space - going on infinitely) and therefore we can think of Gaussian Processes as infinite dimensional extensions of the multivariate Gaussian distribution. But, in practice, we'll always deal with finite sized sets and we can treat them just as we would multivariate Gaussians:

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
$$ f( x ) \sim \mathcal{N} ( 0, k( x, x ) ) $$ 

Thinking of Gaussian Processes as infinite dimensional extensions of multivariate Gaussians allows us see them as distributions over random functions. This distribution is specified by a mean function $$ m( \cdot ) $$ and a covariance function $$ k( \cdot, \cdot ) .$$ So another way to denote $$ f $$ is as

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
$$ f( \cdot ) \sim \mathcal{GP} ( m( \cdot ), k( \cdot, \cdot ) ) $$ 

Just for preciseness, $$ m( \cdot ) $$ must be a real function and $$ k( \cdot, \cdot ) $$ must be a valid kernel function (which means being positive semidefinite) to have a Gaussian Process.

<br>

So, in the same way that we can sample a random vector from a multivariate Normal distribution, we can sample a random function from a Gaussian distribution. Note the difference here: a vector is finite-sized, but a function is a mapping with a potentially infinite size codomain. And just like the types of vectors that we sample from a multivariate Normal distribution are determined by its mean vector and covariance matrix, the types of functions we sample from a Gaussian Process are determined by the mean function $$ m( \cdot ) $$ and the covariance function $$ k( \cdot, \cdot ) .$$

<br>

The covariance function $$ k( \cdot, \cdot ) $$ is much more interesting than the mean function $$ m( \cdot ) .$$ It determines the shape of the function we sample. In this post, I’ll only look at one-dimensional Gaussian Processes with a zero mean function, i.e. $$ m(\cdot) = 0.$$ A non-zero mean function in the 1-dim case would just correspond to a vertical translation of the function. Below, we'll see 5 different kernels, and in each of the figures I show a graph with just one function and another graph showing twenty functions sampled from the GP. This isn't meant to be a thorough introduction to kernel function theory, but it'll hopefully give you a better understanding of the different types of kernel out there.

<br>

<p align="center">
    <img src="//raw.githubusercontent.com/eweik/eweik.github.io/master/images/gaussian-process-regression/gp_number.png" width="600">
</p>
_Figure 2_: Constant valued functions sampled from a Gaussian Process with a constant kernel $$ k(x_i, x_j) = \sigma^2 $$. Notice how the numbers center around 0 and vary in both directions; this is exactly like sampling a number from Gaussian distribution $$ x \sim \mathcal{N} (0, 1) $$, except here the numbers are constant functions!

<br>

<p align="center">
    <img src="//raw.githubusercontent.com/eweik/eweik.github.io/master/images/gaussian-process-regression/gp_line.png" width="600">
</p>
_Figure 3_: Linear functions are sampled from a Gaussian Process with a linear kernel $$ k(x_i, x_j) = \sigma^2 x_i \cdot x_j $$. Notice, in the right graph, that at each point $$ x $$ the values $$ f(x) $$ center at 0 and vary in proportion to $$ x. $$ That is, $$ f(x) $$ at $$ x = 4 $$ varies much more than at $$ x = 1. $$ The distribution of $$ f(x) $$  at $$ x $$ is distributed according to $$ x \sim \mathcal{N} (0, x). $$

<br>

<p align="center">
    <img src="//raw.githubusercontent.com/eweik/eweik.github.io/master/images/gaussian-process-regression/gp_se.png" width="600">
</p>
_Figure 4_: Smooth functions are sampled from a Gaussian Process with a squared exponential kernel $$ k(x_i, x_j) = \sigma^2 \mathrm{exp}( -\dfrac{1}{2 l^2}| x_i - x_j |^2 ). $$ . These functions are infinitely differentiable at every point. In this figure, the characteristic length scale $$ l $$ is set to 1, but changing $$ l $$ would result in either more stable (with higher $$ l )$$ or more volatile (with smaller $$ l $$) functions.

<br>

<p align="center">
    <img src="//raw.githubusercontent.com/eweik/eweik.github.io/master/images/gaussian-process-regression/gp_symmetric.png" width="600">
</p>
_Figure 5_: Symmetric functions about $$ x = 0 $$ are sampled from a Gaussian Process with the kernel $$ k(x_i, x_j) = \mathrm{exp} ( - \alpha ( \mathrm{min}( |x_i - x_j|, |x_i + x_j| ) )^2). $$

<br>

<p align="center">
    <img src="//raw.githubusercontent.com/eweik/eweik.github.io/master/images/gaussian-process-regression/gp_periodic.png" width="600">
</p>
_Figure 6_: Periodic functions are sampled from a Gaussian Process with the kernel $$ k(x_i, x_j) = \sigma^2 \mathrm{exp} ( - \dfrac{2}{l^2} \mathrm{sin}^2 ( \alpha \pi (x_i - x_j) ) ) .$$

<br>

Each of these kernels has different hyperparameters associated with them that can effect the types of functions sampled. For example, in the periodic kernel, $$ k = \mathrm{exp} ( - \mathrm{sin}^2 ( \alpha \pi (x_i - x_j) ) ) $$, in figure 7, the associated hyperparameter is $$ \alpha .$$ Making $$ \alpha $$ larger would give functions with much higher frequencies of oscillation, while smaller values of $$ \alpha $$ would give functions with lower frequencies.

This isn't by any means an exhaustive introduction to Gaussian Processes. But, hopefully this gives you a good intuitive understanding of what they are and some of the different things you can do with them. 

<br>

# Gaussian Process Regression
In a typical regression problem we are given a dataset $$ \{({\bf x}_{i} , y_{i}) |i=1,...,m\} $$, where $$ y = f({\bf x}) + \epsilon $$ is a noisy observation of the underlying function $$ f({\bf x}) $$. It's assumed that the noise is $$ \epsilon \sim \mathcal{N}(0,\sigma^2)$$  (this is important for the derivation).

<br>

The goal is to predict the $$y$$ values for future $$x$$’s. If we knew what the function $$ f(\cdot) $$ was, then we wouldn’t need to do GP regression because we could just plug in our future $$x$$’s in $$ f(\cdot) $$ and get our prediction. But, we don’t know $$ f(\cdot). $$ So, the idea is to sample functions from a Gaussian Process.

<br>

In theory, we can sample an infinite number of functions and choose only the ones that fit our data. But, in practice, this is obviously intractable. So, if we write down our model 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
$$ y = f({\bf x}) + \epsilon $$ where $$ f( {\bf x}) \sim \mathcal{GP}(0, k(\cdot, \cdot)) $$ and $$ \epsilon \sim \mathcal{N}(0,\sigma^2), $$

then we’ll notice that both $$ f({\bf x}) $$ is multivariate Gaussian distributed (from the definition of Gaussian Processes) and $$ \epsilon $$ is Gaussian distributed (by assumption), and therefore $$y$$ must also be multivariate Gaussian distributed from the additive property of multivariate Gaussians, i.e. $$ y \sim \mathcal{N}(0, k(\dot{},\dot{}) + \sigma^2)! $$

<br>

If we have our training dataset $$ \{({\bf x}_{i} , y_{i}) |i=1,...,m\} $$ and we also have the test dataset $$ \{ {\bf x}_{i} |i=m+1,...,n\}, $$
for which we want to predict $$y_*$$, then a reasonable assumption is that both $$y$$ and $$y_*$$ come from the same distribution, namely

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
$$ \begin{bmatrix} y \\ y_* \end{bmatrix} \sim \mathcal{N}( 0, \begin{bmatrix} k(x,x) + \sigma^2I && k(x,x_*) \\ k(x_*, x) && k(x_*, x_*) + \sigma^2 I \end{bmatrix} ) $$.

where $$ x $$ denotes training data and $$ x_* $$ denotes test data. From the conditioning property of multivariate Gaussians, we can get the conditional distribution of $$ y_* $$ given $$ x $$ as

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
$$ {\bf y}_*|X, {\bf y}, X_* \sim \mathcal{N}( \mu_*, \Sigma_*) $$

where

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
$$ \mu_* = k_*[k + \sigma_n^2 I]^{-1}{\bf y} $$ and $$ \Sigma_* = k_{**} - k_*[k + \sigma_n^2 I]^{-1} k_*. $$

And that’s it. We can now get our estimate $$ \mu_* $$ and our uncertainties $$ \Sigma_* !$$ 

<br>

So, essentially Gaussian Process regression is using the conditioning property of multivariate Gaussians. Of course, we can also incorporate our prior knowledge of the data by specifying the mean function $$ m(\cdot) $$ and the covariance function $$ k(\cdot, \cdot) . $$

<br>

Below is the segment of Python code that’s calculates the estimate $$( \mu_*)$$ and the covariance $$ ( \Sigma_* ). $$ The algorithm I use is taken from Rasmussen et al, chapter 2, where instead of directly taking the inverse of the prediction kernel matrix, they calculate the Cholesky decomposition (i.e. the square root of the matrix).

```python
n = len(X)
K = variance( X, k )  // returns variance of X defined by kernel function k
L = np.linalg.cholesky( K + noise*np.identity(n) )    
    
# predictive mean
alpha = np.linalg.solve( np.transpose(L), np.linalg.solve( L, Y ) )
k_hat = covariance( X, x_hat, k ) // returns covariance of X and x_hat defined by kernel function k
mu_hat = np.matmul( np.transpose(k_hat), alpha )
    
# predictive variance
v = np.linalg.solve( L, k_hat )
k_hathat = variance( x_hat, k )  
a = np.matmul( np.transpose(v), v )
covar_hat = k_hathat - np.matmul( np.transpose(v), v )
var_hat = covar_hat.diagonal()
```

<br>

We did it! Okay, well, at least the math is done. But, it's always nice to see these types of things graphically. Figure 7 below shows Gaussian process regression with a squared exponential kernel in work. Specifically, the evolution of the posterior distribution as more observations are made. the evolution of posterior distribution on predictions as more observations are made.  

<br>

Before any observations, the mean prediction is zero and shaded area is 2 standard deviations from the mean (1.96 in this case). After the first observation is made, the prediction curve changes to accomodate the new observation and the variance shrinks near the region at that point. Subsequent observations produce bigger changes and smaller uncertainties. After ten observations are made, we can already see a pretty nice curve and prediction, with low uncertainties near regions with some points.

<p align="center">
    <img src="//raw.githubusercontent.com/eweik/eweik.github.io/master/images/gaussian-process-regression/evolution.png" width="1000">
</p>
_Figure 7_: These pictures shows how the posterior distribution of the prediction changes as more observations are made. The GP here uses the squared exponential kernel.

<br>

Below are some figures where I play around with Gaussian Process Regression with different types of observations and different kernels. Figure 8 shows linear observations predicted using a GP with a linear kernel. This can also be interpreted as just doing Bayesian linear regression and shows how Gaussian Process Regression is more general than BLR.  

<p align="center">
    <img src="//raw.githubusercontent.com/eweik/eweik.github.io/master/images/gaussian-process-regression/gpr_x_k2.png" width="600">
</p>
_Figure 8_: This plot shows GP regression with a linear kernel o observations corresponding to a noisy $$f(x) = x.$$

<br>

In Figure 9, I model $$ f(x) = x \mathrm{sin} (x) $$ with injected noise. Here I show a couple different kernels I tried for this: the squared exponential kernel and the symmetric kernel. In my opinion, the difference in performance between the two isn’t significant, but, in my opinion the symmetric one does look a bit nice. Although, disclosure, I didn’t optimize any of the hyperparameters in this post.

<p align="center">
    <img src="//raw.githubusercontent.com/eweik/eweik.github.io/master/images/gaussian-process-regression/gpr_xsinx.png" width="600">
</p>
_Figure 9_: Gaussian processes regression using the squared exponential kernel and the symmetric kernel. The observations are based on the function $$ f(x) = x \mathrm{sin} (x). $$

<br>

Figure 10 shows GPR with the squared exponential kernel and the periodic kernel to model noisy observations of $$ \mathrm{sin}(x). $$ I find it neat that beyond the range of observations $$(-5, +5)$$ the periodic kernel is able to follow $$ \mathrm{sin}(x) $$ much better. Which makes sense, because the squared exponential kernel has no reason to continue with the periodic pattern beyond the range where it sees no data. So, in most problems, I don't think this would be a huge deal breaker. It doesn't seem to make sense to have to make predictions on data that isn't closely related to your training set. So, the squared exponential kernel seems to the best all-purpose kernel for the types of data that I looked at.

<p align="center">
    <img src="//raw.githubusercontent.com/eweik/eweik.github.io/master/images/gaussian-process-regression/gpr_sinx.png" width="600">
</p>
_Figure 10_: Gaussian processes regression using the squared exponential kernel and the periodic kernel. The observations are based on the function $$ \mathrm{sin}(x) .$$

<br>

# Conclusion
Gaussian Process regression is powerful all-purpose, general tool for Bayesian regression. Although I didn't show it here, GPR in geostatistics (where it's 2-dimensional) is particularly cool visually. It makes me think of GPR as a tool to _fill in the blanks_ in between observations for continuous setting.

<br>

The time complexity of GPR is $$O(n^3)$$ based on the implementation above (this comes from inverting the covariance matrix). This is fine for small datasets, but we start looking at datasets on the order of millions of events, then this becomes ugly. There are faster methods for doing GPR, but they are beyond the scope of this post (you can find a textbook on this stuff in the reference pages below).

<br>

_Aside - Reflection_: I think it was pretty cool to do this sort of introduction/explanation style post. My first try at something like this! If you see anything in this post that needs correcting, please let me know so I can fix it (contact info in _About_ page). Thanks for reading this also! Hopefully, this stuff makes a little more sense to you :)


### References
* Carl E. Rasmussen and Christopher K. I. Williams. Gaussian Processes for Machine Learning. MIT Press, 2006. Online: [http://www.gaussianprocess.org/gpml/](http://www.gaussianprocess.org/gpml/)


