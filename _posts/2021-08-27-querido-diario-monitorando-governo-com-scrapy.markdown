---
layout: post
title:  "Querido Diário - Monitorando o governo de Americana/SP"
date:   2021-08-27 16:00:34 -0300
categories: python webscraping scrapy
description: Aqui descrevo como contribuí para o projeto Querido Diário, o qual tem o propósito de baixar todos os diários oficiais do Brasil, sendo um ótimo meio de monitorar ações do governo
image: https://user-images.githubusercontent.com/23728459/131188661-9fd6453c-ba20-4a67-b26f-f0c180026ff9.jpg
---


_Escrevi um texto bem legal sobre o um uso do [Scrapy], confira [aqui]._

![monitorado](https://user-images.githubusercontent.com/23728459/131188661-9fd6453c-ba20-4a67-b26f-f0c180026ff9.jpg){: style="float: left; margin-right: 1em;" class="img-responsive" height="40%" width="40%"}

O [Open Knowledge Brasil]{:target="\_blank"} é um movimento que luta pelo
conhecimento livre, um dos muitos projetos é o
[Querido Diário]{:target="\_blank"}, o qual colaborei criando uma 
[spider (aranha)]{:target="\_blank"}, para realizar a coleta de todos os
diários oficiais do município de [Americana]{:target="\_blank"}, até o 
momento foram 800 diários oficiais adicionados
[nesta colaboração]{:target="\_blank"}. A principal ideia de uma spider, é
andar em diversos links da internet de forma automatizada, geralmente para
realizar a extração dessas informações web. Se você tem dúvida entre usar
[Selenium]{:target="\_blank"}, [BeautifulSoap]{:target="\_blank"} e
[Scrapy]{:target="\_blank"}, saiba que cada ferramenta tem sua utilidade,
nesse caso o projeto utiliza o Scrapy, pois se trata de milhares de páginas.

Qualquer pessoa pode colaborar, não
importa se você pensa [assim ou assado, ou se é x ou y]{:target="\_blank"}.


<h2>Clonando o repositório e configurando o ambiente</h2>

O projeto tem algum tempo, então a nossa colaboração não foi a primeira, então
vamos baixar primeiro o código já existente.

{% highlight bash %}
git clone https://github.com/okfn-brasil/querido-diario
{% endhighlight %}

Depois disso, o projeto foi clonado para seu computador, sendo assim, temos
boa parte do ambiente necessário, mas ainda precisamos de um pouco mais, no
caso precisamos criar nosso ambiente virtual no python, para garantir que
nosso sistema operacional não seja afetado.

{% highlight bash %}
pip3 install virtualenv
virtualenv --python=python3.9 venv
source venv/bin/activate
{% endhighlight %}

Agora com nosso ambiente criado e ativo (deve ter aparecido venv do lado
esquerdo no seu terminal), agora podemos instalar os requerimentos do projeto.

{% highlight bash %}
pip install -r querido-diario/data_collection/requirements.txt
{% endhighlight %}

Depois disso, você tem o ambiente completo configurado no seu computador,
podendo executar qualquer spider existente no projeto. Se você for até o
diretório das spiders, poderá encontrar um punhado delas!

{% highlight bash %}
cd querido-diario/data_collection/gazette/spiders
{% endhighlight %}

Agora podemos executar por exemplo, a spider da cidade de Jaú, do Estado de
São Paulo:

{% highlight bash %}
scrapy crawl sp_jau
{% endhighlight %}

Com esse comando, a spider será executada e todos os diários oficiais da cidade
serão baixados no seu computador, hoje (27 de agosto de 2021) ela está funcinal.
Mas além de executar, a ideia é criar uma spider nova, então notei no arquivo
CITIES.md, que tem todas as cidades e as spiders existentes, que faltam muitas
cidades. Peguei uma lista das 200 maiores cidades do Estado de SP e filtrei as
que estavam faltando. A cidade de Americana é uma delas! Vamos fazer :)

<h2>O nascimento da nossa Spider</h2>

![spider](https://media4.giphy.com/media/aUPfvs5MOXpxm/giphy.gif?cid=ecf05e47h79nh42rkbl6q7s78jivfxc7rme8brewvk8uutwe&rid=giphy.gif&ct=g){: style="float: left; margin-right: 1em;" class="img-responsive" height="40%" width="40%"}

Para criar nossa spider, basta usar o recurso do próprio Scrapy, ele já cria um modelo legal:

{% highlight bash %}
scrapy genspider sp_americana diariooficial.americana.sp.gov.br
{% endhighlight %}

Depois disso, você pode abrir o arquivo gerado com seu editor de texto preferido,
no meu caso, vou utilizar o [Vim]{:target="\_blank"}.

{% highlight bash %}
vim sp_americana.py
{% endhighlight %}


Você irá se deparar com um arquivo contendo uma configuração básica:

{% highlight python linenos %}
import scrapy


class SpAmericanaSpider(scrapy.Spider):
    name = 'sp_americana'
    allowed_domains = ['diariooficial.americana.sp.gov.br']
    start_urls = ['http://diariooficial.americana.sp.gov.br/']

    def parse(self, response):
        pass
{% endhighlight %}

<h3>Explicação</h3>
Linha 1: estamos importando a biblioteca scrapy;

Linha 4: é declarada a classe SpAmericanaSpider e entre parênteses é informado
que ela herda a classe _scrapy.Spider_;

Linha 5: Define o nome da nossa spider;

Linha 6: Define o domínio permitido, muito útil para garantir que nada sairá de controle;

Linha 7: Define onde nossa spider vai começar sua atuação;

Linha 9: Criado o método _parse_, o qual recebe o objeto _response_. O objeto response contem inclusive o
código fonte da página atual que a spider está;


<h2>Alterando o arquivo base</h2>

{% highlight python linenos %}
import dateparser
from scrapy import Spider
import datetime
from dateutil.relativedelta import relativedelta
from gazette.items import Gazette
import re


class SpAmericanaSpider(Spider):
    TERRITORY_ID = "3501608"
    name = "sp_americana"
    allowed_domains = ["diariooficial.americana.sp.gov.br"]
    start_date = datetime.date(2018, 9, 1)  # First gazette available
    start_urls = ["https://diariooficial.americana.sp.gov.br/diario-oficial-edicaoAnterior.php"]
    extra_editions = "https://diariooficial.americana.sp.gov.br/diario-oficial-edicaoExtra.php"

    locations = {
        "gazette": "//*[@class='day table-light']/strong/a",
        "extra_gazette_url": "//*[@class='list-group-item']/a/@href",
        "extra_gazette_date": "//*[@class='list-group-item']/text()"}
        
    def parse(self, response):
        pass
{% endhighlight %}
<h3>Explicação</h3>
Linha 1 a 6: Importa diversas bibliotecas que usaremos no nosso projeto, o dateparser é uma
ferramenta para criar objetos datetime, sendo o próprio datetime uma ferramenta para trabalhar
com datas, já a relativedelta nos ajuda a somar tempos cronologicamente e a biblioteca re a
filtrar padrões;

Linha 10: Definimos o número de identificação do território de Americana/SP;

Linha 13: Determinamos a data de início das publicações do diário oficial;

Linha 14: A [URL]{:target="\_blank"} de início foi alterada para a página de diários oficiais anteriores;

Linha 15: Determinamos o endereço onde os diários extraordinários estão publicados;

Linha 17: Criado uma estrutura de dicionário para organizar os locais onde nossa spider
irá interagir, isso facilita demais a manutenção e legibilidade do código!

<h2>Desenvolvendo o método parse</h2>

Depois de alterarmos nosso arquivo base, agora devemos criar o método parse,
o qual é o ponto de retorno padrão quando outros métodos não definem o _callback_,
por isso este acaba sendo responsável também por nutrir nossa classe com links!

{% highlight python linenos %}
def parse(self, response):

        param_url = f"?mes={self.start_date.month}&ano={self.start_date.year}"
        base_url = response.url
        url = base_url + param_url
        date = self.start_date

        while date.year < datetime.date.today().year or date.month < datetime.date.today().month:
            yield response.follow(url, self.parse_gazette)
            date = date + relativedelta(months=+1)
            url = base_url + f"?mes={date.month}&ano={date.year}"

        yield response.follow(self.extra_editions, self.extra_parse_gazette)
{% endhighlight %}

<h3>Explicação</h3>

Linha 3: Definimos os parâmetros da URL, é algo intríseco ao funcionamento do site
do diário oficial de Americana, por isso é importante navegar no site antes de
criar a sua spider;

Linha 4: Apenas copiei o conteúdo do start_urls, será útil para montarmos nossa URL;

Linha 5: Concatenamos _base\_url_ e _param\_url_ para formar nossa _url_;

Linha 6: apenas puxamos o _self.start\_date_ para a variável _date_ para ficar mais legível;

Linha 8: O que muda nessa URL? Apenas os parâmetros de mês e ano, então um laço
while da conta do recado! Enquanto o número do ano *ou* o número do mês iniciais forem menores do
que os meses e anos atuais, o laço irá funcionar (;

Linha 9: Usando o yield fazemos com que o comando seguinte entre para a fila de processamento,
enquanto ele acontece o laço continua, é a beleza do [processamento paralelo]{:target="\_blank"}.
O que vai para a fila, é o _response.follow_, que recebe dois parãmetros: a _url_ que montamos
anteriormente e também o método que será chamado, no caso _self.parse\_gazette_;

Linha 10: É realizada a soma das datas, acrescentando 1 mês;

Linha 11: Após somas as datas, criamos novamente a nossa URL;

Linha 13: Colocamos na fila mais um _response.follow_, dessa vez para acessar
a URL que tem os diários extraordinários, executando o método _self.extra\_parse\_gazette_;

<h2>Desenvolvendo o método parse_gazette</h2>

Depois de gerar todos os links que temos interesse no parse, agora podemos
criar métodos para explorá-los, extraindo as informações da página com os [xpaths]{:target="\_blank"}
definidos no _locations_:

{% highlight python linenos %}
    def parse_gazette(self, response):

        gazettes = response.xpath(self.locations["gazette"]).extract()
        
        for gazette in gazettes:
            file_url = gazette.split('"')[1]
            date = gazette.split('"')[7][6:16]
            date = dateparser.parse(date, date_formats=["%d/%m/%Y"]).date()
            edition = re.findall(r'\d+', gazette.split(';')[2])[0]

            yield Gazette(
                date=date,
                file_urls=[file_url],
                is_extra_edition=False,
                power="executive",
                edition_number=edition,

            )
{% endhighlight %}

<h3>Explicação</h3>

Linha 1: Definimos nosso método parse_gazette, ele recebe dois parâmetros, o self e o response.
O self pode parecer um mistério, mas pode-se dizer que é a nossa conexão com as informações
globais da classe, ou seja, todos os métodos e objetos desse escopo, o response recebe as requisições
feitas para o URL que geramos e o HTML da página está nele também;

Linha 3: O método xpath do response faz com que seja realizado a visualização da informação do
local especificado, o xpath é um dos muitos jeitos de dizer a localização de um elemento em uma página.
Depois utilizo o método extract, que vai me trazer todos os elementos em um [vetor (array)]{:target="\_blank"},
o que será muito útil para interagir com a informação coletada.

Linha 5: O laço for pega elemento por elemento do vetor

Linha 6: O elemento tratado da vez é recortado usando o método split, isso gera um novo
vetor, o qual me interessa a posição 1, onde temos o URL do arquivo PDF do diário oficial;

Linha 7: Usando a mesma tática anterior, chegamos à extração da data do diário oficial;

Linha 8: O padrão é a data no objeto datetime, então para isso usamos o dateparser, que trata
a informação e retorna um objeto de data para nós;

Linha 9: Usando um poucode regex, pegamos o número de edição do nosso diário oficial;

Linha 11: Damos início à criação do nosso objeto a ser inserido no projeto, estruturando
a informação no modelo do projeto, o yield adiciona isso à fila de processamento.


<h2>Desenvolvendo o método extra_parse_gazette</h2>

Depois de criarmos o nosso método para os diários oficiais, falta agora
os diários oficiais extraordinários, segue o código:

{% highlight python linenos %}
    def extra_parse_gazette(self, response):

        gazettes = response.xpath(self.locations['extra_gazette_url']).extract()
        dates = response.xpath(self.locations['extra_gazette_date']).extract()[2::3]
        counter = 0
        while counter < len(gazettes):
            file_url = gazettes[counter]
            date = dateparser.parse(dates[counter][2:12], date_formats=["%d/%m/%Y"]).date()
            counter += 1
            yield Gazette(
                date=date,
                file_urls=[file_url],
                is_extra_edition=True,
                power="executive",
            )
{% endhighlight %}

<h3>Explicação</h3>

Linha 3: Responsável por coletar o elemento que tem todos os diários extraordinários e fazer
um vetor com eles;

Linha 4: responsável por pegar o elemento que contêm todas as datas dos diários extraordinários
(pois é, eles estão em elementos diferentes);

Linha 5: Crio um contador para poder sincronizar a iteração entre os dois vetores

Linha 6: Enquanto o contador for menor que o tamanho do vetor, o laço executa;

Linha 7: Coleta o link do PDF direto do vetor gerado na linha 3;

Linha 8: Gera o objeto de datetime com a extração da data do vetor;

Linha 9: incrementa o contador

Linha 10: Coloca nosso objeto na fila para ser adicionado

<h2>Subindo o código</h2>

É necessário criar um novo branch ou fork (ramificação) do código

{% highlight bash %}
git checkout -b new_branch
{% endhighlight %}

Depois disso, adicionamos o repositório original:

{% highlight bash %}
git remote add upstream https://github.com/okfn-brasil/querido-diario
{% endhighlight %}

Damos início ao nosso commit:

{% highlight bash %}
git commit -m "Add spider Americana/SP"
{% endhighlight %}

Adicionamos o arquivo criado:
{% highlight bash %}
git add sp_americana.py
{% endhighlight %}

Subimos nosso código para o repositório:
{% highlight bash %}
git push -u origin new_branch
{% endhighlight %}

![comemoracao](https://media1.giphy.com/media/IwAZ6dvvvaTtdI8SD5/giphy.gif?cid=ecf05e47aahkyosu6docb8kkvbrgyabdymqwzt4j7mwukir9&rid=giphy.gif&ct=g#center){: style="display: block; margin: auto;" class="img-responsive" height="50%" width="50%"}

Depois de fazer tudo isso, ao entrar na página do GitHub, vai
aparecer o botão para realizar o Pull Request, isso encerra uma contribuição com o projeto.
Se você tiver interesse em contribuir ou quiser conversar sobre, basta abrir um issue na página
do GitHub do projeto.


<h2>Conclusões</h2>

O projeto do Querido Diário tem tudo para jogar luzes nas atitudes dos governos,
com certeza é algo que equilibra as relações de poder, a prova disso é que o projeto
já fez [deputado devolver o dinheiro]{:target="\_blank"}. O que precisamos é de sua ajuda,
espero que este breve relato sirva de trilha em sua saga :D

E aí, qual cidade você vai escolher para libertar?


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
[processamento paralelo]: https://lcsvillela.github.io/bash-na-velocidade-da-luz.html
[explicação teórica]: https://towardsdatascience.com/web-scraping-with-scrapy-theoretical-understanding-f8639a25d9cd
[xpath]: https://pt.wikipedia.org/wiki/XPath
[vetor (array)]: https://pt.wikipedia.org/wiki/Arranjo_(computa%C3%A7%C3%A3o)
[deputado devolver o dinheiro]: https://g1.globo.com/distrito-federal/noticia/apos-ser-flagrado-por-app-deputado-devolve-a-camara-r-727-por-13-refeicoes-no-mesmo-dia.ghtml
