---
title: "Bayesian Non-Linear Multilevel Models"
author: "Sahir"
date: "February 23, 2017"
output: html_document
layout: post
tags: [R, regression, Bayesian, Multilevel]
permalink: bayesian-stan
comments: yes
---



Consider the following repeated measures model:

$$ y_{ij} =\beta_0 + \beta_1 a_{ij}  + \beta_1^2 b_{ij} + \mu_i + \varepsilon_{ij}$$

for $$i = 1, \ldots, n$$, $$j = 1, 2$$ where $$n$$ is the sample size, $$j$$ represents the index of the repeated measure, i.e., each subject has two measurements, $$\mu_i$$ is a normally distributed random effect, $$\varepsilon_{ij}$$ is a normally distributed error term, $$y_{ij}$$ is the continuous response, and $$a_{ij}, b_{ij}$$ are covariates. This is a multilevel model because of the nested structure of the data, and also non-linear in the $$\beta_1$$ parameter. In this post I simulate some data under this model, and try to leverage Bayesian computation techniques to estimate the parameters using the [brms](https://github.com/paul-buerkner/brms) which is an interface to fit Bayesian generalized (non-)linear multilevel models using [Stan](http://mc-stan.org/).

![](http://i.imgur.com/r0IXN1w.jpg)

<!--more-->





## Load Packages

We first load the required packages:


{% highlight r %}
if (!requireNamespace("pacman", quietly = TRUE)) 
  install.packages("pacman")
pacman::p_load('brms')
{% endhighlight %}


## Simulate Data (n = 500, two timepoints)

Next I simulate some multilevel data:


{% highlight r %}
rm(list=ls())

# number of subjects
n = 500

# number of time points
t = 2

# SNR
SNR = 2

# true coefficient value
b <- 3

# simulate independent covariates
A <- rgamma(n*t, shape = 3)
B <- rnorm(n*t)

# random effect: same for time 1 and time 2
random_eff <- rpois(n, lambda = 10)

y.star <- b * A + b^2 * B + rep(random_eff,2)
error <- rnorm(n*t)
k <- sqrt(stats::var(y.star)/(SNR*stats::var(error)))

# response
yij <- y.star + k*error 

dat1 <- data.frame(yij, 
                   aij = A, 
                   bij = B, 
                   ID = rep(1:n, times = 2))
{% endhighlight %}


## Fit Non-linear Multilevel Bayesian Model

We first need to specify priors for $$\beta_1$$ and the random effect $$\mu_i$$. I have found the easiest way to do this, is to first get information for which priors may be specified using the `brms::get_prior` function:


{% highlight r %}
(prior <- brms::get_prior(
  brms::bf(yij ~ beta * aij + (beta^2) * bij + mui, 
           mui ~ 1 + (1|ID),
           beta ~ 1, 
           nl = TRUE),
  data = dat1, 
  family = gaussian()))
{% endhighlight %}



{% highlight text %}
##                 prior class      coef group nlpar bound
## 1 student_t(3, 0, 13) sigma                            
## 2                         b                  beta      
## 3                         b Intercept        beta      
## 4                         b                   mui      
## 5                         b Intercept         mui      
## 6 student_t(3, 0, 13)    sd                   mui      
## 7                        sd              ID   mui      
## 8                        sd Intercept    ID   mui
{% endhighlight %}

The `nl = TRUE` argument indicates that this should be treated as a non-linear model. We need to specify priors for the population level effects as well as the standard deviation of the random effect (what is referred to as group-level effects in the output):


{% highlight r %}
prior$prior[c(2,4)] <- "normal(0,10)"
prior$prior[8] <- "student_t(3, 0, 11)"
prior
{% endhighlight %}



{% highlight text %}
##                 prior class      coef group nlpar bound
## 1 student_t(3, 0, 13) sigma                            
## 2        normal(0,10)     b                  beta      
## 3                         b Intercept        beta      
## 4        normal(0,10)     b                   mui      
## 5                         b Intercept         mui      
## 6 student_t(3, 0, 13)    sd                   mui      
## 7                        sd              ID   mui      
## 8 student_t(3, 0, 11)    sd Intercept    ID   mui
{% endhighlight %}

We can check the corresponding STAN code to verify that the prior information has been updated accordingly using the `brms::make_stancode` function:


{% highlight r %}
brms::make_stancode(
  bf(yij ~ beta * aij + (beta^2) * bij + mui, 
     mui ~ 1 +  (1|ID),
     beta ~ 1, 
     nl = TRUE),
  data = dat1, 
  family = gaussian(),
  prior = prior)
{% endhighlight %}



{% highlight text %}
## // generated with brms 1.5.0
## functions { 
## } 
## data { 
##   int<lower=1> N;  // total number of observations 
##   vector[N] Y;  // response variable 
##   int<lower=1> K_mui;  // number of population-level effects 
##   matrix[N, K_mui] X_mui;  // population-level design matrix 
##   int<lower=1> K_beta;  // number of population-level effects 
##   matrix[N, K_beta] X_beta;  // population-level design matrix 
##   int<lower=1> KC;  // number of covariates 
##  matrix[N, KC] C;  // covariate matrix 
##   // data for group-level effects of ID 1 
##   int<lower=1> J_1[N]; 
##   int<lower=1> N_1; 
##   int<lower=1> M_1; 
##   vector[N] Z_1_mui_1; 
##   int prior_only;  // should the likelihood be ignored? 
## } 
## transformed data { 
## } 
## parameters { 
##   vector[K_mui] b_mui;  // population-level effects 
##   vector[K_beta] b_beta;  // population-level effects 
##   real<lower=0> sigma;  // residual SD 
##   vector<lower=0>[M_1] sd_1;  // group-level standard deviations 
##   vector[N_1] z_1[M_1];  // unscaled group-level effects 
## } 
## transformed parameters { 
##   // group-level effects 
##   vector[N_1] r_1_mui_1; 
##   r_1_mui_1 = sd_1[1] * (z_1[1]); 
## } 
## model { 
##   vector[N] mu_mui; 
##   vector[N] mu_beta; 
##   vector[N] mu; 
##   mu_mui = X_mui * b_mui; 
##   mu_beta = X_beta * b_beta; 
##   for (n in 1:N) { 
##     mu_mui[n] = mu_mui[n] + (r_1_mui_1[J_1[n]]) * Z_1_mui_1[n]; 
##     // compute non-linear predictor 
##     mu[n] = mu_beta[n] * C[n, 1] + (mu_beta[n] ^ 2) * C[n, 2] + mu_mui[n]; 
##   } 
##   // prior specifications 
##   b_mui ~ normal(0,10); 
##   b_beta ~ normal(0,10); 
##   sigma ~ student_t(3, 0, 13); 
##   sd_1 ~ student_t(3, 0, 11); 
##   z_1[1] ~ normal(0, 1); 
##   // likelihood contribution 
##   if (!prior_only) { 
##     Y ~ normal(mu, sigma); 
##   } 
## } 
## generated quantities { 
## }
{% endhighlight %}

Next we fit the model with 6 Markov chains:


{% highlight r %}
# refresh = 0 to surpress output
fit1 <- brms::brm(
  bf(yij ~ beta * aij + (beta^2) * bij + mui, 
     mui ~ 1 + (1|ID),
     beta ~ 1, 
     nl = TRUE),
  data = dat1, 
  family = gaussian(),
  prior = prior,
  control = list(adapt_delta = 0.9),
  iter = 10000, 
  thin = 2, 
  chains = 6, 
  cores = 6, 
  refresh = 0)
{% endhighlight %}



{% highlight text %}
## Compiling the C++ model
{% endhighlight %}



{% highlight text %}
## Start sampling
{% endhighlight %}



{% highlight text %}
## Warning: There were 14 divergent transitions after warmup. Increasing adapt_delta above 0.9 may help. See
## http://mc-stan.org/misc/warnings.html#divergent-transitions-after-warmup
{% endhighlight %}



{% highlight text %}
## Warning: Examine the pairs() plot to diagnose sampling problems
{% endhighlight %}

We can check the summary to see if the model was able to accurately estimate $$\beta_1$$: 


{% highlight r %}
(res <- summary(fit1))
{% endhighlight %}



{% highlight text %}
##  Family: gaussian (identity) 
## Formula: yij ~ beta * aij + (beta^2) * bij + mui 
##          mui ~ 1 + (1 | ID)
##          beta ~ 1
##    Data: dat1 (Number of observations: 1000) 
## Samples: 6 chains, each with iter = 10000; warmup = 5000; thin = 2; 
##          total post-warmup samples = 15000
##    WAIC: Not computed
##  
## Group-Level Effects: 
## ~ID (Number of levels: 500) 
##                   Estimate Est.Error l-95% CI u-95% CI Eff.Sample
## sd(mui_Intercept)     2.13      0.79     0.28     3.42       2384
##                   Rhat
## sd(mui_Intercept)    1
## 
## Population-Level Effects: 
##                Estimate Est.Error l-95% CI u-95% CI Eff.Sample Rhat
## mui_Intercept     10.28      0.30     9.69    10.87      13058    1
## beta_Intercept     2.97      0.04     2.89     3.06      12924    1
## 
## Family Specific Parameters: 
##       Estimate Est.Error l-95% CI u-95% CI Eff.Sample Rhat
## sigma     8.05      0.25     7.56     8.55       4058    1
## 
## Samples were drawn using sampling(NUTS). For each parameter, Eff.Sample 
## is a crude measure of effective sample size, and Rhat is the potential 
## scale reduction factor on split chains (at convergence, Rhat = 1).
{% endhighlight %}

The estimate of interest in the output above is `beta_Intercept` under `Population-Level Effects`. This corresponds to the estimate of $$\beta_1$$ in the equation above. We see that the estimate is close to the true value of 3. We also see that the estimate of the standard deviation of the random effect is 2.13 [95% CI: 0.28, 3.42], indicating a strong subject specific effect (which is what we would expect since we generated the data this way). 

We can also plot some model diagnostics to ensure proper mixing of the Markov chains:


{% highlight r %}
plot(fit1)
{% endhighlight %}

![plot of chunk unnamed-chunk-9](/figure/posts/2017-02-23-non_linear_bayesian/unnamed-chunk-9-1.png)

{% highlight r %}
plot(marginal_effects(fit1), points = TRUE, ask = FALSE)
{% endhighlight %}

![plot of chunk unnamed-chunk-9](/figure/posts/2017-02-23-non_linear_bayesian/unnamed-chunk-9-2.png)![plot of chunk unnamed-chunk-9](/figure/posts/2017-02-23-non_linear_bayesian/unnamed-chunk-9-3.png)![plot of chunk unnamed-chunk-9](/figure/posts/2017-02-23-non_linear_bayesian/unnamed-chunk-9-4.png)

## Session Info


{% highlight text %}
## R version 3.4.0 (2017-04-21)
## Platform: x86_64-pc-linux-gnu (64-bit)
## Running under: Ubuntu 16.04.2 LTS
## 
## Matrix products: default
## BLAS: /usr/lib/openblas-base/libblas.so.3
## LAPACK: /usr/lib/libopenblasp-r0.2.18.so
## 
## attached base packages:
## [1] methods   stats     graphics  grDevices utils     datasets 
## [7] base     
## 
## other attached packages:
## [1] brms_1.5.0    ggplot2_2.2.1 Rcpp_0.12.11 
## 
## loaded via a namespace (and not attached):
##  [1] highr_0.6            compiler_3.4.0       plyr_1.8.4          
##  [4] tools_3.4.0          digest_0.6.12        bayesplot_1.1.0     
##  [7] evd_2.3-2            statmod_1.4.27       evaluate_0.10       
## [10] tibble_1.3.0         gtable_0.2.0         nlme_3.1-131        
## [13] lattice_0.20-35      DBI_0.5-1            parallel_3.4.0      
## [16] loo_1.0.0            gridExtra_2.2.1      coda_0.19-1         
## [19] stringr_1.2.0        dplyr_0.5.0          knitr_1.16          
## [22] stats4_3.4.0         grid_3.4.0           inline_0.3.14       
## [25] R6_2.2.0             rstan_2.14.1         pacman_0.4.1        
## [28] reshape2_1.4.2       magrittr_1.5         scales_0.4.1        
## [31] codetools_0.2-15     StanHeaders_2.14.0-1 matrixStats_0.51.0  
## [34] rstantools_1.1.0     abind_1.4-5          assertthat_0.1      
## [37] colorspace_1.3-2     labeling_0.3         stringi_1.1.5       
## [40] lazyeval_0.2.0       munsell_0.4.3
{% endhighlight %}










