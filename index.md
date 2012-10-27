---
layout: page
---
{% include JB/setup %}

<ul class="posts">
  {% for post in site.posts %}
    <li>
      <h1><a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></h1>
      <span>{{ post.date | date_to_string }}</span>
      Tags: 
      {% for tag in post.tags %}
        <a href="{{ BASE_PATH }}{{ site.JB.tags_path }}#{{ tag }}-ref">{{ tag }}</a>, 
      {% endfor %}
      
      Categroies: 
      {% for category in post.categories %}
        <a href="{{ BASE_PATH }}{{ site.JB.categories_path }}#{{ category }}-ref">{{ category }}</a>
      {% endfor %}
      <br>
      {{ post.id }}
      <br>
      {{ post.content }}
      <hr>
    </li>
  {% endfor %}
</ul>