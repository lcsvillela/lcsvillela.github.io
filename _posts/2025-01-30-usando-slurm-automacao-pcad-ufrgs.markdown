---
layout: post
title:  "Gerenciando tarefas no Slurm e automatizando"
date:   2025-01-30 03:00:34 -0300
categories: slurm hpc bash linux
description: Computação de alto desempenho com Slurm!
image: https://github.com/user-attachments/assets/e2217870-ef98-44d5-a59c-b0a08101792a 
thumbnail: https://github.com/user-attachments/assets/e2217870-ef98-44d5-a59c-b0a08101792a
---

<h2>O que é o Slurm?</h2>

Imagine a siguinte situação, você tem muitos computadores e seu uso será compartilhado, pois quanto mais tempo
um dispositivo passa ocioso, mais dinheiro é desperdiçado. Nesse caso, a centralização de infraestrutura
computacional compartilhada é algo ideal, pois possibilita que os recursos fiquem mais tempo ocupados e se existir
um meio de gerenciar a utilização em forma de filas, então temos um cenário incrívelmente bom.

Para esses casos, o [Slurm]{:target="\_blank"} é uma ferramenta sensacional, não é por acaso que
60% dos TOP500 utilizam essa ferramenta para gerir seus recursos.
Se você é discente do INF/UFRGS, deve(ria) conhecer o PCAD e este texto foi feito baseado no que utilizei 
na disciplina de mestrado de Introdução a Computação de Alto Desempenho e fizeram
parte do meu projeto para a disciplina, com esse conteúdo, você também terá a capacidade de automatizar todos as
tarefas para serem executadas quantas vezes quiser, sem precisar se preocupar com a administração da
infraestrutura em si e contando com as [melhores placas de GPU]{:target="\_blank"} do mercado,
porque o time que administra o PCAD é incrível!

<h2>Comandos básicos do Slurm</h2>

Os comandos do Slurm podem ser utilizados de forma interativa ou em lote, no caso podemos ter interesse de ter
acesso a apenas uma máquina, mas primeiro precisamos saber a mesma está disponível para utilização. Na página
do [PCAD]{:target="\_blank"} tem um resumo muito bom sobre os comandos do Slurm no PCAD
e recomendo muitíssimo a leitura para a continuação deste texto.

<h4>sinfo</h4>

Com o comando sinfo, podemos verificar se tem alguma máquina disponível para uso imediato! Se sim, podemos
utilizar o modo iterativo, apesar de ser mais recomendado o sbatch.

{% highlight bash %}
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
shared*      up 1-00:00:00      3  down* bali1,orion[1-2]
shared*      up 1-00:00:00      9   idle bali2,draco[1-7],turing
blaise       up 1-00:00:00      1  alloc blaise
sirius       up   infinite      1   idle sirius
tupi         up 1-00:00:00      1  down* tupi3
tupi         up 1-00:00:00      4  alloc tupi[1-2,4,6]
tupi         up 1-00:00:00      1   idle tupi5
cidia        up 2-00:00:00      1   idle cidia
orion        up 1-00:00:00      2  down* orion[1-2]
draco        up 1-00:00:00      7   idle draco[1-7]
beagle       up 1-00:00:00      1   idle beagle
marcs        up 3-00:00:00      1   idle marcs
poti         up 1-00:00:00      5   idle poti[1-5]
lunaris      up 1-00:00:00      1   idle lunaris
{% endhighlight %}

<h4>salloc</h4>

Podemos criar um job iterativo, é bem simples de se fazer e é para termos acesso a um shell e fazer as tarefas
manualmente:

{% highlight bash %}
salloc -p tupi -w tupi1 -J teste -t 00:15:00
{% endhighlight %}

Nesse caso alocamos a partição tupi1 por 15 minutos e demos o nome do job de teste, depois disso basta acessar a
máquina com ssh e o shell ficará disponível:

{% highlight bash %}
ssh tupi1
{% endhighlight %}

Para poupar a rede, devemos usar sempre o $SCRATCH para salvar os arquivos de registro ou para arquivos que
serão utilizados no programa.

<h4>Truque com Sbatch</h4>

O sbatch pode receber um arquivo de entrada, assim como é mostrado no portal do PCAD, porém gostaria de mostrar
um uso diferente, imagine que seja necessário executar um experimento em 10 máquinas diferentes do PCAD, seria
necessário criar um arquivo para para máquina para o sbatch receber e se for necessário alterar os comandos,
o tempo de execução ou qualquer outra informação, será que é uma boa ideia usar esse modelo?

O Bash é uma linguagem maravilhosa e assim como a linguagem C/C++, tudo é um fluxo (stream) de dados, podemos por
exemplo utilizar os redirecionadores <<< para criar um arquivo que vale por 10, 100 e quantos fossem necessários!


{% highlight bash %}
MACHINES="blaise tupi draco draco beagle marcs poti sirius lunaris grace"

for i in $MACHINES
do

sbatch <<< "#!/bin/bash
#SBATCH --job-name=experimento-$i-$1
#SBATCH --partition=$i
#SBATCH --nodes=1
#SBATCH --ntasks=16
#SBATCH --time=23:00:00
#SBATCH --output=%x_%j.out
#SBATCH --error=%x_%j.err

./experimento.sh $1"
done
{% endhighlight %}

Implementei o código acima durante a disciplina de Introdução a Computação de Alto Desempenho (CMP270) do
PPGC da UFRGS, o que permitiu fazer um experimento com mais de 5000 execuções, completamente automatizado e o
código está disponível: [HPC UFRGS]{:target="\_blank"}

<h4>Cancelando jobs</h4>

Obviamente podemos querer cancelar alguma job que foi enviado para a fila de execução, então podemos ver as
informações com o comando squeue e cancelar com scancel:

{% highlight bash %}
lcsvillela@gppd-hpc:~$ squeue

 JOBID  PARTITION                    NAME            USER    STATE         TIME   TIME\_LIMIT  NODES               NODELIST(REASON)  CPUS
699852     blaise         lcsvillela      lcsvillela  RUNNING     22:08:55   1-00:00:00      1                         blaise    88

lcsvillela@gppd-hpc:~$ scancel 699852
{% endhighlight %}

<h3>Conclusão</h3>

O Slurm é uma ferramenta incrível de se utilizar, pois permite o compartilhamento de hardware de forma eficiente,
permitindo que o acesso à infraestrutura seja ampliado e possibilitando que experimentos sejam realizados por
todos os discentes que necessitarem de GPUs por exemplo, que é um equipamento de alto custo.


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
