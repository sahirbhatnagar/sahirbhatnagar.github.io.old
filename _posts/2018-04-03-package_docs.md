---
layout: post
title: Documenting R Packages
date: 2018-04-03 15:09:00
---


Some brief notes on my `R` package documentation workflow.


## `styler`

I first use the [`styler`](http://styler.r-lib.org/index.html) package for pretty-printing of `R` source code

```r
pacman::p_load_gh("r-lib/styler")
# run style_dir on R directory
styler::style_dir("./R")
```


## `prefixer`

Next I use the [`prefixer`](https://github.com/dreamRs/prefixer) package to prefix all my functions with their NAMESPACE

```r
pacman::p_load_gh("dreamRs/prefixer")
#  launch the addin via RStudio's Addins menu
```


## `sinew`

In a third step, I use the [`sinew`](https://github.com/metrumresearchgroup/sinew) package to generate a roxygen2 skeleton on all my `R` source code files

```r
pacman::p_load_gh("metrumresearchgroup/sinew")
# the main function I use is sinew::makeOxyFile()
sinew::makeOxyFile("./R/methods.R")
```

## `devtools`

Finish with `devtools::document()`. 
