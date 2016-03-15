---
layout: post
title:  "vim-ctrlspace"
date:   2014-06-27 23:50
categories: vim workflow
icon: "icon-keyboard"
---

No meu dia a dia costumo lidar com funcionalidades que envolvem diversos arquivos de diferentes propósitos como: javascripts, views, models, controllers entre outros que até o momento cada arquivo ficava na sua própria aba no **Vim**, criando uma mar confuso de abas.

Para tentar sanar esse problema recentemente encontrei o plugin **[vim-ctrlspace](https://github.com/szw/vim-ctrlspace)**, que tem a proposta muito interesse de utilizar as abas do Vim como um "grupo de buffers" (nativamente as abas e splits acessam o stack de buffers global), possibilitando, por exemplo, criar abas que agrupem os arquivos que você está editando por tipo.

O uso do plugin é bastante simples. Depois de instalado tudo que você vai abrindo em uma aba entra para o stack de buffers da mesma, que fica a disposição em seletor de buffers ativado através do atalho `Ctrl + Espaço` (usa-se `k` e `j` para navegar para cima e para baixo respectivamente).

<div class="image-container">
  <img src="/images/vim-ctrlspace.png" class="full-image"/>
</div>

## Atalhos mais importantes

Os atalhos a seguir devem ser executados com o seletor de buffers do plugin aberto (`Ctrl + Espaço`).

* Fechar um buffer: `c`
* Renomear a aba: `=`
* Buscar um buffer: `/`
* Listar diretório do arquivo selecionado: `E`
* Transferir um buffer para a aba da esquerda: `{`
* Transferir um buffer para a aba da direita: `}`
* Visualizar todos os commands: `?`
* Pré-visualizar um buffer (mostrar o buffer sem sair do seletor): `Espaço`

Abaixo um vídeo do autor apresentando o plugin em detalhes:

<iframe width="640" class="video-iframe" height="360" src="//www.youtube.com/embed/U1hbGJm3J0g" frameborder="0" allowfullscreen></iframe>

Muito prático para organizar workflows que envolvem a edição de muitos arquivos que podem ser agrupados.
