---
title: Loops in Jekyll Liquid templates
tags: jekyll
---

This blog is generated using Jekyll and the templating engine is Liquid. I
wanted to display `var tags = [ tag1, tag2, tag3 ]` with the tags of the post
at the end of the page. Note that the comma after `tag3` is absent.

For the posts, I wanted to display the list of tags, separated by a comma. When
looping, the templating engine defines a few extra variables that can be used
to implement that very easily.

```
forloop.length      # => length of the entire for loop
forloop.index       # => index of the current iteration
forloop.index0      # => index of the current iteration (zero based)
forloop.rindex      # => how many items are still left?
forloop.rindex0     # => how many items are still left? (zero based)
forloop.first       # => is this the first iteration?
forloop.last        # => is this the last iteration?
```

The code to display the `var tags = [ tag1, tag2, tag3 ]` at the bottom of the
page is implemented by:

```
{% raw  %}{% for tag in page.tags %}
  {% capture tag_name %}{{ tag }}{% endcapture %}
  {% unless forloop.last %}
      <a href="/tag/{{ tag_name }}"><nobr>{{ tag_name }}</nobr></a>,
  {% else %}
      <a href="/tag/{{ tag_name }}"><nobr>{{ tag_name }}</nobr></a>
  {% endunless %}
{% endfor %}{% endraw %}
```