---
layout: post
title:  "Jinja Basics"
author: skalavala
categories: [ jinja ]
image: assets/images/jinja.png
jinja: true
---

Want to learn basics of Jinja? Let's get started...

Sample Code:


```
{% raw %}
{% macro humidex(T, H) %}
  {% set t = 7.5*T/(237.7+T) %}
  {% set et = 10**t %}
  {% set e= 6.112 * et * (H/100) %}
  {% set humidex = T+(5/9)*(e-10) %}
  {% if humidex < T %}
    {% set humidex = T %}
  {% endif %}
  {{humidex}}
{% endmacro %}

{{ humidex(23,45) }}
{% endraw %}
```

The output would be the following:

```
24.455823489173447
```
