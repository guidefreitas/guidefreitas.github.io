---
layout: post
title:  "Aplicações MVC Client-Side (Single-Page Applications)"
date:   2015-07-16 22:00:00 -0300
categories: Programming
---
Olá

Recentemente participei do desenvolvimento do aplicativo [Gooparties](http://gooparties.com) onde utilizamos uma arquitetura totalmente client-side utilizando o framework Backbone. Neste post vou falar um pouco sobre esse tipo de arquitetura, quais as características e como essa arquitetura se diferencia do tradicional MVC utilizado em frameworks server-side como Rails, ASP.NET MVC, Zend entre outros.

## O que é uma arquitetura MVC client-side (também conhecida como single-page applications)?

Antes de começarmos a falar de aplicações client-side vamos dar uma olhada nos modelos mais conhecidos de aplicações web.
No modelo tradicional (server-side) MVC temos os componentes Model, View e Controller sendo executados no servidor, gerando o html e enviando esse html para ser visualizado no browser do usuário. Nesse modelo o Model, Controller e View são executados no lado servidor.

![Server Side MVC Image](/images/2015-07-16-aplicacoes-mvc-client-side/server_side_mvc.png)

Em um formato um pouco mais evoluído temos aplicações onde o html ainda é gerado no lado servidor mas são utilizadas técnicas como ajax para atualizar de partes da página ou funcionalidades implementadas em javascript em alguns componentes para melhorar a experiência do usuário evitando um reload completo da página a cada interação do usuário. Neste modelo o Model, Controller e View ainda são executados no servidor, mas algumas funcionalidades da View passam a ser executadas no lado cliente.

![Hibrid Mode MVC Image](/images/2015-07-16-aplicacoes-mvc-client-side/hibrid_mode_mvc.png)

O terceiro modelo são as aplicações client-side. Nesse modelo a responsabilidade de controller e view são executadas no lado cliente. O Model passa a ter duas partes, uma executada no lado servidor, quer fornece a API e validações e outra executada também no lado cliente, que também fornece validações e informações sobre a estrutura do modelo.

![Hibrid Mode MVC Image](/images/2015-07-16-aplicacoes-mvc-client-side/client_side_mvc.png)

## Quais as vantagens de utilizar MVC client-side?


### Melhor experiência para o usuário
A principal vantagem é um ganho muito grande no desempenho da aplicação. Como o browser não precisa fazer requisições, download e parser de páginas completas a cada interação do usuário o resultado é uma aplicação que responde muito mais rápido aos comandos. Aplicações construídas com a arquitetura client-side funcionam de uma forma tão natural quanto uma aplicação desktop instalada na máquina do usuário. 

### Melhor desempenho na transferência de dados  
Existe também um ganho considerável em velocidade na transmissão dos dados, pois ao invés de trafegar o conteúdo html completo a cada interação do usuário, na arquitetura client-side o aplicativo completo é transferido na primeira requisição e as requisições seguintes são responsáveis por trafegar apenas os dados brutos entre o cliente e o servidor, normalmente no formato JSON. Este ganho é visível em aplicações móveis onde a velocidade na transferência dos dados normalmente é baixa. 

### Redução de carga no lado servidor
O aplicativo completo passa a ser fornecido através de arquivos html, css e javascript que podem ser comprimidos e distribuídos através de CDN's com facilidade. Após baixados pela primeira vez esses arquivos são mantidos em cache no browser do usuário. O servidor fica responsável apenas por fornecer uma API e enviar e receber os dados brutos no formato JSON. Dessa forma todo o processamento responsável por parsing dos dados e geração de templates fica no lado cliente e não mais no servidor, liberando recursos. 

### Facilidade de manutenção
Como aplicações client-side dependem apenas da API fornecida pelo servidor, as manutenções no lado servidor podem ser feitas de forma independente e transparente para o lado cliente (desde que não mude a API, obviamente). Da mesma forma, o lado cliente pode ser alterado sem a necessidade de alterar nada no lado servidor.

### Gerenciamento de equipes
Como o lado cliente e o lado servidor passam a ser desenvolvidos de forma completamente independente, a única coisa necessária para os dois times trabalharem em paralelo é a definição da estrutura da API. Com esta definição pronta ambos os times podem trabalhar em paralelo onde o time front-ent desenvolve toda a parte client-side utilizando uma API com dados fictícios enquanto o time de back-end desenvolve a parte servidora se preocupando apenas em respeitar a especificação da API acordada entre as equipes.

### Facilidade de inclusão de novos front-ends
Como o lado servidor fornece apenas a API, torna-se muito mais fácil desenvolver novos front-ends para dispositivos específicos como uma aplicação nativa para iOS ou Android, por exemplo.


## Os frameworks MVC client-side substituem totalmente os frameworks MVC server-side?

Não. Os frameworks server-side passam a fornecer apenas uma API, normalmente no formato REST, para ser consumida pela aplicação MVC no lado cliente. Toda a parte de controle de rotas, renderização de templates e validação ficam no lado cliente. A parte servidora fica responsável pelas validações (novamente? sim, as validações devem ocorrer tanto no lado cliente quanto no lado servidor) e por armazenar e recuperar os dados em um banco de dados.


## Frameworks

Existem diversos frameworks MVC client-side mas alguns que vem ganhando mais destaque ultimamente são:

- [Backbone.js](http://backbonejs.org/)
- [Ember.js](http://emberjs.com/)
- [Angular.js](angularjs.org)

Alguns outros frameworks que possuem a parte MVC e mais alguns componentes gráficos.

- [Sencha Touch](http://www.sencha.com/products/touch)
- [ExtJS](http://www.sencha.com/products/extjs)

## Quais as desvantagens de utilizar MVC client-side?

A principal dificuldade é a necessidade de aprender mais um (ou as vezes mais do que um) framework específico para trabalhar exclusivamente com o front-end. A inclusão dessa parte da aplicação, apesar de facilitar a manutenção como comentado anteriormente, adiciona uma nova camada na aplicação, que precisa ser compreendida e respeitada pelo time. 
Outro fator importante a ser considerado é o fato de que aplicações client-side necessitam da execução do código javascript para gerarem o conteúdo html e exibi-lo ao usuário. Apesar de praticamente não existirem usuários com javascript desativado em seus navegadores, os mecanismos de busca ainda tem dificuldade em indexar páginas com conteúdo gerado dinamicamente no lado cliente. Se o seu projeto exige que o conteúdo do seu aplicativo seja indexado por mecanismos de busca, talvez adotar uma arquitetura puramente mvc client-side não sejam a melhor opção.

Resolvi fazer esse post de introdução sobre a arquitetura MVC client-side para demonstrar as vantagens deste tipo de aplicação. Em posts futuros vou falar mais sobre o Backbone e como utilizá-lo na construção de aplicações.