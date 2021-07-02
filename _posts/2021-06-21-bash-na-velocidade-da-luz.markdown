---
layout: post
title:  "Bash na velocidade da luz"
date:   2021-07-1 20:00:34 -0300
categories: programacao_bash linux
image: https://user-images.githubusercontent.com/23728459/124209653-56d04d80-dac0-11eb-9e10-6e1b87d8a2e2.jpg
---

leia também: [Bash pra que te quero!]{:target="\_blank"}

Muitos falam da velocidade do java, python e até mesmo bash, porém
quem reclama da velocidade do Java, de acordo com o livro 
[Java Core - Volume 1]{:target="\_blank"}, não tem conhecimentos sobre
[multithreading e processamento paralelo]{:target="\_blank"} em [Java]{:target="\_blank"}, portanto não sabe 
sobre a linguagem e o mesmo serve para o Bash, se a gente tem a possibilidade de usar esses recursos,
então podemos chegar na [velocidade da luz]{:target="\_blank"} no nosso sistema. A ideia aqui é mostrar um 
pouco sobre a velocidade na comunicação [HTTP]{:target="\_blank"}, no caso, para bots do
[Telegram]{:target="\_blank"}! Vamos tentar enviar 60 mensagens para o nosso bot.


<h2>Criando Bot do Telegram</h2>

Para criar o Bot do Telegram, você precisa acessar o aplicativo e procurar
pelo BotFather, após encontrá-lo e dar start, você pode enviar comandos para ele,
no caso precisamos enviar os seguintes comandos:


![01](https://user-images.githubusercontent.com/23728459/124205959-c04c5e00-dab8-11eb-9666-4fde86676d8a.png){: style="float: center" class="img-responsive" height="70%" width="70%"}


<h2>Enviando mensagens</h2>

Para realizar o envio de mensagens, precisamos primeiro pegar o ID do perfil
que iremos enviar a mensagem, envie uma mensagem qualquer para seu bot no
telegram pelo seu usuário e então execute o comando para pegar as mensagens
enviadas, podemos fazer a requisição utilizando a ferramenta curl:

{% highlight bash %}
TOKEN="seu token"
curl https://api.telegram.org/bot$TOKEN/getUpdates | \
jq -c '.result[].message.from.id'
{% endhighlight %}

Depois que pegar nosso ID, podemos agora enviar uma mensagem para nós mesmos ou
para um usuário qualquer que entrou em contato.

No nosso exercício, vamos enviar 60 mensagens pra o usuário pelo Bot do
Telegram, para fazer isso, podemos escrever em um arquivo texto com seu editor
favorito (Use o [Vim]{:target="\_blank"}!):

{% highlight bash %}
TOKEN="seu token"
URL="https://api.telegram.org/bot$TOKEN/sendMessage"
ID=$(curl https://api.telegram.org/bot$TOKEN/getUpdates | \
jq -c '.result[].message.from.id')

for mensagem in {1..60}
do
    curl -s -X POST $URL -d chat_id=$ID -d text="$mensagem"
done
{% endhighlight %}

Após salvar o seu arquivo, precisamos dar permissão de execução no sistema, para
isso o seguinte comando:

{% highlight bash %}
chmod +x arquivo.sh
{% endhighlight %}

Para executar nosso programa medindo seu tempo, vamos usar o comando time, então executamos:

{% highlight bash %}
time ./arquivo.sh
{% endhighlight %}

Podemos ver ao vivo também no nosso telegram, enquanto recebemos uma mensagem por vez,
de forma tão lenta que poderíamos ir pegar um cafezinho e na volta,
notaríamos que ainda está lá sendo executado. No meu caso, o resultado
do tempo de execução foi:

{% highlight bash %}
real	0m53,014s
user	0m1,085s
sys	0m0,493s
{% endhighlight %}

Enviamos 60 mensagens em 53 segundos, que é uma eternidade quando
falamos em computação e nesse tempo a luz já deu cerca de 397 voltas na Terra,
podemos reduzir esse tempo drasticamente usando forks! A diferença
no código é sutil no caso do bash, mas em outras linguagens como python, java ou
C é um pouco mais extenso. Um outro jeito de utilizar forks é utilizando o comando [coproc]{:target="\_blank"}

<h2>Enviando mensagens na velocidade da luz!</h2>

Para executar um comando em um processo separado (fork, que não é multithreding) jogamos todo o
trabalho para o init (você pode ver isso acontecendo usando por exemplo a ferramenta htop),
o principal processo do sistema para administrar o problema. O efeito na prática, é que o
processamento paralelo tem efeito notório na velocidade.

{% highlight bash %}
TOKEN="seu token"
URL="https://api.telegram.org/bot$TOKEN/sendMessage"
ID=$(curl https://api.telegram.org/bot$TOKEN/getUpdates | \
jq -c '.result[].message.from.id')

for mensagem in {1..60}
do
    $(curl -s -X POST $URL -d chat_id=$ID -d text="$mensagem" >> log)&
done
{% endhighlight %}

O tempo de execução retornado foi:

{% highlight bash %}
real	0m0,031s
user	0m0,005s
sys	0m0,012s
{% endhighlight %}

O mais interessante do uso de forks (as vezes chamam de subshell), é que essa ferramenta de executar comandos em
subshell, pode ser utilizado em qualquer lugar do código, sendo assim podemos enviar informações do nosso sistema:

{% highlight bash linenos %}
TOKEN="seu token"
URL="https://api.telegram.org/bot$TOKEN/sendMessage"
ID=$(curl https://api.telegram.org/bot$TOKEN/getUpdates | \
jq -c '.result[].message.from.id')

for mensagem in {1..60}
do
    $(
        TEMPERATURA="$(sensors)"
        curl -s -X POST $URL -d chat_id=$ID -d \ 
        text="$TEMPERATURA" >> log
    )&
done
{% endhighlight %}

Porém, você deve ter percebido algo estranho, o nosso processo foi finalizado antes de recebermos todas as mensagens!
Isso acontece porque, pelo fato de serem processos independentes, cada um segue sua vida e é finalizado de forma
independente, pois são todos gerenciados pelo init. Para fazermos o programa esperar que todos os processos sejam
terminados, podemos usar o comando wait.

{% highlight bash linenos %}
TOKEN="seu token"
URL="https://api.telegram.org/bot$TOKEN/sendMessage"
ID=$(curl https://api.telegram.org/bot$TOKEN/getUpdates | \
jq -c '.result[].message.from.id')

for mensagem in {1..60}
do
    $(
        TEMPERATURA="$(sensors)"
        curl -s -X POST $URL -d chat_id=$ID -d \ 
        text="$TEMPERATURA" >> log
    )&
done
wait
{% endhighlight %}

Quando nosso processo acabou, ele ficou esperando os processos que estavam rodando no background serem finalizados, para
então poder ser finalizado, isso nos dá o tempo real de execução de todos os processos:

{% highlight bash %}
real	0m8,007s
user	0m1,470s
sys	0m0,579s
{% endhighlight %}

um exemplo com coproc:

{% highlight bash %}
TOKEN="seu token"
URL="https://api.telegram.org/bot$TOKEN/sendMessage"
ID=$(curl https://api.telegram.org/bot$TOKEN/getUpdates | \
jq -c '.result[].message.from.id')

for mensagem in {1..60}
do
    coproc curl -s -X POST $URL -d chat_id=$ID -d text="$mensagem" >> log
done
{% endhighlight %}

Então agora sabemos que o tempo de realizar 60 conexões simultâneas utilizando os processos múltiplos do Bash demora
cerca de 60 voltas da luz na [Terra]{:target="\_blank"}, ou 8 segundos, se preferir. Porém, caso desejemos rodar algo no sistema utilizando
de fato multithreading e não fork? Para isso, existe a ferramenta [GNU Parallel]{:target="\_blank"} e 
este ótimo [livro sobre] .
Porém, algo a se considerar é que o Telegram [tem limitações]{:target="\_blank"} de envio por segundo, 
que são 30 por segundo.

<h2>GNU Parallel</h2>

Esta ferramenta escrita em [Perl]{:target="\_blank"} tem referências sólidas, não apenas faz parte
do [projeto GNU]{:target="\_blank"}, mas utilizado amplamente em [diversas pesquisas]{:target="\_blank"}

Podemos por exemplo trabalhar com processamento paralelo para leitura de arquivos muito grandes, coisa de milhares,
milhões, bilhões de linhas, sendo algo que pode demorar muito se for feito linearmente o processo, então o parallel
pode agilizar essa parte.

Um exemplo legal é para combinações de [RNA]{:target="\_blank"}, onde diferentes combinações podem ter o mesmo efeito no organismo,
então quando se faz algum tipo de análise utilizando [bioinformática]{:target="\_blank"} é interessante combinar todas as sequências
possíveis e assim, temos todas as sequências equivalentes entre si, algo do tipo:
{% highlight bash %}
parallel echo ::: GTA CGG ACA CAA  ::: GTA CGG ACA CAA
{% endhighlight %}
O comando que queremos executar fica sempre antes de :::, após isso fica as entradas que queremos realizar
separadas por espaço. Como temos duas entradas de informação, o resultado será a combinação das duas, algo como:
{% highlight bash %}
GTA GTA
GTA CGG
GTA ACA
GTA CAA
CGG GTA
CGG CGG
CGG ACA
CGG CAA
ACA GTA
ACA CGG
ACA ACA
ACA CAA
CAA GTA
CAA CGG
CAA ACA
CAA CAA
{% endhighlight %}

Se quiséssemos contar as linhas de milhares de arquivos em um sistema, podemos utilizar:
{% highlight bash %}
parallel wc -l ::: arquivos.*
{% endhighlight %}

Se quiséssemos copiar um arquivo em nosso sistema para centenas de computadores de uma rede:

{% highlight bash %}
parallel scp programa ::: 192.168.0.{2..200}:/usr/bin/
{% endhighlight %}

Ou como estávamos vendo, enviando 100 mensagens no Telegram na velocidade da luz!


{% highlight bash linenos %}
TOKEN="seu token"
URL="https://api.telegram.org/bot$TOKEN/sendMessage"
ID=$(curl https://api.telegram.org/bot$TOKEN/getUpdates | \
jq -c '.result[].message.from.id')
parallel curl -s -X POST $URL -d chat_id=$ID -d \ 
           text="{}" ::: TEMPERATURA="$(sensors)"
{% endhighlight %}

Essa ferramenta maravilhosa tem diversas funções importantes para lidar com processamento paralelo, como por exemplo
semáforo, mutex e funções de sincronização. Tudo isso cria facilitação na hora de criar bashs ou qualquer outro programa
de qualquer linguagem usando processamento paralelo.

<h2>Conclusão</h2>

Com processamento paralelo, é possível executar nossos programas usando todo o poder de processamento disponível,
passamos de minutos para segundos, de horas para minutos e de dias para horas, também derrubamos o mito de que uma
linguagem qualquer é ruim porque é lenta, é lenta porque usamos a linguagem da forma errada, então é culpa nossa e não
da linguagem (inclusive Ruby é usado nos servidores do Github, uma linguagem que muitos dizem ser lenta).


[Bash pra que te quero!]: https://lcsvillela.github.io/bash-pra-que-te-quero.html
[Java Core - Volume 1]: https://blogs.oracle.com/javamagazine/core-java-11th-ed-volumes-1-and-2
[multithreading e processamento paralelo]: https://www.linuxjournal.com/article/1363
[Java]: https://pt.wikipedia.org/wiki/Java_(linguagem_de_programa%C3%A7%C3%A3o)
[velocidade da luz]: https://pt.wikipedia.org/wiki/Velocidade_da_luz
[HTTP]: https://pt.wikipedia.org/wiki/Hypertext_Transfer_Protocol
[Vim]: https://pt.wikipedia.org/wiki/Vim
[coproc]: https://www.gnu.org/software/bash/manual/html_node/Coprocesses.html
[Terra]: https://pt.wikipedia.org/wiki/Terra
[GNU Parallel]: https://www.gnu.org/software/parallel/
[livro sobre]: https://zenodo.org/record/1146014
[tem limitações]: https://hackernoon.com/experiences-building-a-high-performance-telegram-bot-1e6bb70dcaac
[diversas pesquisas]: https://www.sciencedirect.com/science/article/pii/S1476927117305753
[RNA]: https://pt.wikipedia.org/wiki/%C3%81cido_ribonucleico
[bioinformática]: https://www.ufrgs.br/bioinfo/o-que-e-bioinformatica/
[Perl]: https://pt.wikipedia.org/wiki/Perl
[Projeto GNU]: https://www.gnu.org/gnu/gnu.html
