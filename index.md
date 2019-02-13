---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: page
title: Recent Articles
permalink: /
---

<div class="home">
  {%- if site.posts.size > 0 -%}
    <ul class="post-list">
      {%- for post in site.posts limit: 5 -%}
      <li>
        {%- assign date_format = site.minima.date_format | default: "%b %-d, %Y" -%}
        <span class="post-meta">{{ post.date | date: date_format }} • {% include read-time.html content=post.content %} {% include author.html author=post.author %}</span>
        
        <h3>
          <a class="post-link" href="{{ post.url | relative_url }}">
            {{ post.title | escape }}
          </a>
        </h3>
        {%- if site.show_excerpts -%}
          {{ post.excerpt }}
        {%- endif -%}
      </li>
      {%- endfor -%}
    </ul>

    <p class="feed-subscribe"><svg class="svg-icon orange"><use xlink:href="{{ '/assets/minima-social-icons.svg#rss' | relative_url }}"></use></svg><a href="{{ "/feed.xml" | relative_url }}">Subscribe</a></p>
  {%- endif -%}
</div>
