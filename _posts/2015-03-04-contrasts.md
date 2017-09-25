---
layout: post
title: Contrasts in R
date: 2015-03-04 15:09:00
---

In this post I discuss how to create custom contrasts for factor variables in `R`. First lets create some simulated data. Create the data, and factor Disease status:


{% highlight r %}
Disease <- c(rep("RA", 5), rep("SLE", 5), rep("Scleroderma", 5), 
             rep("Myositis", 5), rep("Control", 5))
set.seed(1234)
sex <-  rbinom(25,1, 0.5)
age <-  rnorm(25, 40, 5)
y <- rnorm(25, 0.5, 0.12)
data <- data.frame(y,sex,age,Disease=factor(Disease))
str(data)
{% endhighlight %}



{% highlight text %}
## 'data.frame':	25 obs. of  4 variables:
##  $ y      : num  0.506 0.323 0.552 0.492 0.513 ...
##  $ sex    : int  0 1 1 1 1 1 0 0 1 1 ...
##  $ age    : num  44.4 46.9 31.6 36.9 40.1 ...
##  $ Disease: Factor w/ 5 levels "Control","Myositis",..: 3 3 3 3 3 5 5 5 5 5 ...
{% endhighlight %}

We want the following contrasts:

* Control versus all 4 diseases combined
* RA versus the combination of (SLE, Scleroderma, Myositis), leaving out the Controls

<!--more-->

## Default settings

Let $$x_1,x_2,x_3,x_4$$ be the indicators for Myositis, RA, Scleroderma and SLE, respectively. The standard linear model `R` will fit is given by (for simplicity I am ignoring age and sex, but it won't make a difference when you add them in the model):

$$ \mu_y = \beta_0 + \beta_1 x_1 + \beta_2 x_2 + \beta_3 x_3 + \beta_4 x_4 $$


{% highlight r %}
summary(fit <- lm(y ~ Disease, data=data))
{% endhighlight %}



{% highlight text %}
## 
## Call:
## lm(formula = y ~ Disease, data = data)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -0.18718 -0.05324  0.02859  0.07511  0.12395 
## 
## Coefficients:
##                    Estimate Std. Error t value Pr(>|t|)    
## (Intercept)         0.60943    0.04756  12.814 4.22e-11 ***
## DiseaseMyositis    -0.24835    0.06726  -3.692  0.00144 ** 
## DiseaseRA          -0.13225    0.06726  -1.966  0.06330 .  
## DiseaseScleroderma -0.15540    0.06726  -2.310  0.03165 *  
## DiseaseSLE         -0.05650    0.06726  -0.840  0.41081    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.1063 on 20 degrees of freedom
## Multiple R-squared:  0.4452,	Adjusted R-squared:  0.3342 
## F-statistic: 4.012 on 4 and 20 DF,  p-value: 0.01507
{% endhighlight %}

This is the default contrast matrix with unordered factor variables:

{% highlight r %}
contrasts(data$Disease)
{% endhighlight %}


|            | Myositis| RA| Scleroderma| SLE|
|:-----------|--------:|--:|-----------:|---:|
|Control     |        0|  0|           0|   0|
|Myositis    |        1|  0|           0|   0|
|RA          |        0|  1|           0|   0|
|Scleroderma |        0|  0|           1|   0|
|SLE         |        0|  0|           0|   1|

This compares the mean of the response for the Controls to the mean of the response for Myositis, RA, Scleroderma, and SLE separately. The table can be read by column, and the numbers in the columns represent the weight of the regression coefficient, e.g. in the first column Myositis is being compare to Control. 

## Custom Contrats

Since we want only two contrasts, we want `R` to fit the following model:

$$ \mu_y = \beta_0 + \beta_1 x_1 + \beta_2 x_2 $$

where $$\beta_1$$ represents the contrast estimate for the comparison between controls and all other diseases, and $$\beta_2$$ represents the contrast estimate of RA versus the combination of SLE, Scleroderma, Myositis.

To create custom contrasts, we must specify the contrast matrix as follows:



|            | Control_vs_All| RA_vs_Myos_Scle_SLE|
|:-----------|--------------:|-------------------:|
|Control     |            0.8|           0.0000000|
|Myositis    |           -0.2|          -0.3333333|
|RA          |           -0.2|           1.0000000|
|Scleroderma |           -0.2|          -0.3333333|
|SLE         |           -0.2|          -0.3333333|

Again we look at the above table, column by column. The variables we want to contrast should have opposite signs and the columns should sum to 0. This contrast matrix leads to the following mean response equations for each of the groups:

$$
\begin{align}
\mu_{control} & = \beta_0 + 0.8 \beta_1\\
\mu_{myos} & = \beta_0 - 0.2 \beta_1 - \frac{1}{3} \beta_2 \\
\mu_{ra} & = \beta_0 - 0.2 \beta_1 + \beta_2 \\
\mu_{scler} & = \beta_0 - 0.2 \beta_1 - \frac{1}{3} \beta_2 \\
\mu_{sle} & = \beta_0 - 0.2 \beta_1 - \frac{1}{3} \beta_2 \\
\end{align}
$$

To solve for $$\beta_0$$ we can add up all the equations to get

$$ 
\begin{align}
\mu_{control}+\mu_{myos}+\mu_{ra}+\mu_{scler}+\mu_{sle} & = 5 \beta_0 \\
\beta_0 & = \frac{\mu_{control}+\mu_{myos}+mu_{ra}+\mu_{scler}+\mu_{sle}}{5}
\end{align}
$$

To solve for $$\beta_1$$ we substract $$\mu_{control}$$ from the combined mean of $$\mu_{myos},\mu_{ra},\mu_{scler}$$ and $$\mu_{sle}$$ which gives:

$$
\begin{align}
\mu_{control}-\frac{\mu_{myos}+\mu_{ra}+\mu_{scler}+\mu_{sle}}{4} & = \beta_0 + 0.8 \beta_1 - \frac{4\beta_0 -0.8\beta_1}{4}\\ 
  & = \beta_0 + 0.8 \beta_1 - \beta_0 + 0.2 \beta_1 \\
 & = \beta_1  
\end{align}
$$

To solve for $$\beta_2$$ we substract $$\mu_{ra}$$ from the combined mean of $$\mu_{myos},\mu_{scler}$$ and $$\mu_{sle}$$ which gives:

$$
\begin{align}
\mu_{ra}-\frac{\mu_{myos}+\mu_{scler}+\mu_{sle}}{3} & = \beta_0 - 0.2 \beta_1 + \beta_2 - \frac{3\beta_0 -0.6\beta_1 - \beta_2}{3}\\ 
  & = \beta_0 - 0.2 \beta_1 + \beta_2 - \beta_0 + 0.2 \beta_1 +\frac{1}{3}\beta_2 \\
 & = \frac{4}{3} \beta_2 \\
 \beta_2 & = \frac{3}{4} \left( \mu_{ra}-\frac{\mu_{myos}+\mu_{scler}+\mu_{sle}}{3}  \right)
\end{align}
$$

First we create the contrast matrix with appropriate row and column names for clarity:

{% highlight r %}
my.contr <- matrix(c( 4/5, -1/5, -1/5, -1/5, -1/5,
                 0,-1/3,1,-1/3,-1/3),
              ncol = 2, dimnames = list(c("Control", "Myositis", "RA","Scleroderma","SLE"),
    c("Control_vs_All","RA_vs_Myos_Scle_SLE")))
{% endhighlight %}

Then we store the contrasts attribute to the *Disease* variable. The `how.many` argument specifies how many contrasts we want, therefore this should correspond to the number of columns in the contrast matrix.


{% highlight r %}
contrasts(data$Disease,how.many=2) <- my.contr
contrasts(data$Disease)
{% endhighlight %}



{% highlight text %}
##             Control_vs_All RA_vs_Myos_Scle_SLE
## Control                0.8           0.0000000
## Myositis              -0.2          -0.3333333
## RA                    -0.2           1.0000000
## Scleroderma           -0.2          -0.3333333
## SLE                   -0.2          -0.3333333
{% endhighlight %}



{% highlight r %}
summary(fit <- lm(y ~ Disease, data=data))
{% endhighlight %}



{% highlight text %}
## 
## Call:
## lm(formula = y ~ Disease, data = data)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -0.18778 -0.09120  0.01436  0.06365  0.19806 
## 
## Coefficients:
##                            Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                 0.49093    0.02405  20.410 8.72e-16 ***
## DiseaseControl_vs_All       0.14812    0.06013   2.463   0.0221 *  
## DiseaseRA_vs_Myos_Scle_SLE  0.01587    0.04658   0.341   0.7365    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.1203 on 22 degrees of freedom
## Multiple R-squared:  0.2194,	Adjusted R-squared:  0.1484 
## F-statistic: 3.092 on 2 and 22 DF,  p-value: 0.06557
{% endhighlight %}


Here we check to make sure that the `lm` fit is giving the same result as the formulas derived above:

{% highlight r %}
#' group level means
mu.control <- mean(data[which(data$Disease=="Control"),"y"])
mu.myos <- mean(data[which(data$Disease=="Myositis"),"y"])
mu.ra <- mean(data[which(data$Disease=="RA"),"y"])
mu.scler <- mean(data[which(data$Disease=="Scleroderma"),"y"])
mu.sle <- mean(data[which(data$Disease=="SLE"),"y"])

#' beta0
mean(c(mu.control,mu.myos,mu.ra,mu.scler,mu.sle))
{% endhighlight %}



{% highlight text %}
## [1] 0.4909311
{% endhighlight %}



{% highlight r %}
#' beta1
mu.control - mean(c(mu.myos,mu.ra,mu.scler,mu.sle))
{% endhighlight %}



{% highlight text %}
## [1] 0.1481237
{% endhighlight %}



{% highlight r %}
#' beta2
(mu.ra - mean(c(mu.myos,mu.scler,mu.sle)))*(3/4)
{% endhighlight %}



{% highlight text %}
## [1] 0.01587466
{% endhighlight %}


## References

* [http://www.unc.edu/courses/2006spring/ecol/145/001/docs/lectures/lecture26.htm](http://www.unc.edu/courses/2006spring/ecol/145/001/docs/lectures/lecture26.htm)
* [http://www.ats.ucla.edu/stat/r/library/contrast_coding.htm#User](http://www.ats.ucla.edu/stat/r/library/contrast_coding.htm#User)


