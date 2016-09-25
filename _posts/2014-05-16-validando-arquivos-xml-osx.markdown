---
layout: post
title:  "Validando arquivos XML com XSD via linha de comando no Mac OSX"
date:   2014-05-16 22:00:00 -0300
categories: osx
---
Olá,

Recentemente precisei de um validador xml que validasse um arquivo xml de acordo com um schema xsd, e descobri que o mac já vem com um utilitário via linha de comando incluído na biblioteca libxml para fazer exatamente isso.

{% highlight shell %}
xmllint --noout --schema arq_schema.xsd arquivo_validar.xml
{% endhighlight %}

Até+