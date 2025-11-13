---
layout: post
title:  "Laboratório de Flink 0 - Experimentos com streaming"
date:   2025-11-13 03:00:34 -0300
categories: flink big-data streaming
description: Laboratório de fluxo (streaming) de dados com Apache Flink - Parte 1
image: 
thumbnail: 
---

<h2>Fluxo de dados para quê?</h2>

A realidade muda a todo instante, assim como as nuvens, imaginar uma realidade imutável é algo impossível, por
isso para manter a estabilidade de um sistema qualquer é necessário entender como a realidade (do latim
_realis_, significa o que é verdadeiro ou real) opera, pois a tendência é que todo sistema seja instâvel.

Sendo assim, podemos imaginar um sistema IoT com centenas de dispositivos, talvez milhares, onde a sincronicidade
deve ser garantida para realizar alguma tarefa ou então, podemos pensar em monitoramento para auxiliar o
marketing em redes sociais, no caso [este trabalho que foi publicado em um artigo]{:target="\_blank"}
 fala exatamente sobre isso.

Mas como construir um sistema desse tipo? Pois é necessário analisar o fluxo de dados, não os dados em lote 
(quando já temos um conjunto de dados), isso nos dá uma perspectiva da realidade em tempo real, contribuindo para
práticas de OSINT diversas, por exemplo: Uma das táticas de publicidade é monitorar o seu sinal de celular dentro
de shoppings e assim saber quais vitrinas você observou, tendo esses dados em fluxo é possível saber o momento
exato que uma promoção deve ser anunciada no microfone, uma intervenção deve ser feita no trânsito da cidade,
controlar sinal de trânsito, aplicativos de corrida e afins.


[0 - Preparando o ambiente](#0) \\
[1 - Iniciando o Flink em modo Cluster](#1)\\
[2 - Exemplo do WordCount](#2)\\
[3 - Exemplo do SocketWindowWordCount](#3)\\
[4 - Python com o Flink](#4)\\
[5 - Algumas opções de linha de comando do Flink](#5)\\
[6 - Automatizando o processo completo do SocketWindowWordCount](#6)

<h2 id="0">0 - Preparando o ambiente</h2>

Vamos utilizar uma distribuição Linux, pode ser Ubuntu, mas o importante é que tenhamos acesso ao terminal,
podemos obter o [Flink neste link]{:target="\_blank"} e realizar seu download com o comando:
{% highlight None %}
wget https://dlcdn.apache.org/flink/flink-2.1.1/flink-2.1.1-bin-scala\_2.12.tgz
{% endhighlight %}

Depois disso devemos descompactar:
{% highlight None %}
tar xf flink-2.1.1-bin-scala\_2.12.tgz
{% endhighlight %}

Acessamos o seu diretório:
{% highlight None %}
cd flink-2.1.1
{% endhighlight %}

realizando o comando _ls_ você deve ver os diretórios disponíveis no projeto.

<h2 id="1">1 - Inici4ndo o Flink em m0do clust3r</h2>

Com o seguinte comando o cluster é iniciado:

{% highlight None %}
./flink-2.1.1/bin/start-cluster.sh
{% endhighlight %}

Pode-se verificar sua execução acessando [http://localhost:8081]{:target="\_blank"}, pois o Flink tem interface
web disponível para verificar informações sobre as tarefas executadas.


<h2 id="2">2 - Realizando o WordC0unt </h2>

Podemos gerar um arquivo com vários números com o seguinte Shell Script:

{% highlight bash %}
LINE=""
>numbers.txt
for j in {1..1000}
do
    for i in {1..100}
    do
        LINE+=" $RANDOM"
    done
    echo $LINE >> numbers.txt
done
{% endhighlight %}

A prática de contar palavras, nesse caso, podemos considerar que seja em lote, pois temos um arquivo
que não é dado como um fluxo de dados. Podemos usar o comando:

{% highlight None %}
./flink-2.1.1/bin/flink run /examples/batch/WordCount.jar --input numbers.txt --output numbers-wordcount.txt
{% endhighlight %}

<h2 id="3">3 - Exemplo do SocketWindowWordCount</h2>

Podemos criar um fluxo de dados para o Flink fazer uma análise, no caso temos o exemplo que usa uma
janela de tempo para realizar a análise, o que é interessante para analisar cenários mais dinâmicos.

Para isso precisamos abrir uma porta usando o netcat:
{% highlight None %}
nc -l 9000
{% endhighlight %}

Depois disso, podemos executar o Flink, que irá se conectar na porta que foi disponibilizada:

{% highlight None %}
./flink-2.1.1/bin/flink run ./flink-2.1.1/examples/streaming/SocketWindowWordCount.jar --port 9000
{% endhighlight %}

Volte no terminal que está com o netcat sendo executado, digite algumas mensagens ou palavras aleatórias, sempre
pressionando enter.

Depois de enviar sua mensagem, pressione CTRL+C para finalizar o netcat, volte no terminal do Flink e a tarefa
deve ter sido finalizada.

Sendo assim, haverá o registro de saída da contagem de palavras no fluxo de tempo, podemos verificar com:

{% highlight None %}
cat ./flink-2.1.1/log/*out

{% endhighlight %}


<h2 id="5" >Algumas opções da linha de comando do Flink</h2>

Podemos dar o comando:

{% highlight None %}
./bin/flink --help
{% endhighlight %}

E veremos dezenas de comandos interessantes da aplicação, a dica é ler tudo isso, mas destaco a opção -p, que
possibilita executar várias threads no mesmo computador, portanto poderíamos disponibilizar várias portas
com o netcat por exemplo e colocar o Flink para obter streaming de dados de várias ao mesmo tempo.

<h2 id="6"> Automatizando o processo completo do SocketWindowWordCount</h2>

Podemos criar um programa em bash para automatizar todo o processo que vimos no item sobre o
SocketWindowsWordCount:

{% highlight bash %}
#!/bin/bash

[ -e "flink-2.1.1-bin-scala_2.12.tgz" ] || wget "https://dlcdn.apache.org/flink/flink-2.1.1/flink-2.1.1-bin-scala_2.12.tgz" && tar xf "flink-2.1.1-bin-scala_2.12.tgz"

curl -s --max-time 5 http://localhost:8081/overview | grep '"taskmanagers"' || ./flink-2.1.1/bin/start-cluster.sh

>numbers.txt
for i in {1..100}
do
    LINE="$RANDOM"
    for i in {1..100}
    do
        LINE+=" $RANDOM"
    done
        echo $LINE >> numbers.txt
done


timeout 10 cat numbers.txt | nc -l 9999 &

sleep 4
timeout 30 ./flink-2.1.1/bin/flink run ./flink-2.1.1/examples/streaming/SocketWindowWordCount.jar --hostname 127.0.0.1 --port 9999 &
sleep 20


echo "Deseja parar o cluster flink? [S/N]"
read CHOICE

[ $CHOICE == "S" ] && ./flink-2.1.1/bin/stop-cluster.sh

cat ./flink-2.1.1/log/*out

{% endhighlight %}


<h3>Conclusão</h3>

Esse foi o primeiro texto sobre o Apache Flink, com uma ideia de proporcionar a capacidade de primeiro interagir
com o projeto e ter uma ideia de seu funcionamento.

[Flink neste link]: https://flink.apache.org/downloads/
[este trabalho que foi publicado em um artigo]: https://ieeexplore.ieee.org/abstract/document/10923360
[plugin vulnerável no wordpress]: https://nvd.nist.gov/vuln/detail/CVE-2020-35489
[Google Hacking]: https://pt.wikipedia.org/wiki/Google_Hacking
[Anderson Rocha]: https://youtu.be/GA-q2o-I0VY
[Open Source Intelligence]: https://www.youtube.com/watch?v=iAlwdOznL9E
[BeautifulSoap]: https://beautiful-soup-4.readthedocs.io/en/latest/
[Quotes to Scrape]: https://quotes.toscrape.com/
[primeiro]: https://lcsvillela.github.io/nutrindo-se-da-internet-com-scrapy.html
[Americana]: https://lcsvillela.github.io/querido-diario-monitorando-governo-com-scrapy.html
[Araraquara]: https://lcsvillela.com/querido-diario-monitorando-governo-araraquara-com-scrapy.html
[este caso que criei]: https://lcsvillela.github.io/publicando-tweet-com-python.html
[JSON]: https://pt.wikipedia.org/wiki/JSON
[Scrapy]: https://pt.wikipedia.org/wiki/Scrapy
[aqui]: https://lcsvillela.github.io/nutrindo-se-da-internet-com-scrapy.html
[spider (aranha)]: https://pt.wikipedia.org/wiki/Rastreador_web
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
[melhores placas de GPU]: https://benchmarks.ul.com/pt-br/compare/best-gpus
[Slurm]: https://en.wikipedia.org/wiki/Slurm_Workload_Manager
[PCAD]: http://gppd-hpc.inf.ufrgs.br/
[INF/UFRGS]: https://www.inf.ufrgs.br/
[HPC UFRGS]: https://github.com/lcsvillela/acoustic_waves_opencl_and_cuda/tree/master
