---
title: "Polygenic Risks Scores with data.table in R"
author: "Sahir"
date: "August 11, 2017"
output: html_document
layout: post
tags: [R, data.table, genomics]
permalink: polygenic-risk-scores
comments: yes
---



In this short post, I show how to calculate [polygenic risk scores](https://en.wikipedia.org/wiki/Polygenic_score) (PRS) using the [`data.table`](https://cran.r-project.org/web/packages/data.table/vignettes/datatable-intro.html) package in `R`. I will show an example on a small dataset, but can be easily extended to much larger datasets. The PRS based on $$ p $$ SNPs is given by:  


$$
PRS_i = \sum_{j=1}^{p}\beta_j \times SNP_{ij}
$$

where $$ \beta_j $$ is the beta coefficient for the $$ j^{th} $$ SNP, and $$ SNP_{ij} $$ is the value of $$ j^{th} $$ SNP for the $$ i^{th} $$ individual.  

<!--more-->


First we load the [`data.table`](https://github.com/Rdatatable/data.table/wiki) package:

{% highlight r %}
pacman::p_load(data.table)
{% endhighlight %}

Next we create some sample data, where the rows are SNPs and the columns are individuals. This `data.table` also contains a column with the beta coefficients for each SNP. 


{% highlight r %}
# sample data, rows are SNPs, columns are individuals
DT <- data.table(ID1 = sample(0:2, 10, replace = T),
                 ID2 = sample(0:2, 10, replace = T),
                 ID3 = sample(0:2, 10, replace = T),
                 beta = rnorm(10))
DT
{% endhighlight %}



{% highlight text %}
##     ID1 ID2 ID3        beta
##  1:   2   0   1  2.35674416
##  2:   2   0   1  0.22178574
##  3:   2   0   2 -0.15026475
##  4:   0   1   0 -0.97745574
##  5:   0   0   1 -0.39511441
##  6:   0   0   1  1.10683879
##  7:   2   2   2  0.42516701
##  8:   2   2   2 -0.93244438
##  9:   0   0   1  0.09155816
## 10:   0   2   1 -1.42107331
{% endhighlight %}

In situations where there are many individuals, this can be a tedious calculation because standard methods would require to type out the name of all of the columns to be multiplied by. Instead, using the [`.SDcols`](https://stackoverflow.com/questions/14937165/using-dynamic-column-names-in-data-table?lq=1) argument, greatly simplifies this process by allowing columns to be called on dynamically.  

We first create a character vector of the column names that we want to multiply beta by as well as the new column names that we want to store the results in:


{% highlight r %}
# the names of the columns to be multiplied by beta
id_names <- paste0("ID",1:3)

# the names of the new columns that have been multiplied by beta
new_id_names <- paste0("new_",id_names)
{% endhighlight %}


Now with a simple command we get the desired multiplication:

{% highlight r %}
DT[, (new_id_names) := .SD * beta, .SDcols = id_names]
DT
{% endhighlight %}



{% highlight text %}
##     ID1 ID2 ID3        beta    new_ID1    new_ID2     new_ID3
##  1:   2   0   1  2.35674416  4.7134883  0.0000000  2.35674416
##  2:   2   0   1  0.22178574  0.4435715  0.0000000  0.22178574
##  3:   2   0   2 -0.15026475 -0.3005295  0.0000000 -0.30052949
##  4:   0   1   0 -0.97745574  0.0000000 -0.9774557  0.00000000
##  5:   0   0   1 -0.39511441  0.0000000  0.0000000 -0.39511441
##  6:   0   0   1  1.10683879  0.0000000  0.0000000  1.10683879
##  7:   2   2   2  0.42516701  0.8503340  0.8503340  0.85033403
##  8:   2   2   2 -0.93244438 -1.8648888 -1.8648888 -1.86488876
##  9:   0   0   1  0.09155816  0.0000000  0.0000000  0.09155816
## 10:   0   2   1 -1.42107331  0.0000000 -2.8421466 -1.42107331
{% endhighlight %}

The PRS for each individual can be calculated using the `colSums` function in base `R`:


{% highlight r %}
DT[, colSums(.SD), .SDcols = new_id_names]
{% endhighlight %}



{% highlight text %}
##    new_ID1    new_ID2    new_ID3 
##  3.8419756 -4.8341571  0.6456549
{% endhighlight %}

