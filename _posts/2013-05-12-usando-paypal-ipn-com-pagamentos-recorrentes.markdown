---
layout: post
title:  "Usando PayPal IPN com Pagamentos recorrentes"
date:   2013-05-12 14:00
categories: programacao
icon: "icon-money"
---

O PayPal IPN, para os que não sabem, é o sistema de notificação instantânea que o PayPal oferece cujo o objetivo é retornar em uma URL do seu aplicativo um request POST contendo informações referentes a alguma ação ocorrida em alguma das suas transações. Exemplo: um usuário cancela um pagamento recorrente, logo após o usuário tomar essa ação, na URL configurada anteriormente, você irá receber o POST com as informações da ação.

Não vou entrar no mérito de explicar todo o funcionamento dos pagamentos recorrentes, mas caso você esteja usando Ruby você pode usar a ótima **[paypal-recurring gem](https://github.com/fnando/paypal-recurring)** e recomendo também você ver a [palestra do Nando Vieira sobre pagamentos recorrentes com PayPal e a paypal-recurring gem na Guru-SP](http://blip.tv/agaelebe/gurusp_encontro_17_fnando_paypal-5517398) e o episódio [#289 do Rails Casts (PayPal Recurring Billing)](http://railscasts.com/episodes/289-paypal-recurring-billing).

Para você conseguir testar a recepção de notificações no seu ambiente de desenvolvimento, que provavelmente está rodando localmente, você deve expor o acesso ao seu app para que ele fique acessível ao PayPal, para essa tarefa eu recomendo o uso do **[localtunnel](http://progrium.com/localtunnel)**.

Para pagamentos do tipo *express checkout* você pode definir na criação da transação qual vai ser a URL que vai receber as IPNs para aquele pedido, mas infelizmente essa definição dinâmica parece não funcionar para pagamentos recorrentes.

{% highlight ruby %}
ppr = PayPal::Recurring.new({
  # Não funciona para pagamentos recorrentes :(
  :ipn_url	  => "http://example.com/paypal/ipn"
})

response = ppr.checkout
puts response.checkout_url if response.valid?
{% endhighlight %}

**A solução é você definir qual vai ser a URL que vai receber as IPNs no site do PayPal (tanto produção quanto sandbox)**, no caso do cenário de desenvolvimento acesse a PayPal Developer Network e na conta BUSINESS clique em *sandbox site*.

<div class="image-container">
  <img src="/images/sandbox-link-dashboard.jpeg" class="image-with-shadow full-image"/>
</div>

Depois acesse: **My Account > Profile > My Selling Tools**.

Infelizmente o sandbox do PayPal é bastante lento e problemático, ao clicar no link deveria, no corpo da página, aparecer as devidas configurações, mas isso não acontece, o site fica carrega ad eternum.

<div class="image-container">
  <img src="/images/sandbox-link-copy.jpeg" class="image-with-shadow full-image"/>
</div>

**A solução nesse caso é copiar o link que fica na sidebar esquerda** (imagem acima), retirar o **beta-** da URL e acessar normalmente, exemplo:

``https://www.beta-sandbox.paypal.com/cgi-bin/webscr?cmd=_profile-display-handler&tab_id=SELLER_PREFERENCES```

vira:

```https://www.sandbox.paypal.com/cgi-bin/webscr?cmd=_profile-display-handler&tab_id=SELLER_PREFERENCES```

Feito isso você provavelmente estará tendo acesso a página de configurações de vendedor, nessa página você deve acessar o item **Instant payment notifications** ou algo em torno disso. O link provavelmente irá te levar para um erro 404, faça o processo de "corrigir" a URL novamente.

Na página de configuração defina a **Notification URL** apontando para uma URL pública do seu app que leve para onde você quer que suas IPNs sejam devidamente capturadas.

Um exemplo de action para capturar as notificações usando a própria classe de notificações não documentado da paypal-recurring gem seria mais ou menos isso:

**```routes.rb```**
{% highlight ruby %}
App::Application.routes.draw do
  post 'paypal/ipn_listener' => 'paypal#ipn_listener'
end
{% endhighlight %}

**```paypal_controller.rb```**
{% highlight ruby %}
class PaypalController < ApplicationController
  protect_from_forgery :except => [:ipn_listener]

  def ipn_listener
    notification = PayPal::Recurring::Notification.new(params)

    # Se é uma notificação de pagamento recorrente
    notification.recurring_payment?

    # Se é uma notificação sobre a criação do profile de recorrência
    notification.recurring_payment_profile?

    # ID do profile
    notification.payment_id

    # Status do Profile
    notification.profile_status

    # Referência previamente definida por você
    notification.reference

    render nothing: true
  end
end
{% endhighlight %}

[Dê uma lida nessa classe](https://github.com/fnando/paypal-recurring/blob/master/lib/paypal/recurring/notification.rb), ela é super simples e bem esclarecedora.

Dúvidas? Deixe nos comentários. =D
