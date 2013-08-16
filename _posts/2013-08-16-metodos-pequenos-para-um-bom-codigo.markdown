---
layout: post
title:  "Métodos pequenos para um bom código"
date:   2013-08-16 00:30
categories: programacao
icon: "icon-thumbs-up"
---

Uma *talk* que abriu os meus olhos pra uma coisa que eu vacilava foi a **[Refactoring from Good to Great do Ben Orenstein](http://www.youtube.com/watch?v=DC-pQPq0acs)**, se você não assistiu pare tudo que está fazendo e assista, independente da linguagem de programação que você utiliza (na talk o Ben usa Ruby, mas os exemplos são facilmente entendidos em qualquer outra linguagem).

Nessa talk são apresentados alguns conceitos para um bom código, um deles que vale a pena frisar é o conceito dos **métodos realmente pequenos**, com apenas uma rotina e nada mais.

Pequenos métodos tendem a facilitar muito o entendimento de rotinas, você consegue guiar quem vai ler o seu código posteriormente de uma forma muito mais clara.

No código abaixo exemplifico um cenário onde tenho uma *collection* do [Backbone.js](http://backbonejs.org) onde sua URL é dinâmica, sendo definida de acordo com as opções do objeto.

{% highlight javascript %}
Products = Backbone.Collection.extend({
  model: Product,

  url: function() {
    if (this.options == undefined) {
      return;
    };

    if (this.options.account != undefined) {
      url += "/accounts/" + this.options.account.get("id") + "/products";
    } else {
      url += "/products"
    };

    if (this.options.search_term != undefined) {
      url += "/search?term=" + this.options.search_term;
    };

    this.url = url;
  },

  initialize: function(models, options) {
    this.options = options;
  }
});
{% endhighlight %}

Perceba que o método responsável por definir a URL contém uma rotina não muito complexa, mas que você vai ter que tirar alguns segundos para entender o que cada bloco de condição está fazendo ali, em situações mais complexa isso seria um problema bem maior.

Uma forma de refatorar isso seria:

{% highlight javascript %}
Products = Backbone.Collection.extend({
  model: Product,

  initialize: function(models, options) {
    this.options = options;
  },

  url: function() {
    this.defineUrl();
  },

  defineUrl: function() {
    if (this.options == undefined) {
      return;
    }

    this.defineUrlRootSearch();
    this.resolveSearchUrl();
  },

  defineUrlRoot: function() {
    if (this.options.account != undefined) {
      this.url += "/accounts/" + this.options.account.get("id") + "/products";
    } else {
      this.url += "/products";
    }
  },

  resolveSearchUrl: function() {
    if (this.options.search_term != undefined) {
      this.url += "/search?term=" + this.options.search_term;
    }
  }
});
{% endhighlight %}

Perceba que cada condição anterior no grande método **url** ganhou um próprio método com um nome explicativo para as novas pequenas rotinas geradas.

É isso, dica rápida que se você não aplica nos seus códigos tem que começar agora.
