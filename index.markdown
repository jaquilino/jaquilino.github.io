---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home
#layout: default
---

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
      <br />{{ post.excerpt | strip_html | truncatewords:75 }}
      <br />Posted <span>{{ post.date | date_to_string }}</span>
    </li>
  {% endfor %}
</ul>

<div class="footer border-top border-gray-light mt-5 pt-3 text-gray">
   <p>All posts appearing here are the opinions of the author (that would be me).
   All rights reserved</p>
   <p><a href="/about/">About this page</a></p>
   <p><a href="mailto:{{ site.email }}">Contact the author</a></p>
</div>

