---
layout: post
title: Github pages + jekyll build person site
tags: [Build]
---

github 提供很方便的方法用搭建自己的站点,比起wordpress要简单方便很多,相对而言功能比较单一,不过对我而言也足够了.

## Builed github pages
[Github pages](http://pages.github.com/),简单说建立新的repository,使用github指定的site address就可以进行基础的访问.
需要注意的是参考其他guides可能会发现要求建立除了master以外的gh-pages,而这就涉及到[User,Organization,and Project
Pages](https://help.github.com/articles/user-organization-and-project-pages),我的理解是master是建立site的发布分支,通过web访问http://branzhang.github.io/github提供类型的网址都是在请求的此分支的内容,gh-pages用于绑定domain后访问的内容,gh-pages还有作用就是对于已经建立的repo,比如[OSC](https://github.com/BranZhang/OSC),建立gh-pages分支后可以进行web[链接](http://branzhang.github.io/OSC/).

## Easily keep gh-pages in sync with master
{% highlight Bash%}
git checkout gh-pages // go to the gh-pages branch
git rebase master // bring gh-pages up to date with master
git push origin gh-pages // commit the changes
git checkout master // return to the master branch
{% endhighlight %}

## Introducing Jekyll
[Jekyll](http://jekyllrb.com) is a static site generator, an open-source tool for creating simple yet powerful websites of all shapes and sizes. From [the project's readme](https://github.com/mojombo/jekyll/blob/master/README.markdown):

  > Jekyll is a simple, blog aware, static site generator. It takes a template directory [...] and spits out a complete, static website suitable for serving with Apache or your favorite web server. This is also the engine behind GitHub Pages, which you can use to host your project’s page or blog right here from GitHub.

Jekyll takes your content, renders Markdown and Liquid templates, and spits out a complete, static website ready to be served by Apache, Nginx or another web server.

Remember Jekyll is a processing engine. There are two main types of parsing in Jekyll.

Content parsing.
This is done with textile or markdown.
Template parsing.
This is done with the liquid templating language.

### Use lanyon theme
使用开源的主题可以节省很多订制时间,也方便参考修改
[Lanyon](https://github.com/poole/lanyon) is an unassuming Jekyll theme that places content first by tucking away navigation in a hidden drawer. It's based on Poole, the Jekyll butler.

### What is [Liquid](http://jekyllbootstrap.com/lessons/jekyll-introduction.html#toc_16)?

Liquid is a secure templating language developed by Shopify. Liquid is designed for end-users to be able to execute logic within template files without imposing any security risk on the hosting server.

Jekyll uses Liquid to generate the post content within the final page layout structure and as the primary interface for working with your site and post/page data.

_layouts目录下面存放这些模板

### What is [Makedown](http://daringfireball.net/projects/markdown/)?
Markdown is a text-to-HTML conversion tool for web writers. Markdown allows you to write using an easy-to-read, easy-to-write plain text format, then convert it to structurally valid XHTML (or HTML).


###### 参考
[http://24ways.org/2013/get-started-with-github-pages/](http://24ways.org/2013/get-started-with-github-pages/)
[https://help.github.com/categories/20/articles](https://help.github.com/categories/20/articles)
[http://joshualande.com/2014/02/03/jekyll-github-pages-poole/](http://joshualande.com/2014/02/03/jekyll-github-pages-poole/)

Take a gander at some existing [Sites](https://github.com/jekyll/jekyll/wiki/sites)


