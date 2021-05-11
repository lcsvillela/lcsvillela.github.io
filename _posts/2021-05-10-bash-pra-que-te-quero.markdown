---
layout: post
title:  "Bash pra que te quero!"
date:   2021-04-15 20:00:34 -0300
categories: programacao_bash linux
image: https://user-images.githubusercontent.com/23728459/117757604-c3cd1280-b1f6-11eb-8493-523560ed10eb.png
---

Um dos meus primeiros livros sobre [Linux]{:target="\_blank"}, foi o
[Shell Script Profissional]{:target="\_blank"}, um livro que revirou de ponta
cabeça tudo o que achava que sabia e o que não sabia. O Bash é muito mais do
que a linha de comando, é a alavanca que podemos usar para [mover o mundo]{:target="\_blank"}!

Além deste livro, destaco também a importância do
[Conceitos de Linguagens de Programação], que faz a gente ter uma percepção
um pouco além da luz. Analisando a flexibilidade, facilidade de leitura e outros aspectos da linguagem.

<h2>Seu primeiro .sh</h2>

Abre o seu editor de texto preferido, no meu caso o
[vim]{:target="\_blank"} e edite um arquivo texto com o nome:
primeiro\_shell.sh

{% highlight bash %}
read NOME
echo "Olá $NOME, tudo bem com você?"
echo 'Olá $NOME, tudo bem com você?'
{% endhighlight %}

Depois de fechar o arquivo, vocẽ precisa que o sistema operacional permita
que ele seja executado, para isso usamos o [chmod]{:target="\_blank"}:
{% highlight bash %}
chmod +x primeiro_shell.sh
{% endhighlight %}

Para executá-lo:
{% highlight bash %}
./primeiro_shell.sh
{% endhighlight %}

Neste caso, teremos duas linhas exibidas na nossa telinha preta, a primeira
mostrará o nome que digitamos e a segunda mostrará o nome da variável.
Isso acontece porque quando usamos aspas simples, informamos ao bash que
é para tratar como texto puro.

Porém, é um pouco chato ficar esperando para digitar o seu nome, não é? Para isso, o bash tem variáveis especiais, que são criadas para cada vez que chamamos um programa, vamos alterá-lo para usar isso!

{% highlight bash %}
echo "Olá $1, tudo bem com você?"
echo 'Olá $1, tudo bem com você?'
{% endhighlight %}

Salve o arquivo e feche-o, execute-o:

{% highlight bash %}
./primeiro_shell.sh joao
{% endhighlight %}

A variável $1 tem o valor joao e a variável $0 é o nome do programa
(quase tudo começa com zero na computação), se tivesse outro nome na frente do joao 
seria $2 e assim por diante...mas então, se eu colocar 5 nomes, vou ter
que listar as 5 variáveis no código para exibir o conteúdo de todos os
nomes? E se colocar 100 nomes, vou ter que escrever da $1 até a $100?
Impraticável!

<h2>O laço while e fazendo a fila andar!</h2>

A fila anda e a vida segue, com a ajuda de um laço while fica ainda mais fácil,
não precisamos escrever todas as variáveis possíveis que o usuário pode digitar:

{% highlight bash linenos %}
#! /bin/bash
while [ ! -z $1 ]
do
    echo '$1': $1
    shift
    sleep 2
done
{% endhighlight %}

A novidade nesse novo bash reside no laço while, onde colocamos a condição de
que a variável $1 deve ser não nula para que o laço continue, depois a linha 5
usamos o comando shift, que faz com que a fila ande uma posição.
Com isso e um pouco de criatividade, já começamos a ter capacidade de criar
coisas mais complexas, que tal implementarmos aquelas opções do padrão
[POSIX] em nosso .sh?

<h2>Criando opções e restringindo o fluxo</h2>

Você pode, usando opções, restringir o fluxo do seu programa, ou seja, dizer de
forma muito específica para o usuário o que você precisa que seja informado. No
caso, vamos criar as opções --name, --age e --city.

{% highlight bash linenos %}
#! /bin/bash

while [ ! -z $1 ]
do
    case "$1" in
        --name|-n) NAME=$2; shift;;
        --age|-a) AGE=$2; shift;;
        --city|-c) CITY=$2; shift;;
    esac
    shift
done

echo $NAME
echo $AGE
echo $CITY
{% endhighlight %}

Agora nosso sh está bem mais poderoso! Podemos não apenas rotacionar as
variáveis para facilitar a implementação do código, mas também colocar opções
para que o usuário digite. Faça o teste executando:

{% highlight bash linenos %}
./arquivo.sh -n Fulano -a 18 -c Recife
./arquivo.sh -n Fulano -c Recife -a 18
./arquivo.sh -a 18 -n Fulano -c Recife
{% endhighlight %}

A saída para cada execução será a mesma, nosso script está se tornando um
programa de verdade! Com o controle de fluxo nesses termos, não importa a ordem
das opções, o programa funciona!

<h2>Consumindo APIs</h2>

Muitas vezes nosso computador é algo sem muito poder de processamento, ele pode
ser por exemplo apenas um visor com um processador bem fraquinho, porém ele nos
mostra a previsão do tempo dos próximos 15 dias, isso é no mínimo curioso,
não é? Para fazer a previsão do tempo, precisamos de muitos dados, no mínimo, é
necessário saber onde você está, então temos as latitudes e longitudes de todas
as cidades e ainda por cima tem os cálculos meteorológicos...enfim, o seu
celular não vai fazer isso, ele manda alguém fazer! É para isso que serve uma
[API (Interface de Programação de Aplicações)]{:target="_blank"},
seu celular envia suas coordenadas geográficas para um servidor em algum
lugar do mundo [que espiona você]{:target="_blank"} e com estes dados, retorna
para você os dados do clima do seu quarteirão e já recomenda a
lanchonete da esquina, porque sabe que faz tempo que você não vai lá.

Porém, uma outra coisa legal que podemos fazer também, é acompanhar o preço do
[Bitcoin]{:target="_blank"} em Reais e lamentar por não termos mineirado em 2011.

{% highlight bash linenos %}
#! /bin/bash

DOLAR_EM_REAL=$(curl -s https://economia.awesomeapi.com.br/json/daily/USD-BRL | jq .[0].low | tr -d '"')
BITCOIN_EM_DOLAR=$(curl -s https://api.coindesk.com/v1/bpi/currentprice.json | jq .bpi.USD.rate_float)
BITCOIN_EM_REAL=$(bc <<< "$DOLAR_EM_REAL*$BITCOIN_EM_DOLAR") 
BR=$(echo -e R\$$BITCOIN_EM_REAL | tr '.' ',')
zenity --notification --text="Valor do Bitcoin: $BR"
{% endhighlight %}

Aqui consumimos duas APIs diferentes, uma nos retorna o preço do Dólar em Reais
e a outra retorna o preço do Bitcoin em dólares, bastando multiplicarmos
no final. Neste script, temos o bônus de realizarmos uma notificação bonita no
canto de nossa tela, tudo isso com auxílio do zenity.

Outra mudança notável, é o uso de $(comandos), quando fazemos isso, jogamos
apenas o resultado do comando dentro dos parênteses pra fora, algo muito
útil para definição de variáveis, como foi o caso do exemplo.

<h2>Dicas de sobrevivência</h2>

Programar em Bash é algo muito divertido, porém envolve diversos riscos, visto
que nos dá muito poder e com muitos poderes, temos muitas responsabilidades.
Imagine por exemplo o que poderia acontecer neste caso:

{% highlight bash %}
rm -rf /$1
{% endhighlight %}

O usuário poderia não informar nada e executar despretenciosamente o programa,
o resultado seria a destruição de seu sistema e a iminente implosão da sua vida.
Parece absurdo? Mas não é, programas de 1000 linhas ainda podemos controlar se
escrevermos de qualquer jeito, sem indentação e com nomes péssimos de variáveis,
mas com o tempo, isso se torna inviável e nosso programa script vai pro lixo.
Nada pior do que um programa que não se pode editar!

Além disso, quanto mais você dominar o Linux, mais você vai dominar o Shell Script, permitindo que você mesmo crie até mesmo novos comandos (ora bolas, crie
seu binário e jogue no /bin). O Shell em geral, é considerado uma linguagem de
programação do tipo "cola" como Lua. Podemos criar um programa em Java, um em Python e talvez outro em PHP e unificar o funcionamento dos três usando bash e seus redirecionadores!

Algo a se considerar, é que o Linux ensina ele mesmo à você, leia a [manpage]{:target="_blank"} dos comandos usados aqui digitando em seu terminal:

{% highlight bash %}
man zenity
man bash
man bc
man curl
man jq
{% endhighlight %}

<h2>Conclusão</h2>

"Metade do mundo hoje é feito em C a outra metade é feita em Python, mas Python
também é feito em C", essa piada ilustra muito bem nossa realidade, C permeia o
mundo todo, inclusive o Bash que aqui abordei brevemente. Faça como o [Júlio Neves]{:target="_blank"}, estude a sua linguagem de programação favorita pelo código fonte! Você verá
que o Bash é em C, o Python é em C (que C é em C rs) e que elas tem muitas coisas em comum.

[que espiona você]: https://www.vice.com/pt/article/gy77wy/pare-de-usar-apps-de-previsao-do-tempo-eles-tao-vendendo-seus-dados-por-ai
[Júlio Neves]: https://www.youtube.com/watch?v=ryoH4wZTshw
[manpage]: http://manpages.org/bash
[API (Interface de Programação de Aplicações)]: https://pt.wikipedia.org/wiki/Interface_de_programa%C3%A7%C3%A3o_de_aplica%C3%A7%C3%B5es
[POSIX]: https://pt.wikipedia.org/wiki/POSIX
[Linux]: https://pt.wikipedia.org/wiki/Linux
[Shell Script Profissional]: https://www.shellscript.com.br/
[mover o mundo]: https://mundoeducacao.uol.com.br/matematica/uso-das-proporcoes-na-teoria-alavancas.htm
