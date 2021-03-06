<!-- knowledge_base.md is generated from knowledge_base.Rmd. Please edit that file -->

# How to set priors:

## Table of contents

[Gaussian models](#gaussian-models)  
[Binomial models](#binomial-models-logit-link)  
[Poisson models](#poisson-models-log-link)

### Gaussian models

### Binomial models (logit link)

As an example we choose the prosocial chimpanzees experiment from
McElreath (Chapter Binomial regression p. 333). They sit in front of a
table with two levers (left, right) and across the table sits either
another chimpanzee or no one. One of the levers gives both chimanzees
food, the other one only the chimpanzee that makes the decision/pulls
the lever. The dependent variable is `pulled_left` (coding: left = 1,
right = 0) with `prosocial_left` (coding: prosocial on the left = 1) and
`condition` (featuring another chimpanzee on the other side = 1) as
independent variables.

#### Intercept a

We want a prior for the intercept of `pulled_left` so what is the
general probability of a chimpanzee pulling either lever. If we use a
Normal(0, 10) prior the probability is only on 0 and 1, so the model
thinks that chimpanzees either never or always pull the the left lever.
Better is something that distributes the probability more in the middle
like Normal(0, 1). We have to take into account that a normal
distribution becomes logit-normal because of the logit link function in
binomial regression. Let’s plot prior predictive checks:

``` r
# I'm using the logitnorm package here because there is no standard implementation of a logit-normal distribution in R
# x is the probability of pulling left/right (left = 1)
# y is the density
ggplot(data.frame(x = c(0:1)), aes(x)) + 
  stat_function(fun = logitnorm::dlogitnorm, args = list(mu = 0, sigma = 1), colour = 'red') +
  stat_function(fun = logitnorm::dlogitnorm, args = list(mu = 0, sigma = 10), colour = 'black') +
  scale_x_continuous(limits = c(0,1)) +
  labs(x = "Prior Probability Pulling Left/Right", y = "Density")
```

<img src="figures/unnamed-chunk-2-1.png" width="60%" style="display: block; margin: auto;" />

#### Predictor coefficient b

Let’s start in case we have some dichotomous predictor variable like
`prosocial_left`, but this strategy works also for continuous predictors
(you have to check however whether you want to center or standardise
them; for prior selection that can actually be easier). Basically (as a
rule-of thumb), we can think of a plot with the range of the outcome
(`pulled_left`) on the y-axis and the range of a predictor
(`prosoc_left`) on the x-axis. The range of the outcome is just 0-1
(pulled right vs. pulled left) and the range of our predictor is also
0-1 (prosocial option right vs left). The biggest positive effect
imaginable is going from pulled right (0) to pulled left (1) because the
prosocial options is moved from right (0) to left (1). If we draw a
line, it has a slope of 1. The biggest negative effect would be -1.

``` r
ggplot(data.frame(x = c(0:1))) +
  aes(x) +
  stat_function(fun = function(x) x, colour = "darkolivegreen4") +
  stat_function(fun = function(x) 1 - x, colour = "orangered2")
```

<img src="figures/unnamed-chunk-3-1.png" width="60%" style="display: block; margin: auto;" />

This makes us think that slopes between -1 and 1 are what to expect in
most cases. We hence say that our slopes are normally distributed with
mean = 0 (we don’t nudge the effect to be either positive or negative)
and standard deviation of 0.5 (so 2 SDs equal 1 (or -1) and represent
the \~95% borders). To see how these slopes look like (the relationships
that are possible between x and y) we can plot the linear model part of
our model (while also inversing the logit link so we actually plot
probability rather than log odds). This is important because with our
rule-of-thumb if think in linear terms. In Binomial models however there
is the logit-link which makes our priors behave different than we
expect. So always plot the slope in combination with the intercept prior
and the inverse of the link function.

``` r
N <- 100 # how many prior slopes do you want
a <- rnorm(N, 0, 1) # intercept prior that we decided on earlier
b <- rnorm(N, 0, 1) # beta prior

# we could use a for loop and curve() like McELreath...but we choose ggplot ;)

# the first two lines just create an empty plot
# the map function from purrr applies my function (linear model function with inverse logit to have probability instead of log odds) to each element (.x) of the vector 1:N (N is the number of slopes I want to plot)
# the size argument makes the lines thinner (default is something like 0.5)
ggplot(data.frame(x = 0:1)) +
  aes(x) +
  ylim(0,1) +
  purrr::map(
    1:N,
    ~ stat_function(
      fun = function(x) inv_logit(a[.x] + b[.x]*x),
      size = 0.3)
  )
```

<img src="figures/unnamed-chunk-4-1.png" width="60%" style="display: block; margin: auto;" />

Some slopes are positive, some are negative and all look realistic. If
you go for some prior like N(0, 10) for beta, you will end up with
mostly very extreme relationships. If the predictor would be continuous
(e.g. heigth of the chimpanzee) we would first decide on centering or
standardisation (center is good as a default; if I have big differences
in scale between predictors it would also be good to standardise them;
for instance if one predictor goes from 0 to 1 and the other from -100
to 100). After that we can think about the range of x and y as min/max
on our axises again and come up with a prior. Let’s say we have a
standardised continuous predictor with 95% of values between -2 and 2,
then our SD is 1 (mean 0 as usual if we don’t want to say something
about the valence of the effect).

*Note: There may be other ways to decide on beta priors, such as the
difference between different treatments for instance. For this check
Rethinking chapter about Binomial regression page 336-337.*

### Poisson models (log link)

Let’s say you have a count of how many fish was fished in a lake per
day. So there is no upper boundary (hence we use Poisson)

#### Intercept a

Before seeing the data we expect that people fish mostly between 10 and
40 fish per day but also numbers like 100 can happen. Since Poisson uses
a log link function a normal prior becomes log-normal. To know what a
normal mean and standard deviation translate to on the log-normal scale
we can use the following formula to calculate the log-normal mean:

``` r
exp(mean + sd^2/2) = 5.184706e+21
```

A normal prior like Normal(0, 10) becomes `exp(50)`as mean which is
complete nonsense (super huge number). For our example something like
Normal(3, 1) would be better. Now the mean is around 33 fish. To plot
different mean and SD combinations (on log-normal scale) use:

``` r
# y just the prob density
ggplot2::ggplot(data.frame(x = c(0:200)), aes = (x = x)) + 
  stat_function(fun = dlnorm, args = list(meanlog = 3, sdlog = 1)) +
  scale_x_continuous(limits = c(0,200)) +
  labs(x = "Number of fish", y = "Probability density")
```

<img src="figures/unnamed-chunk-6-1.png" width="60%" style="display: block; margin: auto;" />

#### Predictor coefficient b

This is similar to finding priors in the binomial case, the only
difference when plotting the linear model is that we have to reverse a
log-link instead of a logit-link. We do that with exponentiation
(`exp()` in R). In our fishing example there might be binary predictors
like whether you use a `livebait` or whether you came in a `camper`.
There might also be a variable how many adult `persons` where in the
group (there can be kids too). For the binary predictors we just think
of our rule-of-thumb and end up with a y range of 0-200 and x range of
0-1. On a linear scale we would now say we choose a slope of 200 which
results in a SD of 100 (remember that 200 would be 2 SDs). The problem
is that we have to take the log-link into account. If you want try to
plot slopes with a prior for b with SD = 100 with and without the exp().
Without the inverse we actually get “meaningful” slopes (however the
intercept prior is still matched to the link function and thus binds
everything on the left side together). With the inverse link (exp()) the
slopes are crazy. So we have to try again which ones work with the
inverse link. A SD of 1 seems okay once again.

``` r
N <- 100 # how many prior slopes do you want
a <- rnorm(N, 3, 1) # intercept prior that we decided on earlier
b <- rnorm(N, 0, 1)

ggplot(data.frame(x = 0:1)) +
  aes(x) +
  ylim(0, 200) +
  purrr::map(
    1:N,
    ~ stat_function(
      fun = function(x) exp(a[.x] + b[.x]*x),
      size = 0.3)
  )
```

<img src="figures/unnamed-chunk-7-1.png" width="60%" style="display: block; margin: auto;" />
