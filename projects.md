---
layout: page
title: Projects
permalink: /projects/
modified: 2014-05-29
---

{% assign sorted_pages = (site.pages | sort: 'priority') %}

#Musical interfaces
{% for page in sorted_pages %}
{% if page.category == "musicalinterface" %}
* [{{ page.title }}]({{ site.url }}{{ page.url }}) {{page.description}}
{% endif %}
{% endfor %}


#Wearable Electronics
{% for page in sorted_pages %}
{% if page.category == "wearable" %}
* [{{ page.title }}]({{ site.url }}{{ page.url }}) {{page.description}}
{% endif %}
{% endfor %}
