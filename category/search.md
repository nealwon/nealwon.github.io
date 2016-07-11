---
layout: default
title: Search Engine Usages
category: search
---

{% for category in site.categories %}
    {% if category.first == page.category %}
        {% for post in category.last %}
* [{{ post.title }} {{ post.date | date: "（%Y年%m月%d日）" }}]({{ site.baseurl }}{{ post.url }})

<pre>
 {{ post.excerpt | strip_html }}
 </pre>

        {% endfor %}
    {% endif %}
{% endfor %}
