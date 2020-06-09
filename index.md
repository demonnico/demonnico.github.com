---
layout: page
title: NICO
tagline: we must chuck something in this shit life.
---

------------

喜欢技术，喜欢音乐。写过J2me、android，现在做iOS。

无聊时打打游戏，听听音乐，弹弹吉他。

[Github](https://github.com/demonnico)      ||
[Weibo](http://weibo.com/demont)  		    ||
[Twitter](https://twitter.com/NichoIasTau)

------------

{% include JB/setup %}

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
------------

<!--## To-Do

This theme is still unfinished. If you'd like to be added as a contributor, [please fork](http://github.com/plusjade/jekyll-bootstrap)!
We need to clean up the themes, make theme usage guides with theme-specific markup examples.-->


