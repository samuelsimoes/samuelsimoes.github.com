---
layout: post
title:  "Resolvendo problemas comuns com PostGIS"
date:   2014-09-15 9:00
categories: postgres
icon: "icon-globe"
---

A algum tempo venho realizando operações com dados espaciais e para essa função estou utilizando o **[PostGIS](http://postgis.net/)**. Para quem não conhece o PostGIS é uma extensão para banco de dados **[PostgreSQL](http://www.postgresql.org/)** que adiciona toda essa capacidade espacial ao PG. Abaixo listo a solução de problemas comuns de geolocalização usando o PostGIS.

Obs.: nos exemplos estou usando uma coluna do tipo `geography` com SRID 4326, caso você tenha apenas latitude e longitude no seu banco de dados você vai precisar gerar a geography no runtime da query utilizando algo do tipo:

`ST_GeographyFromText('SRID=4326;POINT(' || longitude || ' ' || latitude || ')')`

## Pontos próximos com limites
> Preciso dos usuários próximos do usuário logado no raio de 100 metros.

Usando a função **[ST_DWithin](http://postgis.refractions.net/docs/ST_Within.html)** conseguimos realizar esta query utilizando o raio da distância, veja:

{% highlight sql %}
SELECT
  users.*
FROM
  users
WHERE
  ST_DWithin(
    Geography(users.geography),
    Geography(ST_GeographyFromText('SRID=4326;POINT(LONGITUDE_REFERENCIA LATITUDE_REFERENCIA)')),
    DISTANCIA_EM_METROS_DO_RAIO_PERMITIDO
  );
{% endhighlight %}

## Ordernar por proximidade
> Quero que o meu usuário logado veja a lista de amigos ordenada pela proximidade entre eles.

Usando a função **[ST_Distance](http://postgis.refractions.net/docs/ST_Distance.html)** conseguimos calcular a distância entre os pontos e com esse valor conseguimos ordenar a nossa query, veja:

{% highlight sql %}
SELECT
  users.*
FROM
  users
ORDER BY
  ST_Distance(
    Geography(users.geography),
    Geography(ST_GeographyFromText('SRID=4326;POINT(LONGITUDE_REFERENCIA LATITUDE_REFERENCIA)')),
  ) ASC
{% endhighlight %}

## Seleção respeitando limites
> Quero carregar apenas os pontos na área visível do meu navegador de mapas na interface.

Utilizando o operador **[&&](http://postgis.net/docs/geometry_overlaps.html)** e a função **[ST_MakeEnvelope](http://postgis.net/docs/ST_MakeEnvelope.html)** é bastante simples realizar esta query:

{% highlight sql %}
SELECT
  users.*
FROM
  users
WHERE
  users.geography &&
    ST_MakeEnvelope(LEFT_COORD, LEFT_COORD, RIGHT_COORD, TOP_COORD);
{% endhighlight %}

### Bônus Google Maps API v3
Se você usa a API de mapas do Google Maps você pode pegar as dimensões da área visualizada utilizando as seguintes propriedades da instância do seu mapa:


{% highlight javascript %}
var map = new google.maps.Map(document.getElementById("map")),
    mapBounds = map.getBounds();

var mapCornersCoords = {
  top_coord: mapBounds.getNorthEast().lat(),
  right_coord: mapBounds.getNorthEast().lng(),
  bottom: mapBounds.getSouthWest().lat(),
  left: mapBounds.getSouthWest().lng()
};
{% endhighlight %}

## Geo clusters (agrupar pontos)
É impraticável apresentar mapas com uma massa de dados muito densa pela quantidade de marcadores na tela.

<div class="image-container">
  <img src="/images/too-many-dots.png" class="image-with-shadow"/>
  <p class="legend">Ficou oh...</p>
</div>

Para sanar esse problema podemos criar geo clusters usando a função **[ST_SnapToGrid](http://postgis.net/docs/ST_SnapToGrid.html)**. Basicamente o que ela faz é agrupar um conjunto de pontos em células de uma grade, onde podemos definir o tamanho dessas células, e dentro de cada uma, onde serão as coordenadas retornada para aquela célula em questão.

`ST_SnapToGrid(geography, x start, y start, width, heigth)`

As medidas utilizadas na função ficam em torno da razão de `tamanhoEmMetro/100000` pelos experimentos que fiz, infelizmente a documentação não entra nesse mérito e esse valor pode não ser preciso em toda a área do mapa.

Uma query consultando clusters com a contagem de itens agrupados fica assim:

{% highlight sql %}
SELECT
  COUNT(users.id) AS count,
  ST_X(ST_SnapToGrid(users.geography, GRID_SIZE)) AS longitude,
  ST_Y(ST_SnapToGrid(users.geography, GRID_SIZE)) AS latitude
FROM
  users
GROUP BY
  ST_SnapToGrid(users.geography, GRID_SIZE);
{% endhighlight %}

Para melhorar um pouco o posicionamento da referência do cluster no mapa nós podemos utilizar duas outras funções, a **[ST_Collect](http://postgis.net/docs/ST_Collect.html)** e a **[ST_Centroid](http://postgis.net/docs/ST_Centroid.html)**. A primeira vai criar uma forma geográfica com os pontos dentro da célula e a segunda vai retornar o centro da forma geográfica gerada, com isso vamos conseguir aproximar melhor o marcador da referência do cluster baseado na localização das pessoas dentro da célula. Veja a imagem para enteder melhor:

<div class="image-container">
  <img src="/images/postgis-grid-center.png"/>
  <p class="legend">O quadrado cinza é resultado da ST_Collect e o centro do quadrado é o resultado do ST_Centroid.</p>
</div>

{% highlight sql %}
SELECT
  COUNT(users.id) AS count,
  ST_X(
    ST_Centroid(ST_Collect(users.geography))
  ) AS longitude,
  ST_Y(
    ST_Centroid(ST_Collect(users.geography))
  ) AS latitude
FROM
  users
GROUP BY
  ST_SnapToGrid(users.geography, GRID_SIZE);
{% endhighlight %}

Infelizmente o que não vamos conseguir fazer (pelo menos eu ainda não descobri como) é posicionar o ponteiro para ficar mais próximo de onde tiver mais pontos.

## Concluindo
O PostGIS é uma ferramenta fantástica para manipulação de informação espacial, porém, IMHO, sua documentação deixa muito a desejar na clareza das explicações e detalhes.

Acredito que os exemplos acima sejam os casos mais comuns de uso, porém a ferramenta oferece várias outras possibilidades interessantes que valem a pena você pesquisar sobre.

Se você usa o ActiveRecord do Ruby on Rails eu recomendo este artigo mostrando como "amarrar" tudo isso: **[PostGIS and Rails: A Simple Approach](http://ngauthier.com/2013/08/postgis-and-rails-a-simple-approach.html)**

Qualquer dúvida ou sugestão deixe nos comentários.
