---
layout: post
title: CDPATH in Bash
date: 2014-07-04 11:03:16 -0400
description: symbolic links
---

Instead of constantly typing the full path when using the `cd` command, `BASH` has a built-in feature called `CDPATH`. Thanks to _lhunath_ who explained in this [SO Post](http://stackoverflow.com/questions/670488/how-to-manage-long-paths-in-bash) how to use this feature. 

<!--more-->

The first time you do this, you need to create a hidden folder and add **CDPATH** to your bashrc (note this step only needs to be done once):

{% highlight bash %}
mkdir ~/.paths
cd ~/.paths
echo 'CDPATH=~/.paths' >> ~/.bashrc
{% endhighlight %}

Then to add symbolic links use:

{% highlight bash %}
ln -s /my/very/long/path/name/to/my/project project
ln -s /some/other/very/long/path/to/my/backups backups
echo 'CDPATH=~/.paths' >> ~/.bashrc
{% endhighlight %}


To update your bashrc:

{% highlight bash %}
source ~/.bashrc
{% endhighlight %}

Now you can enter the folders from anywhere by simply typing

{% highlight bash %}
cd projects
cd backups
{% endhighlight %}




