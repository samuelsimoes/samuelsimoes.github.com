---
layout: post
title:  "Pequenos métodos que traduzem condições"
date:   2013-08-16 00:30
categories: programacao
icon: "icon-thumbs-up"
---

Uma *talk* que abriu os meus olhos pra uma coisa que eu vacilava foi a **[Refactoring from Good to Great do Ben Orenstein](http://www.youtube.com/watch?v=DC-pQPq0acs)**, se você não assistiu pare tudo que está fazendo e assista, independente da linguagem de programação que você utiliza (na talk o Ben usa Ruby, mas os exemplos são facilmente entendidos em qualquer outra linguagem).

Nessa talk são apresentados alguns conceitos para um bom código, um deles que vale a pena frisar é o conceito de **pequenos métodos** que abstraem condições.

Pequenos métodos que "traduzem condições" tendem a facilitar muito o entendimento de rotinas, deixando elas mais semânticas, guiando de forma clara quem está lendo o seu código.

No código abaixo exemplifico um cenário onde tenho um carrinho de compras com uma determinada regra de quando ele pode ser finalizado, vejamos a implementação.

{% highlight javascript %}
ShopCart = function () {
  this.finish = function () {
    if (this.total() <= 500 && this.items.length <= 3) {
      throw new Error("Finish cart without the minimum value to finish.");
    }

    alert("Your order has been registered.");
  };
};
{% endhighlight %}

Perceba que o método responsável por finalizar o carrinho contém uma rotina não muito complexa, mas que você vai ter que tirar alguns segundos para entender a condição na terceira linha, em situações mais complexa isso pode se tornar um problema bem maior que pode piorar caso você precise daquela condição em outras áreas deste código.

Uma forma de refatorar isso seria:

{% highlight javascript %}
ShopCart = function () {
  this.haveMinimumProductsToFinish = function () {
    return this.items.length >= 3;
  };

  this.haveMinimumValueToFinish = function () {
    return this.total() >= 500;
  };

  this.canFinish = function () {
    return this.haveMinimumValueToFinish() && this.haveMinimumProductsToFinish();
  };

  this.finish = function () {
    if (!this.canFinish()) {
      throw new Error("Can't finish this cart.");
    }

    alert("Your order has been registered.");
  };
};
{% endhighlight %}

O número de linhas aumentou, mas concentramos as condições em pequenos métodos que expressam de uma forma bastante semântica o que elas significam na regra do nosso objeto, sem falar que vamos conseguir consultar essas condições sem repetir elas pelo código.

É isso, dica rápida que se você não aplica nos seus códigos tem que começar agora.

Até a próxima.
