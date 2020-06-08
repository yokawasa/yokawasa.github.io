---
layout: post
---

{% if site.paginate %}
  {% assign posts = paginator.posts %}
{% else %}
  {% assign posts = site.posts %}
{% endif %}

{%- if posts.size > 0 -%}
  <ul class="post-list">
  {%- assign date_format = site.minima.date_format | default: "%b %-d, %Y" -%}
  {% assign counter = 0 %}
  {%- for post in posts -%}
    {%- if counter < 10 -%}
      <li>
        <span class="post-meta">{{ post.date | date: date_format }}</span>
        <h2>
          <a class="post-link" href="{{ post.url | relative_url }}">
            {{ post.title | escape }}
          </a>
        </h2>

        <div class="post-content e-content" itemprop="articleBody">
          {{ post.content }}
        </div>
      </li>
    {%- endif -%}
    {% increment counter %}
  {%- endfor -%}
  </ul>

{%- endif -%}
