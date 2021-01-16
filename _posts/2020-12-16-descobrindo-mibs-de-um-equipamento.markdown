---
layout: post
title:  "Descobrindo quais MIBs um equipamento utiliza"
date:   2020-12-16 17:32:34 -0300
categories: snmp monitoramento
---

Existe um projeto chamado [Zabbix], que é extremamente útil quando temos uma rede de computadores bem grande para monitorar. Com ele podemos saber a temperatura de todos os computadores da rede, velocidade de disco e até mesmo (se o sensor existir) a temperatura da sala.

Porém, com o avanço da Internet Das Coisas ([IOT]), passaram a existir dispositivos nos quais a instalação do agente do Zabbix - que faz a coleta de informações de sistemas operacionais -, se fizesse impossível, por falta de capacidade de processamento. Alguns exemplos: Câmeras, ares-condicionados e termostatos. [Aqui] há um ótimo texto sobre.

Além desse, existem vários inconvenientes. Para coletarmos as informações manualmente e registrá-las no nosso Zabbix, precisamos saber o que o equipamento responde quando enviamos uma requisição [SNMP] - requisição essa feita com o comando [snmpwalk].

{% highlight bash %}
snmpwalk -c comunidade -v2c endereco-ip
{% endhighlight %}

Nesse exemplo, usamos a versão dois do protocolo, mas dê uma lida no manual do **snmplwalk** com o comando:

{% highlight bash %}
man snmpwalk
{% endhighlight %}

Após o executarmos, receberemos uma lista extensa de informações, onde há o seguinte formato:

MIB::OID

A surpresa aqui é que, quando realizamos a requisição com o comando **snmpwalk**, ele só vai exibir na tela as respostas para as MIBs que temos em nosso computador([veja sobre]).

Podemos baixar MIBs no site [Best Monitoring Tools] ou no [Circitor] e adicioná-las ao computador, mas como saberemos as MIBs do equipamento, uma vez que elas não são exibidas na tela com a requisição feita acima? E se não quisermos adicioná-las ao sistema?

Para isso, há um truque:

{% highlight bash %}
snmpwalk -c public -v2c 192.168.10.120 .1 | cut -d : -f 1 | uniq
{% endhighlight %}

Com este comando, é feita uma requisição para que o IP 192.168.10.120 consulte todas as OIDs que começam com .1. Ou seja, teremos tudo!

Depois, redireciona-se com o **|** essa saída de texto enorme para o comando **cut**, onde delimita-se com a opção **-d** os campos pelo caractere **:**, e qual campo se quer com a opção **-f**. E, finalmente, redireciona-se para o comando **uniq**, que elimina linhas iguais, resultando numa lista limpa de a quais MIBs o equipamento responde.

Com isso, chegamos ao nosso objetivo, sabendo também todos os OIDs que estão funcionando :D

[Zabbix]: https://www.zabbix.com/br/
[IOT]: https://pt.wikipedia.org/wiki/Internet_das_coisas
[Aqui]: https://errc.sbc.org.br/2020/papers/ST_IC2_2_SNMP_Industriais.pdf
[SNMP]: https://www.gta.ufrj.br/grad/10_1/snmp/snmp.htm
[snmpwalk]: http://www.net-snmp.org/docs/man/snmpwalk.html
[veja sobre]: https://meuladodigital.com.br/2020/04/23/carregando-novas-mibs-no-linux/
[Best Monitoring Tools]: https://bestmonitoringtools.com/
[Circitor]: http://www.circitor.fr/Mibs/Mibs.php
