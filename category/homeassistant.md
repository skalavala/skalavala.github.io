---
layout: page
title: Posts that have "Home Assistant" tags
comments: true
---

  {% for page in (1..paginator.total_pages) %}
    {% if page == paginator.page %}
      <span class="ml-1 mr-1">{{ page }}</span>
    {% elsif page == 1 %}
      <a  class="ml-1 mr-1" href="{{ paginator.previous_page_path | prepend: site.baseurl | replace: '//', '/' }}">{{ page }}</a>
    {% else %}
      <a  class="ml-1 mr-1" href="{{ site.paginate_path | prepend: site.baseurl | replace: '//', '/' | replace: ':num', page }}">{{ page }}</a>
    {% endif %}
  {% endfor %}
  
