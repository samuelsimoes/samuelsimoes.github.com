---
layout: post
title:  "CSS Clearfix"
date:   2013-05-12 22:00
categories: html-css
icon: "icon-cloud"
---

Esse post vai para a galera que está começando com CSS e HTML e certamente passa/passou por esse problema.

Antes de qualquer coisa um **clearfix** é o termo usado para definir alguma forma de corrigir o problema que é gerado quando nós temos um container que contém outros containers que flutuam, calma, eu explico. :)

É comum passarmos pela situação onde usamos o **float** do CSS para flutuar blocos e não incomummente o elemento pai, que estava contendo os nossos blocos flutuantes, fica *"vazio"* ou *"some"*, abaixo um exemplo disso:

{% highlight html linenos %}
<div>
    <div style="float: left; width: 40%;">
        <p>Esquerda</p>
    </div>
    <div style="float: right; width: 40%;">
        <p>Direita</p>
    </div>
</div>
{% endhighlight %}

<div class="example-container">
    <div class="float-container">
        <div class="float-box" style="float: left; width: 40%;">
            <p>Esquerda</p>
        </div>
        <div class="float-box" style="float: right; width: 40%;">
            <p>Direita</p>
        </div>
    </div>
</div>

Usando o pseudo elemento **:after** do CSS podemos simular a existência de um bloco no final da DIV que contém os blocos, formando assim uma *âncora* e fazendo a mesma *"considerar a existência"* dos blocos internos.

{% highlight css %}
.clearfix:after {
    content: '';
    display: block;
    clear: both;
}
{% endhighlight %}

Usando a classe **.clearfix** do CSS criado acima na DIV que engloba os elementos *(linha 1 do HTML acima)* tudo funciona bem!

<div class="example-container">
    <div class="float-container clearfix">
        <div class="float-box" style="float: left; width: 40%;">
            <p>Esquerda</p>
        </div>
        <div class="float-box" style="float: right; width: 40%;">
            <p>Direita</p>
        </div>
    </div>
</div>

Abaixo um exemplo real, de um menu de navegação de um site qualquer **com o clearfix**.

<style>
    .menu {
        background: #0d8797;
        padding: 10px;
        margin-bottom: 20px;
    }

    .menu a {
        border: 2px solid #137683;
        color: #000;
        float: left;
        display: block;
        padding: 4px 10px;
        background: #fff;
        margin-right: 10px;
    }
</style>

<div class="example-container">
    <div class="menu clearfix">
        <a href="#">Home</a>
        <a href="#">Pedidos</a>
        <a href="#">Contato</a>
    </div>
</div>

O código desse menu:

{% highlight html %}
<div class="menu clearfix">
    <a href="#">Home</a>
    <a href="#">Pedidos</a>
    <a href="#">Contato</a>
</div>
{% endhighlight %}

{% highlight css %}
.menu {
    background: #0d8797;
    padding: 10px;
    margin-bottom: 20px;
}

.menu a {
    border: 2px solid #137683;
    color: #000;
    float: left;
    display: block;
    padding: 4px 10px;
    background: #fff;
    margin-right: 10px;
}
{% endhighlight %}

Agora se tirarmos o **clearfix** olha como fica:

<div class="example-container">
    <div class="menu">
        <a href="#">Home</a>
        <a href="#">Pedidos</a>
        <a href="#">Contato</a>
    </div>
</div>

É isso, não se estresse com o float do CSS, é só uma questão de entender o seu funcionamento e usar corretamente o clearfix.

Qualquer dúvida é só deixar nos comentários. =D
