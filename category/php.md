---
layout: default
title: PHP Articles
category: php
---

{% for category in site.categories %}
    {% if category.first == page.category %}
        {% for post in category.last %}
* [{{ post.title }} {{ post.date | date: "（%Y年%m月%d日）" }}]({{ site.baseurl }}{{ post.url }})

> {{ post.excerpt | strip_html }}
>

        {% endfor %}
    {% endif %}
{% endfor %}
