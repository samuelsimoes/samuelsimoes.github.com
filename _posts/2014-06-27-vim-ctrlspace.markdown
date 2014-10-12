---
layout: post
title:  "vim-ctrlspace"
date:   2014-06-27 23:50
categories: vim workflow
icon: "icon-keyboard"
---

No meu dia a dia costumo lidar com features complexas que envolvem diversos arquivos; javascript, views, models, controllers entre outras coisas mais que até o momento cada arquivo ficava na sua própria aba no Vim, criando uma mar confuso de abas.

Para minha felicidade, recentemente conheci o plugin **[vim-ctrlspace](https://github.com/szw/vim-ctrlspace)**, que tem a proposta muito interesse de utilizar as abas do Vim como um "espaço" para agrupar buffers, possibilitando, por exemplo, criar abas que guardam os arquivos que você está editando por tipo, como citado anteriormente. Tudo muito parecido com o que temos hoje no Tmux com as "sessions".

O uso do plugin é bastante simples, tudo que você vai abrindo em uma aba vai "agregando" a um "pilha de buffers" que fica a disposição em seletor que é ativado com o atalho `Ctrl + Espaço` (usa-se `j` e `k` para navegar).

<div class="image-container">
  <img src="/images/vim-ctrlspace.png" class="full-image"/>
</div>

###Atalhos mais importantes

Os atalhos a seguir devem ser executados com o seletor de buffers do plugin aberto (`Ctrl + Espaço`).

* Fechar um buffer: `c`
* Renomear a aba: `=`
* Buscar um buffer: `/`
* Listar diretório do arquivo selecionado: `E`
* Transferir um buffer para a aba da esquerda: `{`
* Transferir um buffer para a aba da direita: `}`
* Pré-visualizar um buffer (mostrar o buffer sem sair do seletor): `Espaço`

Abaixo um vídeo do autor apresentando o plugin em detalhes:

<iframe width="640" class="video-iframe" height="360" src="//www.youtube.com/embed/U1hbGJm3J0g" frameborder="0" allowfullscreen></iframe>

Muito prático para organizar workflows que envolvem a edição de muitos arquivos ao mesmo tempo. :)
