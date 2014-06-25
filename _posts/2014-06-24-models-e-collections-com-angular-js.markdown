---
layout: post
title:  "Models e Collections com Angular.js"
date:   2014-06-24 23:50
categories: javascript
icon: "icon-cogs"
---

Uma coisa que me incomoda no Angular.js é a falta de opinião em uma área importante de qualquer aplicação, a manipulação dos dados. Vejo muitos exemplos com o Angular.js onde o desenvolvedor faz um request com o `$http` e atribui o array retornado pelo servidor em um `this.items` de um controller, o que é ok em casos simples, mas que começa a ficar nebuloso quando esse tipo de coisa aparece:

{% highlight javascript %}
MyController = function () {
  this.showEditControls = function () {
    return this.model.situation == "available" && this.model.price > 500;
  }
}; {% endhighlight %}

Perceba detalhes internos do "model" "vazando" no controller, piora se isso se repetir em outro objeto, onde você vai perder completamente a *single source of truth*, sem falar ainda que **[tell, dont't ask](http://robots.thoughtbot.com/tell-dont-ask)**.

O ideal na situação descrita acima seria algo do tipo:

{% highlight javascript %}
MyController = function () {
  this.showEditControls = function () {
    return this.model.canEdit();
  }
}; {% endhighlight %}

Só que usando objetos literais retornados dos requests você não vai conseguir fazer algo do tipo. Para sanar o problema você tem algumas alternativas.

A primeira seria: instanciar um objeto injetando no seu construtor os dados do JSON retornado de um request ajax simples e esse objeto iria desempenhar o papel de "model" (poderíamos definir isso como um decorator, mas não vou entrar nesse mérito) contendo o método `#canEdit` citado acima. Eu gosto dessa alternativa, mas vou me pegar escrevendo o boilerplate para sincronizar esse objeto com o servidor por vários locais do código e aí que entra a segunda alternativa.

O [Backbone.js](http://backbonejs.org), que apesar de pouquíssimo opiniado, manipula os dados de uma forma que fita bem para a maioria das aplicações que cuido, definindo entidades que representam coleções de models. Já existem alguns módulos para Angular.js que tentam prover estrutura similar, mas nenhum deles me agradou porque a maioria foca demais na ação do request e pouco na manipulação dos dados.

Com tudo isso em mente eu criei um pequeno módulo que copia o design das collections e models do Backbone.js que você pode **[acessar o código aqui](https://gist.github.com/samuelsimoes/537a092132fe64ab1ed3)**. O módulo não e muito complexo e não faz tudo que a coleção e model do Backbone fazem, portei apenas alguns dos métodos mais importantes (`Collection#fetch, Model#fetch, Model#save, Collection#add` e outros mais que você pode ver no código), entretanto você pode usar a estrutura de models e collections do Backbone.js no Angular, mas vai ter que lidar com a adição provavelmente desnecessária do Underscore.js, jQuery e de todo o Backbone, o que pode comprometer o tempo de carregamento do seu aplicativo.

O uso do módulo é idêntico ao uso da estrutura do Backbone.js; abaixo defino um model e uma collection que utiliza esse model, tudo isso utilizando factories do Angular.js.

{% highlight javascript %}
angular.module("MyApp", ["ModelCollection"]);

angular.module("MyApp").factory("MyModel", [
  "ModelBase",
  function (ModelBase) {
    return ModelBase.extend({
      urlRoot: "my_collection_endpoint",

      canEdit: function () {
        return this.attributes.situation == "available";
      }
    });
  }
]);

angular.module("MyApp").factory("MyCollection", [
  "CollectionBase", "MyModel",
  function (CollectionBase, MyModel) {
    return CollectionBase.extend({
      model: MyModel,

      url: "my_collection_endpoint",

      // Defino um método qualquer pertinente ao estado da minha coleção
      moreThanTwentyItems: function () {
        return this.models.length > 20;
      }
    });
  }
]);
{% endhighlight %}

E depois, no meu controller, faço o uso da collection.

{% highlight javascript %}
angular.module("MyApp").controller("MyController", [
  "MyCollection",
  function (MyCollection) {
    var collection = new MyCollection();

    this.items = collection.models;

    collection.fetch().then(function () {
      // O array já vem devidamente mapeado com o meu model e o prévio
      // declarado método "canEdit()"
      alert(collection.models[0].canEdit());
    });
  }
]);
{% endhighlight %}

Dessa forma mapeamos a informação no nosso aplicativo em objetos, ajudando na manutenibilidade e concentração do boilerplate responsável por manipular esses objetos.

E você, qual a solução que você usa nos seus aplicativos Angular.js para manipular dados?
