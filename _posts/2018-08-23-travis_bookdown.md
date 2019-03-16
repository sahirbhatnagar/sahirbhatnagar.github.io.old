---
layout: post
title: Getting Travis to Auto Push to GitHub
date: 2018-08-23 15:09:00
comments: yes
---

Following the advice given on the bookdown [site](https://bookdown.org/yihui/bookdown/github.html), I use Travis-CI to automatically build my bookdown book and push it to `gh-pages`. I always forget how to do this, so I'm writing these notes to supplement what is already written there. 

1. Follow [these instructions on GitHub](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/) to create a personal access token (PAT)
2. Copy the generated PAT to your clipboard  
3. From the command line and within the root directory of your repository hosting the source of the bookdown, enter the following command:  

{% highlight bash %}
travis encrypt GITHUB_PAT="paste_your_PAT_here" --add
{% endhighlight %}

Note 1: I'm not entirely sure that I need to generate a new PAT for each bookdown repository that I create, but so far this strategy has worked for me. Comments are welcome.    

Note 2: that the PAT must be within double quotes. the `--add` flag will automatically add the encrypted token to the `.travis.yml` file.  

Note 3: if you get some error about a bad interpretor, trying re-installing travis:

{% highlight bash %}
gem install travis
{% endhighlight %}

<br><br>  


