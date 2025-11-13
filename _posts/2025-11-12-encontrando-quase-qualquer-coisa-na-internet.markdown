---
layout: post
title:  "Agulha no palheiro: dorks em motores de busca"
date:   2025-11-12 03:00:34 -0300
categories: osint dorks duckduckgo google-hacking
description: Não basta jogar palavras no Google, StartPage, DuckDuckGo e outros. Precisamos refinar a pesquisa!
image: https://github.com/user-attachments/assets/0adc02fb-7eb2-45b7-9186-086ba43d59b4
thumbnail: https://github.com/user-attachments/assets/ca3f66c0-f5a3-4b1f-92f1-d4dcb7eeb15b
---

<h2>OSINT</h2>

Nos anos 2000 começou a surgir a discussão da importância dos dados abertos, esses dados que hoje definem
boa parte da vida na socidade, publicações em redes sociais, fotos, vídeos, textos e nesse contexto surgiu
também o chamado Big Data, que nos anos 2000 era algumas centenas de Gigas e hoje falamos na ordem de Peta e Yota.

Sendo assim, surgiu o conceito de [Open Source Intelligence]{:target="\_blank"}, imagine a possibilidade de
extrair todos os vídeos e fotos de determinado lugar e horário, criar um personagem virtual que consegue andar
nessas informações e solucionar uma investigação. Isso já é feito faz alguns anos e você pode ver sobre
isso neste projeto chamado DéjàVu, o qual tive a oportunidade de ver a apresentação
feita pelo [Anderson Rocha]{:target="\_blank"}, sendo um exemplo
interessante de OSINT com Inteligência Artificial com potencial de solucionar diversos problemas.

Mas uma das ferramentas acessíveis para todas as pessoas, são os motores de busca
(criei a página [Google Hacking]{:target="\_blank"}), teoricamente simples de usar,
mas que podem esconder mais coisas do que podemos imaginar!
Por exemplo, com uma foto é possível descobrir o nome de alguém? Sim.
Com o nome dessa pessoa é possível descobrir o endereço e telefone? Depende, mas provavelmente sim.

Agora imagine extrair todos os dados da internet e utilizar isso para prever eventos problemáticos ou saber
que algo inesperado está ocorrendo quase em tempo real, nesse caso teríamos uma ferramenta de suporte à decisão
com fontes abertas, com acesso à informações sem burocracia e de forma ética, interessante, não?

Mas aqui, quero compartilhar algo que aprendi faz algum tempo e sempre me auxiliou muito em pesquisas diversas,
o uso de dorks, que são comandos especiais em motores de busca (Google, StartPage, DuckduckGo...alguém usou o
Altavista?).

Nosso roteiro é

[0 - como descobrir de onde é uma citação feita com aspas?](#0) \\
[1 - O que é falado sobre uma pessoa específica em sites específicos?](#1)\\
[2 - Existe alguma menção sobre um determinado livro realizado em alguma universidade?](#2)\\
[3 - Quais arquivos PDF existem sobre determinado livro?](#3)\\
[4 - É possível encontrar comentários de contas privadas no Instagram em perfis abertos?](#4)\\
[5 - Encontrar sistemas vulneráveis (não cometa nada ilegal, isso é apenas didático)?](#5)\\
[6 - Como pesquisar em todos os sites de universidades federais ou estaduais?](#6)\\
[7 - NULL](#7) 

<h2 id="0">0 - Como descobrir de onde é uma citação?</h2>

Essa é muito clássica, basta colocar o texto entre aspas e pesquisar no google ou em outro motor de busca
e os resultados entregarão os resultados caso exista, se não existir significa que provalvelmente a pessoa
tirou o texto da cabeça dela ou de algum LLM.

{% highlight None %}
"não há nada escondido que não venha a ser descoberto, ou oculto que não venha a ser conhecido"
{% endhighlight %}

O truque é que se você conhece apenas uma parte da citação, pode usar o asterísco:

{% highlight None %}
"não há nada escondido * oculto que não venha a ser conhecido"
{% endhighlight %}


<h2 id="1">1 - O que é falado sobre uma pessoa específica em sites específicos?</h2>

Podemos pesquisar em um site específico o que é falado sobre uma pessoa usando o comando inurl por exemplo:

{% highlight None %}
"george orwell" inurl:unesp.br
{% endhighlight %}

Ou então procurar em mais sites:

{% highlight None %}
"george orwell" (inurl:unesp.br OR inurl:usp.br OR inurl:ufrgs.br)
{% endhighlight %}

Podemos inclusive filtrar um intervalo de tempo:

{% highlight None %}
inurl:unicamp.br "George Orwell" after:2015-01-01 before:2018-01-01
{% endhighlight %}

<h2 id="2">2 - Existe alguma menção sobre um determinado livro realizado em alguma universidade?</h2>

Podemos pesquisar também o nome dos livros da mesma forma que o nome de uma pessoa,
mas no caso de livros podemos também restringir ao nome da pessoa com o AND:

{% highlight None %}
("a revolução dos bichos" AND "george orwell") inurl:unesp.br
{% endhighlight %}

<h2 id="3">3 - Quais arquivos existem sobre determiando livro?</h2>

{% highlight None %}
"a revolução dos bichos" filetype:pdf (inurl:unesp.br OR inurl:usp.br)
{% endhighlight %}

<h2 id="4">4 - É possível encontrar comentários de contas privadas no Instagram em perfis abertos?</h2>

As vezes as pessoas acham que só porque tem o perfil fechado em alguma rede social estão seguras,
infelizmente (ou felizmente) existem várias maneiras de ver o que alguém está fazendo nas redes sociais
mesmo com perfil fechado, uma das maneiras é simplesmente usando um motor de busca como o Google e utilizando
o comando:

{% highlight None %}
inurl:instagram.com intext:"nome\_do\_usuário"
{% endhighlight %}

Só utilizando esse simples comando é possível ver todos os comentários que determinado perfil fez em
páginas abertas no instagram por exemplo.

<h2 id="5" >5 - Pesquisando Sistemas Vulneráveis</h2>


Não apenas de Google o bom OSINTer vive, tem pesquisas interessantes que podemos fazer
em outros motores de busca (DuckDuckGo/StartPage/PubliWWW/Shodan), por exemplo,
é possível pesquisar o código fonte da página, o
que permite encontrar serviços que executam sistemas com vulnerabilidades conhecidas em sistemas ou outros,
por exemplo no PubliWWW podemos pesquisar uma string que corresponde à utilização
de um [plugin vulnerável no wordpress]{:target="\_blank"}:

{% highlight None %}
"contact-form-7/includes/css/styles.css?ver=5.3.1'"
{% endhighlight %}

Isso nos retorna diversos sistemas que estão vulneráveis faz alguns anos e podem servir de vetores de ataques
por alguma Botnet caso o agente que orquestre tenha capacidade de fazer isso. Mas além do PubliWWW existe nosso
queridíssimo shodan também, é uma ótima dica.

<h2 id="6">6 - Como pesquisar em todos os sites de universidades federais ou estaduais?</h2>

Montei este dork para me auxiliar em pesquisas para aprendizado de praticamente qualquer tema ou mesmo
consultar se existe alguma análise realizada por algum trabalho acadêmico, podemos pesquisar literalmente
todos os arquivos PDFs em todas as universidades federais e estaduais do Brasil (você pode criar um dork como
esse para outras universidades de outros países com um pouco de pesquisa).

No caso, vamos pesquisar em todas as universidades federais sobre escalonamento de tarefas utilizando
algoritmos meta-heurísticos, combinando os termos utilizando o +.

{% highlight None %}
"escalonamento de tarefas" + "meta-heurística" filetype:pdf (site:novo.ufra.edu.br OR site:portais.univasf.edu.br OR site:portal.ufcg.edu.br OR site:portal.ufgd.edu.br OR site:portal.ufpel.edu.br OR site:portal.ufrrj.br OR site:portal.ufvjm.edu.br OR site:portal.unifesp.br OR site:portal.unila.edu.br OR site:portalpadrao.ufma.br OR site:portalufj.jatai.ufg.br OR site:ufac.br OR site:ufal.br OR site:ufam.edu.br OR site:ufape.edu.br OR site:ufba.br OR site:ufc.br OR site:ufca.edu.br OR site:ufcat.edu.br OR site:ufcspa.edu.br OR site:ufdpar.edu.br OR site:ufersa.edu.br OR site:ufes.br OR site:ufg.br OR site:ufla.br OR site:ufmg.br OR site:ufms.br OR site:ufmt.br OR site:ufnt.edu.br OR site:ufob.edu.br OR site:ufop.br OR site:ufpa.br OR site:ufpr.br OR site:ufr.edu.br OR site:ufrb.edu.br OR site:ufrj.br OR site:ufrr.br OR site:ufsb.edu.br OR site:ufsc.br OR site:ufsj.edu.br OR site:uftm.edu.br OR site:ufu.br OR site:ufv.br OR site:unb.br OR site:unifal-mg.edu.br OR site:unifap.br OR site:unifei.edu.br OR site:unilab.edu.br OR site:unipampa.edu.br OR site:www.furg.br OR site:www.ufabc.edu.br OR site:www.uff.br OR site:www.uffs.edu.br OR site:www.ufopa.edu.br OR site:www.ufpb.br OR site:www.ufpe.br OR site:www.ufpi.br OR site:www.ufrgs.br OR site:www.ufrn.br OR site:www.ufrpe.br OR site:www.ufs.br OR site:www.ufscar.br OR site:www.ufsm.br OR site:www.uft.edu.br OR site:www.unifesspa.edu.br OR site:www.unir.br OR site:www.unirio.br OR site:www.utfpr.edu.br OR site:www2.ufjf.br)
{% endhighlight %}

Podemos pesquisar em todas as universidades estaduais do Brasil também (existe um limite de caracteres na busca
então é necessário separar as estaduais e federais) os trabalhos relacionados à segurança de computadores:

{% highlight None %}
("Control Flow Hijacking" OR "Shellcode" OR "buffer overflow" OR "heap overflow" OR "stack overflow" OR "Cross-Site Scripting")  filetype:pdf (site:uneal.edu.br OR site:uncisal.edu.br OR site:uneb.br OR site:uefs.br OR site:uesc.br OR site:uesb.br OR site:uece.br OR site:uva.br OR site:urca.br OR site:uema.br OR site:uemasul.edu.br OR site:uepb.edu.br OR site:upe.br OR site:uespi.br OR site:uern.br OR site:ueap.br OR site:uea.br OR site:uerr.br OR site:unitins.br OR site:ueg.br OR site:unemat.br OR site:uems.br OR site:uemg.br OR site:unimontes.br OR site:uem.br OR site:uepg.br OR site:uel.br OR site:unioeste.br OR site:unicentro.br OR site:unespar.edu.br OR site:uenp.br OR site:uergs.edu.br OR site:udesc.br OR site:usp.br OR site:unicamp.br OR site:unesp.br OR site:uerj.br OR site:uenf.br)
{% endhighlight %}

Gostaria de saber também o que é falado nas universidades sobre os livros "Armas de Destruição Matemática" ou "Inteligência Artificial a Nosso Favor"
(colocar em inglês o nome da obra também facilita encontrar mais referências):

{% highlight None %}
("Armas de Destruição Matemática" OR "Weapons of math destruction" OR "Human Compatible: Artificial Intelligence and the Problem of Control" OR "Inteligência Artificial a Nosso Favor")  filetype:pdf (site:uneal.edu.br OR site:uncisal.edu.br OR site:uneb.br OR site:uefs.br OR site:uesc.br OR site:uesb.br OR site:uece.br OR site:uva.br OR site:urca.br OR site:uema.br OR site:uemasul.edu.br OR site:uepb.edu.br OR site:upe.br OR site:uespi.br OR site:uern.br OR site:ueap.br OR site:uea.br OR site:uerr.br OR site:unitins.br OR site:ueg.br OR site:unemat.br OR site:uems.br OR site:uemg.br OR site:unimontes.br OR site:uem.br OR site:uepg.br OR site:uel.br OR site:unioeste.br OR site:unicentro.br OR site:unespar.edu.br OR site:uenp.br OR site:uergs.edu.br OR site:udesc.br OR site:usp.br OR site:unicamp.br OR site:unesp.br OR site:uerj.br OR site:uenf.br)
{% endhighlight %}

<h2 id="7"> NULL - Pesquisas automatizadas e geradores</h2>

Se observar outras publicações que já fiz nesse site e em outros lugares, perceberá que tenho interesse
em realizar extrações em massa de informação e quero contar uma história. Uma vez trabalhando para uma empresa,
me pediram para procurar perfis de pessoas em determinada rede social para captar clientes em potencial, sendo
assim, juntei o que sabia que web scrapping e dorks e fiz um sistema que me entregava milhares de perfis de
brasileiros extraídos e depois filtrei chegando em algumas centenas e dito isso, queria dar a dica de diversos
repositórios no github mesmo, que tem vários códigos desenvolvidos por pessoas do mundo todo (muito cuidado em
executar qualquer código em sua máquina, a responsabilidade é sua).

Então temos, por exemplo: https://github.com/topics/google-dorks

Se observamos essa URL, podemos também fazer dork em cima dela:
{% highlight None %}
("dorks" OR "osint" OR "open source intelligence" OR "dork" OR "dorking" ) inurl:https://github.com/topics 
{% endhighlight %}

<h3>Conclusão</h3>

Open Source Intelligence (OSINT) é uma pauta interessantíssima que mistura diversos conhecimentos,
de Web Scrapping até Processamento de Sinais e com as perguntas certas podemos descobrir muito sobre diversos
tópicos! Se você leu até aqui, mande para seu colega que gosta de descobrir informações por diversão :D


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
