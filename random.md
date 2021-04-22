<title>
   {%if page.title %}
       {{ page.title }}
   {% else %}
       {{ site.title }}
   {% endif %}
</title>

<meta itemprop="description" name="description" content="{% if page.description %}{{ page.description | truncate: 160 }}{% else %}{{ site.description | truncate: 160  }}{% endif %}" />

---
title: How to Create a Beautiful Jekyll Blog?
description: I created this beautiful looking Jekyll blog by forking a repository. You can also fork it to make it yours. Jekyll is a simple blog generator. The community is growing and the number of plugins is also growing. I have moved all my blogs to Jekyll!
---

permalink: /:title/