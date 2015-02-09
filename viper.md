---
layout: page
title: VIPER Project Log
permalink: /viper/
---

VIPER is the idea and a DIY project to assemble and implement a "virtual presence robot", controlled by an operator on a
dedicated web site, and having a biderctional secured video and audio transmission over the Intenet.

This project is largely a set of expriments and studies aroud the final goal, and here I will be logging my ideas and 
plans, and findings, among which, I hope, some might be useful for you.

Short summary of a current state of things:

 - a simple robot based on a combnation of Arduino and Android is built, and
 - can be controlled from a web site, thus, over the Internet, and
 - I'm now trying to add an audio and video transmission from the robot to the web site used by an operator

Below are my thoghts and finding logged during the project:

<h2 id="{{ category }}">{{ category }}</h2>
<ul>
{% for post in site.categories["arduino"] %}
<li>{{ post.date | date: "%b %-d, %Y" }} <a href="{{ post.url }}">{{ post.title }}</a></li>
{% endfor %}
</ul>

<p class="rss-subscribe">subscribe <a href="{{ "/feed.xml" | prepend: site.baseurl }}">via RSS</a></p>
