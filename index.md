---
layout: page
# title: Hello World!
tagline: Supporting tagline
---
{% include JB/setup %}


  <ul class="post-list">
    {% for post in site.posts %}
      <li>

        <h2>
            <a class="post-link" href="{{ post.url | prepend: site.baseurl }}"> {{ post.title}} </a>
        </h2>
         
        <p class="post-meta">{{ post.date | date: "%b %-d, %Y" }} </p>

        <p class="post-digest"> {% if post.digest %} {{ post.digest }} {% endif %}</p> 
      </li>
      <br/><br/>
    {% endfor %}
  </ul>


<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

