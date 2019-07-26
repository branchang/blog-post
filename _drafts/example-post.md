---
title: "Example Post"
date: 1900-01-01 00:00:00 +0800
category: Test
tags: [Testing]
bid: 201801020
---


## Typograpy

# H1

<h2 data-toc-skip>H2</h2>

<h3 data-toc-skip>H3</h3>

#### H4


## Tables

|Company|Contact|Country|
|:---|:--|---:|
|Alfreds Futterkiste | Maria Anders | Germany
|Island Trading | Helen Bennett | UK
|Magazzini Alimentari Riuniti | Giovanni Rovelli | Italy

## Hightlight

This is a `sample` to `code hightlight`.

## Link

[http://127.0.0.1:4000](http://127.0.0.1:4000)


## Footnote

Click the hook will locate the footnote. [^footnote]


## Image

![Desktop View](https://cotes.gitee.io/blog-pics/1808110/1.jpg)


## Code Snippet

### Liquid

{% highlight liquid %}
{% raw %}
{% highlight html %}
<p>This is some text in a paragraph.</p>
{% endhighlight %}
{% endraw %}
{% endhighlight %}

### Line Number

**No Scrolling**

{% capture _code %}
{% highlight html linenos %}
{% raw %}
<div class="panel-group">
  <div class="panel panel-default">
    <div class="panel-heading" id="{{ category_name }}">
      <i class="far fa-folder"></i>&nbsp;
      </a>
    </div>
  </div>
</div>
  {% endif %}
{% endfor %}
{% endraw %}
{% endhighlight %}
{% endcapture %}{% include fixlinenos.html %}{{ _code }}

**Horizontal Scrolling**

{% capture _code %}
{% highlight html linenos %}
{% raw %}
<div class="panel-group">
  <div class="panel panel-default">
    <div class="panel-heading" id="{{ category_name }}">
      <i class="far fa-folder"></i>
      <p>This is a very long long long long long long long long long long long long long long long long long long long long long line.</p>
      </a>
    </div>
  </div>
</div>
  {% endif %}
{% endfor %}
{% endraw %}
{% endhighlight %}
{% endcapture %}{% include fixlinenos.html %}{{ _code }}


***


[^footnote]: The footnote source.
