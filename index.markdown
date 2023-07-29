---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
---

{% for post in site.posts %}
  <div class="post">
    <a class="post-link" href="{{ post.url }}">
      <h4 class="post-title">
        {{ post.title }}
      </h4>
    </a>
    <p class="post-text">
      {{ post.excerpt }}
    </p>
    <p>
    {{ post.date | date: "%-d %B %Y" }}
    </p>
  </div>
  <hr />
{% endfor %}