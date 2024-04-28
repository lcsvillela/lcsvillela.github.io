---
layout: post
title:  "Gerando redes aleatórias no iSPD e automatizando"
date:   2024-04-28 00:00:34 -0300
categories: bash shell java simulador ispd
description: gerando redes aleatórias no iSPD
image: https://github.com/lcsvillela/lcsvillela.github.io/assets/23728459/b1be66c1-f621-4a5e-a8b5-2a0333eafb99 
thumbnail: https://github.com/lcsvillela/lcsvillela.github.io/assets/23728459/b1be66c1-f621-4a5e-a8b5-2a0333eafb99
---

<h2>Um pouco sobre</h2>

Existem problemas que não podemos estudar por conta do seu tamanho, imagine por 
exemplo analisar como um material para construir prédio deve ser feito, com qual
liga metálica e em qual quantidade. Deveríamos construir o prédio para ver se
dará certo? Óbvio que não, é realizado um estudo para verificar como será feito,
nesse caso entram os engenheiros civis.

Mas no caso de sistemas computacionais distribuídos, precisamos de computadores,
muitas vezes cada computador custa no Brasil o valor de uma casa ou um carro,
pois não estamos falando de notebooks do varejo. São máquinas que ficam
empilhadas em armários próprios, em salas com resfriamento, muitas vezes
centenas, milhares ou até dezenas de milhares de máquinas.

Você pode se perguntar, para que tudo isso? Bom, temos problemas muito grandes
para resolver, só a infraestrutura do facebook daria para ser uma cidade
pequena para conseguir armazenar todos os dados. Mas existem lugares de quase
30km² só de computadores armazenando e processando informações a todo instante e
nesse caso temos um problema: Tenho N tarefas para uma quantidade de
computadores que podem ser até o mesmo modelo, mas são únicos e desta forma,
sofrem a ação do tempo de forma única, além disso podemos com o tempo, adquirir
mais máquinas em nosso datacenter e que obviamente terão uma capacidade de
processamento diferente. Nesse sentido, temos uma pergunta inquietante:

Como distribuir as tarefas para esses dispositivos da melhor forma possível? Mas
este ainda não será nosso tema! O que quero relatar aqui, é um pedaço do 
caminho, algo que contribui para que tenhamos um ambiente de estudo
automatizado.

![Splash](https://github.com/lcsvillela/lcsvillela.github.io/assets/23728459/1cb2b315-71c7-4586-b815-5b4d051279d4){: style="float: left; margin-right: 1em;" class="img-responsive" height="40%" width="40%"}
Para conseguir responder essa pergunta, precisava primeiro ter o ambiente criado
no simulador e para isso o [iSPD]{:target="\_blank"}
tem um ambiente gráfico, o que permite que
fiquemos focados na resolução do problema, sem precisar pensar sobre a questão
do ambiente em si. Os outros simuladores, por exemplo o SimGrid, CloudSim,
GridSim e outros, precisam ter as características do ambiente realizadas de
forma programática, o que dificulta substancialmente o processo de estudo.

Porém, no caso analisado, era necessário criar ambientes com 10, 50 e 100
máquinas (o que daria em um ambiente de 100 máquinas e muitas horas clicando,
arrastando e digitando configurações),
com capacidades de comunicação e processamento diferentes e fazer isso
com o mouse iria ser algo aterrador, além de que não teria como garantir que o
ambiente fosse significativamente diferente entre si. Nesse caso, o iSPD não
poderia me ajudar, então resolvi criar uma rede com 3 nós, colocando um deles
como principal e salvar o arquivo de configuração.

O iSPD gera um arquivo XML de configuração da rede e analisando o arquivo, criei
um Bash Script que constrói um arquivo de configuração para o iSPD e tudo o que
preciso fazer, é passar a quantidade de máquinas que quero.

Abaixo você pode conferir o Bash Script:

{% highlight bash %}

HEAD='<?xml version="1.0" encoding="ISO-8859-1" standalone="no"?><!DOCTYPE system SYSTEM "iSPD.dtd"><system version="2.1">    <owner id="user1" powerlimit="100.0"/>'

END='</system>'

BANDA=0
ID_LINK=0
LATENCIA=0
LOAD=0
DESTINATION=0
ORIGINATION=0
X=0
Y=0
X2=0
Y2=0
ID_GLOBAL=0
ID_LOCAL=0

ENERGY=$(echo $[ $RANDOM % 100 + 30 ])
ID_MAC=0
LOAD_MAC=0 #`bc <<< "scale=2;$(echo $[ $RANDOM % 1000 + 1 ])/1000"`
USER_MAC='user1'
POWER=$(echo $[ $RANDOM % 10 + 3 ])
X_MAC=0
Y_MAC=0
MASTER="<master scheduler=\"---\"/>"
ID_MASTER=0
HARD_DISK=$(echo $[ $RANDOM % 10000000 + 100000 ])
PROCESS=$(echo $[ $RANDOM % 7 + 3 ])
MEMORY=$(echo $[ $RANDOM % 100000 + 30000 ])

ENERGY=$(echo $[ $RANDOM % 100 + 30 ])
ID_MAC=0 #$(($ID_GLOBAL+1))
LOAD_MAC=0 #`bc <<< "scale=2;1*0.$(echo $[ $RANDOM % 10 + 9 ])"`
POWER=$(echo $[ $RANDOM % 132250+ 10000 ])

STRING=""

for i in $(eval echo {1..$(($1*2))..2})
        do
        STRING=$(echo "$STRING <slave id=\"$i\"/>")
done


MACHINE="<machine energy=\"100.0\" id=\"mac$ID_MAC\" load=\"0\" owner=\"$USER_MAC\" power=\"661250.0\">    <position x=\"$X_MAC\" y=\"$Y_MAC\"/>    <icon_id global=\"0\" local=\"0\"/>    <characteristic>        <process number=\"100\" power=\"661250.0\"/>        <memory size=\"100000.0\"/>        <hard_disk size=\"10000000.0\"/> </characteristic><master scheduler=\"RoundRobin\">$STRING </master> /></machine>"
echo $HEAD > arquivo$1.imsx
echo $MACHINE >> arquivo$1.imsx


for i in $(eval echo {1..$1})
        do
                BANDA=1000 #$(echo $[ $RANDOM % 100 + 1 ])
                ID_LINK=$(($ID_LINK+1))
                #LATENCY=`bc <<< "scale=2;$(echo $[ $RANDOM % 100 + 10 ])/10000"`
                LATENCY=0.005
                LOAD=0 #`bc <<< "scale=2;$(echo $[ $RANDOM % 1000 + 1 ])/1000"`
                X=$(($X+10))
                Y=$(($Y+10))
                X2=0
                Y2=0

                ENERGY=100 #$(echo $[ $RANDOM % 100 + 30 ])
                ID_MAC=$(($ID_GLOBAL+1))
                LOAD_MAC=0 #`bc <<< "scale=2;1*0.$(echo $[ $RANDOM % 10 + 3 ])"`
                POWER=$(echo $[ $RANDOM % 30000+ 10000 ])
                X_MAC=$X
                Y_MAC=$Y


                HARD_DISK=0 #$(echo $[ $RANDOM % 10000 + 3000 ])
                PROCESS=1 #$(echo $[ $RANDOM % 7 + 3 ])
                MEMORY=$(echo $[ $RANDOM % 35000 + 13000 ])

                ID_GLOBAL=$(($ID_GLOBAL+1))
                LOCAL_MAC=$(($ID_LOCAL+1))
                M=$ID_GLOBAL
                MACHINE="<machine energy=\"$ENERGY.0\" id=\"mac$ID_MAC\" load=\"0$LOAD_MAC\" owner=\"$USER_MAC\" power=\"$POWER.0\">    <position x=\"$X_MAC\" y=\"$Y_MAC\"/>    <icon_id global=\"$ID_GLOBAL\" local=\"$LOCAL_MAC\"/>    <characteristic>        <process number=\"20\" power=\"$POWER.0\"/>        <memory size=\"100000.0\"/>        <hard_disk size=\"10000.0\"/> </characteristic> </machine>"

                echo $MACHINE >> arquivo$1.imsx



                LINK="<link bandwidth=\"$BANDA.0\" id=\"link$ID_GLOBAL\" latency=\"0.01\" load=\"0$LOAD\">    <connect destination=\"$ID_MASTER\" origination=\"$ID_GLOBAL\"/>    <position x=\"0\" y=\"0\"/>    <position x=\"$X2\" y=\"$Y2\"/>    <icon_id global=\"$ID_GLOBAL\" local=\"$ID_LOCAL\"/> </link>"


                echo $LINK >> arquivo$1.imsx

                ID_GLOBAL=$(($ID_GLOBAL+1))
                ID_LOCAL=$(($ID_LOCAL+1))
                LINK="<link bandwidth=\"$BANDA.0\" id=\"link$ID_GLOBAL\" latency=\"0.01\" load=\"0$LOAD\">    <connect destination=\"$M\" origination=\"$ID_MASTER\"/>    <position x=\"$X2\" y=\"$Y2\"/>    <position x=\"0\" y=\"0\"/>    <icon_id global=\"$ID_GLOBAL\" local=\"$ID_LOCAL\"/> </link>"


                echo $LINK >> arquivo$1.imsx

done

echo $END >> arquivo$1.imsx
{% endhighlight %}

<h2>Automatizando o simulador</h2>

Outro problema, foi que o estudo que estava realizando, era necessário a
execução de 100, 1.000, 10.000 e 100.000 tarefas para cada uma das redes de 10,
50 e 100 máquinas, considerando que cada uma deveria ser realizada 20 vezes para
termos um resultado minimamente confiável.

Temos 4 quantidades de tarefas, 3 tipos de redes, então temos 12 ambientes
diferentes, multiplicado por 20 resulta em 240 execuções. Porém, neste estudo,
analisei 4 algoritmos diferentes, portanto seriam 960 execuções e ainda tem
mais uma observação, os testes seriam realizados em ambientes com máquinas
com processamento e comunicação heterogênea e também homogênea, o que significa 
que são 1920 execuções. Quando me deparei com esse problema, simplesmente
desisti e decidi automatizar tudo!
Infelizmente a documentação do iSPD não é muito vasta, mas navegando nos
diretórios de sua build, pude encontrar um arquivo interessante:

{% highlight bash %}
ispd/build/distributions/ispd-merge-1.0-SNAPSHOT/bin/ispd-merge
{% endhighlight %}

Quando realizei o comando abaixo, tive uma grata surpresa:

{% highlight bash %}
./ispd-merge --help

usage: java -jar iSPD.jar
 -a,--address <arg>      specify the server address.
 -c,--client             run iSPD as a client.
 -conf,--conf <arg>      specify a configuration file.
 -e,--executions <arg>   specify the number of executions.
 -h,--help               print this help message.
 -in,--input <arg>       specify the input file of themodel to simulate.
 -o,--output <arg>       specify an output folder for the html export.
 -P,--port <arg>         specify a port.
 -p,--parallel           runs the simulation parallel.
 -s,--server             run iSPD as a server.
 -t,--threads <arg>      specify the number of threads.
 -v,--version            print the version of iSPD.

{% endhighlight %}

Então gerei builds do iSPD com os algoritmos que desejava analisar e os coloquei
em diretórios específicos, criei o seguinte Bash Script:

{% highlight bash %}
for arquivo in $(ls homogeneo/*imsx)
do

    total=21
    sucesso=0

    while [ $total -gt $sucesso ]
    do
        nome=$(cut -d . -f 1 <<< $arquivo)
        ./ispd-merge -in ~/projeto/desenvolvimento/homogeneo/$arquivo -o $1$nome-bw-homogeneo-$(date +%s)  >> logs && sucesso=$(bc <<< $sucesso+1)
        ./ispd-merge -in ~/projeto/desenvolvimento/heterogeneo/$arquivo -o $1$nome-bw-heterogeneo-$(date +%s)  >> logs && sucesso=$(bc <<< $sucesso+1)
    done
done
{% endhighlight %}

Depois disso, pude deixar a execução dos algoritmos sendo realizada, o que
demorou pouco mais de 10 horas, enquanto isso pude estudar, escrever sobre
os algoritmos e fazer mais café :D


<h2>Conclusão</h2>

Muitas vezes temos que descobrir como resolver os problemas sem a possibilidade
de contar com a ajuda externa, no caso alguém implementar a funcionalidade de
criar redes aleatorizadas, o problema foi resolvido com algumas linhas de Bash
que podem assustar muitas pessoas, mas é simples :)
De quebra, descobri um bug no iSPD, não era possível gerar redes com mais de 402
máquinas e isso foi corrigido na nova versão do projeto, que está sendo
reescrita de Java para C++ por corajosos discentes da UNESP/IBILCE.

[iSPD]: https://dl.acm.org/doi/abs/10.5555/2331751.2331756?download=true
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
