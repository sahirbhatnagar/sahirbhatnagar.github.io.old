---
layout: post
title: I Am Color Blind
date: 2018-04-03 15:09:00
---


I recently gave a [mini-course](/assets/pdf/cart_animation.pdf) on regression trees. As I talked about the red region in the following figure:

![](/assets/img/cart.png)

one of the audience members said they had no clue what I was referring to. It turns out they were color blind and could not identify the red region. Yikes!  

Here is a colorblind friendly pallette courtesy of [Cookbook for R](http://www.cookbook-r.com/Graphs/Colors_(ggplot2)/#a-colorblind-friendly-palette)


{% highlight r %}
cbbPalette <- c("#000000", "#E69F00", "#56B4E9", "#009E73", "#F0E442", "#0072B2", "#D55E00", "#CC79A7")
{% endhighlight %}