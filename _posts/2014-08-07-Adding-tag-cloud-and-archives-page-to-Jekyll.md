---
layout: post
title: Adding tag cloud and archives page to Jekyll
tags:
- jekyll
published: true
---

This blog is running on [Jekyll](http://jekyllrb.com/), hosted by [GitHub](https://github.com/).

I wanted a tag cloud and archives page for it, but most plugins would\'t work well with Jekyll version 2.2.x.

I spent nearly two days before I found [this post](http://kalapun.com/blog/2014/01/27/liquid-tag-management-for-jekyll/), and finally got it working.

Here is a shortcut to a working tag cloud / archives page for people like me:

----

## 1. Create a tags page

Generate a file named `tags.html`.

{% highlight bash %}
$ cd /path/to/my-jekyll-project/
$ touch tags.html
{% endhighlight %}

Then fill it with following lines:

{% highlight liquid %}
{% raw %}
---
layout: page
permalink: /tags/
---

<ul class="tag-cloud">
{% for tag in site.tags %}
  <li style="font-size: {{ tag | last | size | times: 100 | divided_by: site.tags.size | plus: 70  }}%">
    <a href="#{{ tag | first | slugize }}">
      {{ tag | first }}
    </a>
  </li>
{% endfor %}
</ul>

<div id="archives">
{% for tag in site.tags %}
  <div class="archive-group">
    {% capture tag_name %}{{ tag | first }}{% endcapture %}
    <h3 id="#{{ tag_name | slugize }}">{{ tag_name }}</h3>
    <a name="{{ tag_name | slugize }}"></a>
    {% for post in site.tags[tag_name] %}
    <article class="archive-item">
      <h4><a href="{{ root_url }}{{ post.url }}">{{post.title}}</a></h4>
    </article>
    {% endfor %}
  </div>
{% endfor %}
</div>
{% endraw %}
{% endhighlight %}

Now this page is accessible through url: [http://to.your.jekyll/tags/](/tags/).

----

## 2. Display tags in your posts

Create an include file for embedding in the posts:

{% highlight bash %}
$ touch _includes/post-tags.html
{% endhighlight %}

with following content:

{% highlight liquid %}
{% raw %}
<div class="post-tags">
  Tags: 
  {% if post %}
    {% assign tags = post.tags %}
  {% else %}
    {% assign tags = page.tags %}
  {% endif %}
  {% for tag in tags %}
  <a href="/tags/#{{tag|slugize}}">{{tag}}</a>{% unless forloop.last %},{% endunless %}
  {% endfor %}
</div>
{% endraw %}
{% endhighlight %}

and include it - `{% raw %}{% include post-tags.html %}{% endraw %}` - in the post layout:

{% highlight bash %}
$ vi _layouts/post.html
{% endhighlight %}

{% highlight liquid %}
{% raw %}
---
layout: default
---
<article class="post">
  <h1>{{ page.title }}</h1>
  
  {% include post-tags.html %}

  <div class="entry">
    {{ content }}
  </div>

  <div class="date">
    Written on {{ page.date | date: "%B %e, %Y" }}
  </div>

  {% include disqus.html disqus_identifier=page.disqus_identifier %}
</article>
{% endraw %}
{% endhighlight %}

----

## 3. Edit the styles.css file

{% highlight bash %}
$ vi styles.css
{% endhighlight %}

Add following:

{% highlight scss %}
// for tag cloud and archives
.tag-cloud {
  list-style: none;
  padding: 0;
  text-align: justify; 
  font-size: 16px;
  li {
    display: inline-block;
    margin: 0 12px 12px 0; 
  }
}
#archives {
  padding: 5px;
}
.archive-group {
  margin: 5px;
  border-top: 1px solid #ddd;
}
.archive-item {
  margin-left: 5px;
}
.post-tags {
  text-align: right;
}
{% endhighlight %}

----

## 4. Add tags to your posts

Add tags in the [front matter](http://jekyllrb.com/docs/frontmatter/) of your posts.

This is an example:

{% highlight liquid %}
{% raw %}
---
layout: post
title: This is an example.
tags:
- jekyll
- example
published: true
---

This is a post for an example.
{% endraw %}
{% endhighlight %}

----

## All done!

