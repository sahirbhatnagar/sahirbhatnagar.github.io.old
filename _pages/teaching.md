---
layout: page
permalink: /teaching/
title: teaching
description: Materials for full term courses, short courses and tutorials that I've taught or created.
years: [2021, 2019, 2018, 2017, 2016, 2015, 2013]
---


{:.no_toc}

* Will be replaced with the ToC, excluding the "Contents" header
{:toc}


## Graduate Courses
<hr>


{% bibliography -f papers -q @*[status=course]* %}



## Short Courses



{% for y in page.years %}
  <h3 class="year">{{y}}</h3>
  {% bibliography -f papers -q @*[status=short,year={{y}}]* %}
{% endfor %}





## Tutorials
<hr>

{% bibliography -f papers -q @*[status=tutorials]* %}






