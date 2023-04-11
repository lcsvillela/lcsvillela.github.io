---
layout: post
title:  "Descobrindo computadores na rede"
date:   2011-01-28 10:00:34 -0300
categories: Linux network 
description: Como descobrir computadores na rede? 
image: https://user-images.githubusercontent.com/23728459/231237583-7d5cacf2-3088-4280-93a6-222cc8468d2c.png
thumbnail: https://user-images.githubusercontent.com/23728459/231237583-7d5cacf2-3088-4280-93a6-222cc8468d2c.png
---

Quando estamos numa rede, algo muito útil em diversas situações é ver quais computadores existem nesta rede. Há vários maneiras de se fazer isso. Hoje irei ensinar como fazer isso usando o netdiscover, que é bem simples.

O netdiscover faz um scan na rede usando o protocolo arp, ou seja, você não precisa ter um ip na rede. Por exemplo, quando você conecta o seu sistema GNU/Linux em uma rede windows, ele não ira adquirir um ip, mas será possivel ver as máquinas do mesmo jeito, é possível ver o ip das máquinas e o Mac Adress (: 

| Diretiva      | Função | Exemplo |
| ----------- | ----------- | ----------- |
| -i |Especifica qual a interface de rede será escaneada |netdiscover -i eth1|
| -r |Define qual a faixa de endereços ip você quer escanear. Por padrão, se nada for especificado, ele usa o IP da placa em questão|	netdiscover -r 192.168.0.0/16|
| -l |Manda o programa ler o arquivo que contém várias faixas de endereços IP, uma por linha| 	netdiscover -l arquivo|
| -p |Usando esse modo, você fica invisível na rede, já que o netdiscover vai apenas capturar os pacotes para ver as maquinas na rede. Demora um pouco mais, porém não envia requisições arp. Desta forma, você não será descoberto na rede facilmente|netdiscover -p -i eth1|
|s|Manda as requisições de segundos em segundos, também é ideal para manter seu anonimato na rede|netdiscover -s 5 -i eth1|
| -c |Define a quantidade de pacotes a serem enviados para a máquina alvo, excelente para redes lentas ou com perdas de pacotes| netdiscover -s 5 -c 1 -i eth1|
| -S |Ativa o tempo de espera entre cada requisição. Irá utilizar toda a capacidade da rede, muito bom para redes wireless que tem grande perda de pacotes| 	netdiscover -S -i eth1|
|-f| 	Faz um scan rápido, útil para descobrir as máscaras de rede utilizadas 	|netdiscover -f -i eth1
|-d| 	Ignora os arquivos de configuração localizados no /home do usuário e ativa as configurações padrão| 	netdiscover -d -f -i eth1

<h2>Arquivos de Configuração</h2>

 O netdiscover conta com arquivos de configuração para facilitar seu uso:

{% highlight bash %}
  /home/user/.netdiscover/ranges
{% endhighlight %}


exemplo:
{% highlight bash %}
  192.168.0.0/8
  10.0.0.0/16
  192.168.15.0/24
{% endhighlight %}

Esse arquivo guarda as faixas padrão de endereços IP (-r)

{% highlight bash %}
  /home/user/.netdiscover/fastips
{% endhighlight %}

Esse arquivo guarda as configurações padrões para o scan rápido (-f)

exemplo:
{% highlight bash %}
  2
  5
  10
  100
  150
  200
  250
{% endhighlight %}


Originalmente publicado em [descobrindo computadores na rede]{:target="\_blank"} 

[descobrindo computadores na rede]: https://www.dicas-l.com.br/arquivo/descobrindo_computadores_da_rede.php#.ZDWNX-vMLIV
[BeautifulSoap]: https://beautiful-soup-4.readthedocs.io/en/latest/
[Quotes to Scrape]: https://quotes.toscrape.com/
[primeiro]: https://lcsvillela.github.io/nutrindo-se-da-internet-com-scrapy.html
[segundo]: https://lcsvillela.github.io/querido-diario-monitorando-governo-com-scrapy.html
[este caso que criei]: https://lcsvillela.github.io/publicando-tweet-com-python.html
[JSON]: https://pt.wikipedia.org/wiki/JSON
[Scrapy]: https://pt.wikipedia.org/wiki/Scrapy
[aqui]: https://lcsvillela.github.io/nutrindo-se-da-internet-com-scrapy.html
[spider (aranha)]: https://pt.wikipedia.org/wiki/Rastreador_web
[Americana]: https://pt.wikipedia.org/wiki/Americana
[Querido Diário]: https://queridodiario.ok.org.br/
[Open Knowledge Brasil]: https://ok.org.br/
[nesta colaboração]: https://github.com/okfn-brasil/querido-diario/issues/467
[assim ou assado, ou se é x ou y]: https://www.python.org/community/diversity/
[BeautifulSoap]: https://beautiful-soup-4.readthedocs.io/en/latest/
[Selenium]: https://selenium-python.readthedocs.io/
[vim]: https://pt.wikipedia.org/wiki/Vim
[URL]: https://pt.wikipedia.org/wiki/URL
[scrapy-splash]: https://github.com/scrapy-plugins/scrapy-splash
[processamento paralelo]: https://lcsvillela.github.io/bash-na-velocidade-da-luz.html
[explicação teórica]: https://towardsdatascience.com/web-scraping-with-scrapy-theoretical-understanding-f8639a25d9cd
[xpath]: https://pt.wikipedia.org/wiki/XPath
[vetor (array)]: https://pt.wikipedia.org/wiki/Arranjo_(computa%C3%A7%C3%A3o)
[oauth2]: https://oauth.net/2/
