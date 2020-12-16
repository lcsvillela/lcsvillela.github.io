---
layout: post
title:  "Descobrindo quais MIBs um equipamento utiliza"
date:   2020-08-04 17:32:34 -0300
categories: snmp monitoramento
---

Existe um projeto chamado [Zabbix] que é extremamente útil quando temos uma rede de computadores bem grande para monitorar, com ele podemos saber a temperatura de todos os computadores da rede, velocidade de disco e até mesmo (se o sensor existir) a temperatura da sala.
Porém, com o avanço da Internet Das Coisas ([IOT]), passou a existir dispositivos onde a instalação do agente do Zabbix, que faz a coleta de informações de sistemas operacionais, se fizesse impossível por falta de capacidade de processamento, alguns exemplos: Câmeras, ar condicionados e termostatos. [Aqui] tem um ótimo texto sobre.

Mas além disso, existem vários inconvenientes, para podermos pegar as informações manualmente e registrar elas no nosso Zabbix, precisamos saber o que o equipamento responde quando enviamos uma requisição [SNMP], para isso utilizamos o comando [snmpwalk].

{% highlight bash %}
snmpwalk -c comunidade -v2c endereco-ip
{% endhighlight %}

Nesse exemplo, usamos a versão dois do protocolo, mas dê uma lida no manual do comando snmplwalk com o comando:

{% highlight bash %}
man snmpwalk
{% endhighlight %}

Após executamos esse comando, receberemos uma lista extensa de informações, onde temos o seguinte formato:

MIB::OID

Porém, a surpresa aqui é que quando realizamos a requisição com o comando snmpwalk, ele só vai exibir na tela as respostas para as MIBs que temos no nosso computador([veja sobre]), você pode inclusive pegar MIBs no site Best Monitoring Tools ou no Circitor e adicioná-las no seu computador.Contudo, como vou saber as MIBs do equipamento sendo que nem é exibido na tela com a requisição que fizemos acima? E se eu não quiser adicionar elas no meu sisetma? Para isso, temos um truque:

{% highlight bash %}
snmpwalk -c public -v2c 192.168.10.120 .1 | cut -d : -f 1 | uniq
{% endhighlight %}

Com este comando, fazemos uma requisição para o IP 192.168.10.120 para consultar todas as OIDs que começam com .1, ou seja, teremos tudo! Depois redirecionamos com o '|' essa saída de texto enorme para o comando cut, onde delimitamos com a opção '-d' os campos pelo caractere ':' e qual campo queremos com a opção '-f'. E finalmente, redirecionamos para o comando uniq, que elimina linhas iguais e por consequência, recebemos uma lista limpa de quais MIBs o quipamento responde!

Com isso, podemos saber quais as MIBs o equipamento responde e todos os OIDs que estão funcionando :D

[Zabbix]: https://www.zabbix.com/br/
[IOT]: https://pt.wikipedia.org/wiki/Internet_das_coisas
[Aqui]: https://errc.sbc.org.br/2020/papers/ST_IC2_2_SNMP_Industriais.pdf
[SNMP]: https://www.gta.ufrj.br/grad/10_1/snmp/snmp.htm
[snmpwalk]: http://www.net-snmp.org/docs/man/snmpwalk.html
[veja sobre]: https://meuladodigital.com.br/2020/04/23/carregando-novas-mibs-no-linux/
