---
title: "Limma Moderated and Ordinary t-statistics"
author: "sahir"
date: "February 7, 2017"
output: html_document
layout: post
tags: [R, statistics, t test, genomics, limma]
permalink: limma_ttest
comments: yes
---

When analyzing large amounts of genetic and genomic data, the first line of analysis is usually some sort of univariate test. That is, conduct a statistical test for each SNP or CpG site or Gene and then correct for multiple testing. The [limma](https://bioconductor.org/packages/release/bioc/html/limma.html) package on Bioconductor is a popular method for computing _moderated_ t-statistics using a combination of the `limma::lmFit` and `limma::eBayes` functions. In this post, I show how to calculate the _ordinary_ t-statistics from `limma` output.


<!--more-->




First we load the required packages

{% highlight r %}
# clear workspace
rm(list=ls())

if (!requireNamespace("pacman", quietly = TRUE)) 
  install.packages("pacman")

# this will install (if you dont already have it) and load the package
pacman::p_load("minfi")
pacman::p_load("minfiData")
pacman::p_load("limma")
pacman::p_load("CpGassoc")

# check the data that is available in the minfiData package
pacman::p_data("minfiData")
{% endhighlight %}



{% highlight text %}
##   Data        Description                                                                           
## 1 MsetEx      An example dataset for Illumina's Human Methylation 450k dataset, after preprocessing.
## 2 MsetEx.sub  An example dataset for Illumina's Human Methylation 450k dataset, after preprocessing.
## 3 RGsetEx     An example dataset for Illumina's Human Methylation 450k dataset.                     
## 4 RGsetEx.sub An example dataset for Illumina's Human Methylation 450k dataset.
{% endhighlight %}

Next, we extract some sample data and create a covariate of interest

{% highlight r %}
# get the M-values for the sample data
DT <- minfi::getM(MsetEx.sub)

dim(DT)
{% endhighlight %}



{% highlight text %}
## [1] 600   6
{% endhighlight %}



{% highlight r %}
# rows are CpGs and columns are samples
head(DT)
{% endhighlight %}



{% highlight text %}
##            5723646052_R02C02 5723646052_R04C01 5723646052_R05C02
## cg00050873          3.502348         0.4414491          4.340695
## cg00212031         -3.273751         0.9234662         -2.614777
## cg00213748          2.076816        -0.1309465          1.260995
## cg00214611         -3.438838         1.7463950         -2.270551
## cg00455876          1.839010        -0.9927320          1.619479
## cg01707559         -3.303987        -0.6433201         -3.540887
##            5723646053_R04C02 5723646053_R05C02 5723646053_R06C02
## cg00050873        0.24458355        -0.3219281         0.2744392
## cg00212031       -0.21052257        -0.6861413        -0.1397595
## cg00213748       -1.10373279        -1.6616553        -0.1270869
## cg00214611        0.29029649        -0.2103599        -0.6138630
## cg00455876       -0.09504721        -0.2854655         0.6361273
## cg01707559       -0.74835377        -0.4678048        -1.1345421
{% endhighlight %}



{% highlight r %}
# create a fake covariate. 3 intact and 3 degraded samples
tissue <- factor(c(rep("Intact",3), rep("Degraded",3)))
design <- model.matrix(~ tissue)
{% endhighlight %}

Then we calculate the moderated and ordinary t-statistics and compare them:

{% highlight r %}
# limma fit
fit <- lmFit(DT, design)
fit_ebayes <- eBayes(fit)

# ordinary t-statistic
ordinary.t <- fit$coefficients[,"tissueIntact"] / 
  fit$stdev.unscaled[,"tissueIntact"] / 
  fit$sigma

plot(x = ordinary.t, 
     y = fit_ebayes$t[,"tissueIntact"], 
     xlab = "ordinary t-statistic", 
     ylab = "moderated t-statistic")
abline(a = 0, b = 1, col = "red")
{% endhighlight %}

![plot of chunk unnamed-chunk-2](/figure/posts/2017-02-07-ttest_limma/unnamed-chunk-2-1.png)


We can calculate the corresponding p-values from the ordinary t-statistics. This is given by 

{% highlight r %}
# t-distribution with n-p-1 degrees of freedom 
# (n: number of samples, p:number of covariates excluding intercept)
ordinary.pvalue <- pt(q = abs(ordinary.t), 
                      df = dim(DT)[2]-(ncol(design)-1)-1, 
                      lower.tail = F) * 2
{% endhighlight %}

We can also use the [`CpGassoc`](https://cran.r-project.org/package=CpGassoc) package to calculate ordinary t-statistics and compare the result to our manual calculations:


{% highlight r %}
# compare result with CpGassoc package (regular t-test)
t_test_mvalues <- CpGassoc::cpg.assoc(
  beta.val = minfi::getBeta(MsetEx.sub),
  indep = tissue,
  fdr.cutoff = 0.05,
  logit.transform = TRUE)

# these should be equal
plot(t_test_mvalues$results$P.value, ordinary.pvalue)
abline(a = 0, b = 1, col = "red")
{% endhighlight %}

![plot of chunk unnamed-chunk-4](/figure/posts/2017-02-07-ttest_limma/unnamed-chunk-4-1.png)

{% highlight r %}
# t-statistic is the squared F.statistic (in certain cases) 
plot(t_test_mvalues$results$F.statistic, ordinary.t^2)
abline(a = 0, b = 1, col = "red")
{% endhighlight %}

![plot of chunk unnamed-chunk-4](/figure/posts/2017-02-07-ttest_limma/unnamed-chunk-4-2.png)


## Session Info


{% highlight text %}
## R version 3.3.2 (2016-10-31)
## Platform: x86_64-pc-linux-gnu (64-bit)
## Running under: Ubuntu 16.04.1 LTS
## 
## attached base packages:
## [1] stats4    parallel  methods   stats     graphics  grDevices
## [7] utils     datasets  base     
## 
## other attached packages:
##  [1] CpGassoc_2.55                                     
##  [2] nlme_3.1-128                                      
##  [3] limma_3.30.10                                     
##  [4] minfiData_0.20.0                                  
##  [5] IlluminaHumanMethylation450kanno.ilmn12.hg19_0.6.0
##  [6] IlluminaHumanMethylation450kmanifest_0.4.0        
##  [7] minfi_1.20.2                                      
##  [8] bumphunter_1.14.0                                 
##  [9] locfit_1.5-9.1                                    
## [10] iterators_1.0.8                                   
## [11] foreach_1.4.3                                     
## [12] Biostrings_2.42.1                                 
## [13] XVector_0.14.0                                    
## [14] SummarizedExperiment_1.4.0                        
## [15] GenomicRanges_1.26.2                              
## [16] GenomeInfoDb_1.10.2                               
## [17] IRanges_2.8.1                                     
## [18] S4Vectors_0.12.1                                  
## [19] Biobase_2.34.0                                    
## [20] BiocGenerics_0.20.0                               
## [21] knitr_1.15.1                                      
## 
## loaded via a namespace (and not attached):
##  [1] mclust_5.2.2             base64_2.0              
##  [3] Rcpp_0.12.9              lattice_0.20-34         
##  [5] Rsamtools_1.26.1         digest_0.6.12           
##  [7] R6_2.2.0                 plyr_1.8.4              
##  [9] RSQLite_1.1-2            evaluate_0.10           
## [11] highr_0.6                httr_1.2.1              
## [13] zlibbioc_1.20.0          GenomicFeatures_1.26.2  
## [15] data.table_1.10.4        annotate_1.52.1         
## [17] Matrix_1.2-7.1           preprocessCore_1.36.0   
## [19] splines_3.3.2            BiocParallel_1.8.1      
## [21] stringr_1.2.0            RCurl_1.95-4.8          
## [23] biomaRt_2.30.0           rtracklayer_1.34.1      
## [25] multtest_2.30.0          pkgmaker_0.22           
## [27] openssl_0.9.6            GEOquery_2.40.0         
## [29] quadprog_1.5-5           codetools_0.2-15        
## [31] matrixStats_0.51.0       XML_3.98-1.5            
## [33] reshape_0.8.6            GenomicAlignments_1.10.0
## [35] MASS_7.3-45              bitops_1.0-6            
## [37] grid_3.3.2               xtable_1.8-2            
## [39] registry_0.3             DBI_0.5-1               
## [41] pacman_0.4.1             magrittr_1.5            
## [43] stringi_1.1.2            genefilter_1.56.0       
## [45] doRNG_1.6                nor1mix_1.2-2           
## [47] RColorBrewer_1.1-2       siggenes_1.48.0         
## [49] tools_3.3.2              illuminaio_0.16.0       
## [51] rngtools_1.2.4           survival_2.40-1         
## [53] AnnotationDbi_1.36.2     beanplot_1.2            
## [55] memoise_1.0.0
{% endhighlight %}



