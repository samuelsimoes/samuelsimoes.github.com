---
layout: post
title:  "Um pouco sobre as diretrizes do Angular.js"
date:   2014-07-02 20:10
categories: angular javascript
icon: "icon-cog"
---

Uma das funcionalidades mais legais do Angular.js é a sua capacidade de definir comportamentos em um nó do DOM a partir de um módulo que é totalmente integrado ao resto da sua aplicação, o que o Angular chama de "diretriz" (directives).

Um exemplo dentro de tantas possibilidades seria: imagine que você quer desabilitar um botão de submissão de request (um submit de um formulário, por exemplo) enquanto ele está em andamento para evitar que o usuário clique no botão várias vezes e tenha um feedback do processo. De forma muito fácil (se você já entende a magia por traz do `$scope`) podemos criar uma diretriz que vai servir para todos os casos similares no nosso aplicativo.

{% highlight javascript linenos %}
angular.module("DisableOnSendDirective", []).directive("disableOnSend", function () {
  return {
    link: function ($scope, $el, attrs) {
      $scope.$watch("submitPromise", function (newData) {
        if (!newData) { return; }

        var previousText = $el.html();

        $el.attr("disabled", "disabled");
        $el.html(attrs.disableOnSend);

        $scope.submitPromise["finally"](function () {
          $el.removeAttr("disabled");
          $el.html(previousText);
        });
      });

      return $el;
    }
  };
});
{% endhighlight %}

Na nossa diretriz vamos definir que o `$scope` deve observar o atributo `submitPromise`, pois é nesse atributo que vamos definir a **[promise](https://docs.angularjs.org/api/ng/service/$q)** que vai guiar quando o nosso botão vai ser habilitado ou desabilitado (linha 4).

Linha 7: fazemos um cache do atual texto do botão.

Linha 9-10: desabilitamos de fato o botão e trocamos o texto para o que for definido na chamada da diretriz lá no DOM (snippet abaixo). Nesse momento temos acesso ao objeto `$el` que pode ser uma instância do [jqLite](https://docs.angularjs.org/api/ng/function/angular.element) caso você não tenha implementado o jQuery no seu projeto, se tiver, o Angular.js passa essa tarefa para o jQuery.

Linha 12-15: ao fim do request, anexando um callback no `finally` (que vai ser ativado mesmo se o request falhar), vamos reverter tudo, habilitando novamente o botão e trazendo o antigo texto dele.

Abaixo como fica o nosso HTML:

{% highlight html %}
<div ng-controller="MyController">
  <button disable-on-send="Enviando, aguarde..." ng-click="makeRequest()">
    Make a request!
  </button>
</div>
{% endhighlight %}

E no nosso controller a declaração da promise que vai guiar o comportamento do botão de submissão do nosso request.

{% highlight javascript %}
angular.module("MyApp", ["DisableOnSendDirective"]).controller("MyController", [
  "$scope", "$http",
  function ($scope, $http) {
    $scope.makeRequest = function () {
      $scope.submitPromise = $http({ url: "my_endpoint", method: "POST" });
    };
  }
]);
{% endhighlight %}

Este foi um exemplo bastante simples em cima das diretrizes e pretendo postar outros que uso nos meus aplicativos Angular, mas com isso já podemos ver como fica fácil modularizar comportamentos que podem ser reutilizado não só no aplicativo, mas em diversos outros projetos.

Não deixe de ler a [documentação oficial](https://docs.angularjs.org/guide/directive) para ver todas as possibilidades das diretrizes, vale a pena.
