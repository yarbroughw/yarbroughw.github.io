---
layout: default
title: Home
---

# Willem Yarbrough

I'm a computer scientist and coder living in San Francisco.  
I'm currently looking for work in the SF Bay Area. [Hire
me!](https://www.linkedin.com/in/yarbroughw)

+ [View my resume.](resume.pdf)
+ [Check out my projects.](http://github.com/yarbroughw)
+ [Read my blog.](blog)

## Latest Blog Posts:
{% for post in site.posts %} * {{ post.date | date_to_string  }} &raquo; [ {{ post.title  }}  ]({{ post.url  }})
{% endfor %}

