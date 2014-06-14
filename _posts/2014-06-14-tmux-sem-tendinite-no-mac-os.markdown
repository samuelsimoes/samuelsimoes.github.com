---
layout: post
title:  "Tmux sem tendinite no Mac OS"
date:   2014-06-14 20:00
categories: aleatorio
icon: "icon-keyboard"
---

Fui um feliz usuário do **Tmux** por algo em torno de 8 meses, no meu dia a dia preciso lidar com vários projetos em um mesmo dia e o conceito de sessões do Tmux é matador para manter a organização do ambiente de trabalho no terminal, agrupando console, editor de código, possíveis conexões SSH e todo tipo de coisa relacionado a um projeto em um espaço. Eu realmente recomendo [você conhecer o Tmux](http://code.tutsplus.com/tutorials/intro-to-tmux--net-33889), mas não vou me ater a ele neste post.

Como toda aplicação no terminal os comandos são executados a partir do teclado, os key bindings, no caso do Tmux você vai precisar executar dois comandos para quase todas as ações que você queira realizar, exemplo: trocar entre janelas/terminais, primeiro **Ctrl + b** e depois **n** ou **b**, ir para o dashboard de sessões, **Ctrl + b** e depois **s** e por aí vai.

Agora imagine a repetição destes comandos milhares de vezes por dia em um ritmo de trabalho acelerado. Tenso, né? Isso me causou uma **[tendinite](http://pt.wikipedia.org/wiki/Tendinite) no punho esquerdo**, o justo punho que tinha que ficar executando o **Ctrl+b** incessantemente. Com isso me vi obrigado a abandonar o Tmux em favor do uso das abas normais do iTerm com o atalho menos estressante e padrão do Mac OS **⌘ + Shift + \[ e \]**, o que de fato reduziu o estresse no meu punho esquerdo, porém me deixou sem o controle de sessões. :/

Felizmente, pesquisando formas de usar o Cmd (⌘) do Mac em key bindings do Tmux, eu achei um aplicativo muito interessante para a execução de macros com certos atalhos ou outras formas de ativação (digitar um texto por exemplo), o **[Keyboard Maestro](http://www.keyboardmaestro.com)**.

Com o **Keyboard Maestro** a ideia foi escrever macros que são ativados por atalhos padronizados nas aplicações do Mac OS, como o **⌘ + Shift + \[** para aba anterior, por exemplo, que envia para o iTerm, quando em foco, um **Ctrl + b** e depois um **p**, veja a imagem  do macro:

<img src="/images/keyboard-maestro.png" class="full-image without-shadow"/>

O legal é que o Keyboard Maestro sobrescreve os atuais atalhos do aplicativo, o que foi particularmente útil em alguns macros que criei.

Dessa forma pude voltar a utilizar o Tmux no meu workflow sem problemas de estresse no punho. \o/

A parte ruim da brincadeira é que o Keyboard Maestro é pago, e bem pago, R$85. Ele tem um modo trial que ainda não encontrei quais são as limitações então você pode testar ele de boa.
