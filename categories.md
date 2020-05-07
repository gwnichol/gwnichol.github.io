---
layout: page
permalink: /categories:output_ext
title: Categories
---

{% for category in site.categories %}
   {% capture category_name %}{{ category | first }}{% endcapture %}
   {% capture coll_name %}{{ category_name | slugize }}{% endcapture %}
  <div class="card">
    <div class="card-header" id="label-{{ coll_name }}">
      <h2 class="mb-0">
        <button class="btn btn-link" type="button" data-toggle="collapse" data-target="#collapse-{{ coll_name }}" aria-expanded="true" aria-controls="collapse-{{ coll_name }}">
        {{ category_name }}
        </button>
      </h2>
    </div>

    <div id="collapse-{{ coll_name }}" class="collapse show" aria-labelledby="label-{{ coll_name }}" >
      <div class="card-body">
          {% for post in site.categories[category_name] %}
    <article class="archive-item">
      <h4><a href="{{ site.baseurl }}{{ post.url }}">{{post.title}}</a></h4>
    </article>
    {% endfor %}

      </div>
    </div>
  </div>
{% endfor %}
