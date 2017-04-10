---
layout: post
title:  "Listando portas usadas por processos no Mac OSX"
date:   2014-05-16 22:00:00 -0300
categories: osx
---
Para listar quais as portas em modo de escuta usadas por processos no Mac OSX digite o comando abaixo no Terminal:

{% highlight shell %}
lsof -i | grep LISTEN
{% endhighlight %}
