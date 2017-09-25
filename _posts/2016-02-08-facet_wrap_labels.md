---
title: "Math Expressions with Facets in ggplot2"
author: "sahir"
date: "February 8, 2016"
output: html_document
layout: post
tags: [R, ggplot2, facet_wrap, labels]
comments: yes
---



In this post I show how we can use $$\LaTeX$$ math expressions to label the panels in facets to produce the following plot:

<img src="/figure/posts/2016-02-08-facet_wrap_labels/unnamed-chunk-1-1.png" title="plot of chunk unnamed-chunk-1" alt="plot of chunk unnamed-chunk-1" style="display: block; margin: auto;" />




<!--more-->

The updated version of [ggplot2 V 2.0](http://docs.ggplot2.org/dev/index.html) has improved the way we can label panels in [facet plots](http://docs.ggplot2.org/dev/facet_wrap.html) with the use of a [generic labeller](http://docs.ggplot2.org/dev/labeller.html) function. The [latex2exp](https://cran.r-project.org/web/packages/latex2exp/index.html) package has made it much easier to write $$\LaTeX$$ expressions in `R`.

You will need to load the following packages for the code below to work:

1. [devtools](https://cran.r-project.org/web/packages/devtools/index.html)
2. [ggplot2](https://cran.r-project.org/web/packages/ggplot2/)
3. [latex2exp](https://cran.r-project.org/web/packages/latex2exp/index.html)


I have posted some sample data in a [GitHub Gist](https://gist.github.com/sahirbhatnagar/ed3caf50247cae8e3e1c) which you can import into your `R` session using the `source_gist` function from the devtools package:


{% highlight r %}
data <- devtools::source_gist(id = "ed3caf50247cae8e3e1c", filename = "facet_data.R")$value
{% endhighlight %}

Then we create a labelling function which takes as input a string and prepends $$\log(\lambda_{\gamma})$$ to it. Note that `latex2exp::TeX` is the workhorse function that parses $$\LaTeX$$ syntax so that `R` understands it. Otherwise it becomes very messy to try and write more complex math expressions in `R`.


{% highlight r %}
appender <- function(string) 
    TeX(paste("$\\log(\\lambda_{\\gamma}) = $", string))  
{% endhighlight %}

The code to produce the plot above is given by:


{% highlight r %}
ggplot(data, aes(log(lambda.beta), ymin = lower, ymax = upper)) + 
    geom_errorbar(color = "grey") + 
    geom_point(aes(x = log(lambda.beta), y = mse), 
               colour = "red") +
    theme_bw() + 
    facet_wrap(~lg, scales = "fixed", 
               labeller = as_labeller(appender, 
                            default = label_parsed)) + 
    theme(strip.background = element_blank(), 
          strip.text.x = element_text(size = 14)) + 
    xlab(TeX("$\\log(\\lambda_{\\beta})$"))
{% endhighlight %}

Note that we need to provide the `default = label_parsed` argument to the `facet_wrap` function so that it interprets the result from the `appender` function as math expressions.




