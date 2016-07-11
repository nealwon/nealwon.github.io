---
layout: default
title: Golang Articles
category: go
---

{% for category in site.categories %}
    {% if category.first == page.category %}
        {% for post in category.last %}
* [{{ post.title }} {{ post.date | date: "（%Y年%m月%d日）" }}]({{ site.baseurl }}{{ post.url }})

> <span class="text-muted">{{ post.excerpt | strip_html }}</span>
>
        {% endfor %}
    {% endif %}
{% endfor %}
