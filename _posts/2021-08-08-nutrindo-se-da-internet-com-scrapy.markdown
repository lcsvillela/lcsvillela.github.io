---
layout: post
title:  "Pacman de informação - Web Scraping com Scrapy"
date:   2021-08-08 22:00:34 -0300
categories: python webscraping
description: Não basta extrair os dados, é necessário dar um destino a eles! Automatizando publicações de frases legais e aleatórias no Twitter.
image: https://user-images.githubusercontent.com/23728459/128670187-0db991b0-d2b8-4b3b-bb9c-97b413fe0625.jpg
---
![ponteiro](https://user-images.githubusercontent.com/23728459/128670187-0db991b    0-d2b8-4b3b-bb9c-97b413fe0625.jpg){: style="float: left; margin-right: 1em;" cla    ss="img-responsive" height="60%" width="60%"}
Com a internet temos muita informação disponível, são
[terabytes]{:target="\_blank"}, ou melhor, [zettabytes]{:target="\_blank"}
de informação
geradas pelas pessoas no ano de 2021! O que fazemos é alimentar a matrix
com nossa informação, porém como a matrix faz para se alimentar dessa
quantidade absurda de informação?
[A extração de dados em massa]{:target="\_blank"} é um dos muitos jeitos de
alimentar a matrix, aqui vou utilizar o um site de frases que
já é muito conhecido, no caso o www.pensador.com, com cerca de 2 milhões de
frases de autores mais diversos(a ideia não é baixar todas aqui).
O que poderíamos fazer com isso?

Com as frases,
poderíamos por exemplo criar um perfil no twitter que publicasse uma frase 
aleatória a cada 10 minutos de um autor aleatório por exemplo. Seria um perfil
muito legal! Mas apesar de ser um projeto interessante, ainda é algo simples e
não reflete o potencial de aplicação possível. A Professora Solange Rezende
tem um ótimo artigo sobre [mineiração de dados e extração de
informações]{:target="\_blank"}, a aplicação disso é incrível, podemos
entender situações de forma rápida e utilizando estatística. Um exemplo é a
criação do [Manchetômetro]{:target="\_blank"}. Em suma, se você tem interesse em
inteligência artificial, não fique esperando alguém criar um dataset,
[crie o seu]{:target="\_blank"}!

<h2>Configurando o ambiente</h2>
*Veja para [Windows] ou [Mac OS]*

É necessário que você tenha o [Python]{:target="_blank"} instalado, além disso,
utilizaremos o framework Scrapy e Selenium, será o nosso pequeno megazord!
Para isso, precisamos instalar ambas as bibliotecas, sempre crie um ambiente
virtual antes de instalar qualquer coisa para isolar o sistema:

{% highlight bash %}
virtualenv --python=python3.7 venv
source venv/bin/activate
pip install scrapy selenium
{% endhighlight %}

Possivelmente será necessário você instalar o webdriver correspondente ao seu
navegador, no meu caso:

{% highlight bash %}
apt install firefoxdriver
{% endhighlight %}

Infelizmente, precisamos instalar o [Geckodriver]{:target="_blank"} de forma manual em alguns
sistemas, no caso, podemos executar em um Linux:
{% highlight bash linenos%}
wget https://github.com/mozilla/geckodriver/releases/download/v0.29.1/geckodriver-v0.29.1-linux64.tar.gz
tar xf geckodriver-v0.29.1-linux64.tar.gz
sudo mv geckodriver /usr/bin/
{% endhighlight %}

<h2>Criando o projeto com o Scrapy</h2>

Depois de instalar o Scrapy, podemos criar nosso projeto:

{% highlight bash %}
scrapy startproject frases
{% endhighlight %}

Isso resultará criação de uma estrutura de arquivos que utilizaremos, mas aqui,
vamos utilizar apenas três arquivos: pipeline.py, items.py e nossa spider.

<h2>Gerando a spider</h2>

Depois de gerar nosso projeto, podemos também gerar nossa spider, tudo isso pode
ser feito manualmente, mas a ideia do [framework]{:target="\_blank"} é acelerar nossa produção!

{% highlight bash %}
scrapy genspider Pensador www.pensador.com
{% endhighlight %}

<h2>Codificando o Pacman</h2>

<h3>Alterando os itens</h3>

Os itens definem a forma da sua informação, então a coleta de informações é feita na forma de itens individuais, altere o
arquivo para:

{% highlight python linenos %}
class FrasesItem(scrapy.Item):
    phrase = scrapy.Field()
    author = scrapy.Field()
    link = scrapy.Field()
{% endhighlight %}

<h3>Alterando a spider</h3>

Após gerar o arquivo da spider, precisamos abrir o arquivo da spider gerada, ele está em ./frases/frases/spiders/pensador.py, vamos abrí-lo com um editor de
textos qualquer e editar o arquivo deixando-o assim:

{% highlight python linenos %}
import scrapy
from frases.items import FrasesItem


class PensadorSpider(scrapy.Spider):
    name = 'pensador'
    allowed_domains = ['www.pensador.com']
    start_urls = ['https://www.pensador.com/autores/']
    url_base = 'https://www.pensador.com'
    locations = {'next_page': '//*[contains(text(), 'Próxima >')]/@href',
                 'phrases': '//*[@class="frase fr"]/text()',
                 'authors': '/html/body/div[1]/div[2]/div[1]/div[1]/div[3]/ul//a/@href'}


    def parse(self, response):
        authors = response.xpath(self.locations['authors']).extract()
        for author in authors:
            yield response.follow(f"{self.url_base}{author}", self.get_pages)

    def get_pages(self, response):

        try:
            next_page = response.xpath(locations['next_page']).extract()[0]
            author = response.url.split('/')[4]
            content = response.xpath(self.locations['phrases']).extract()
            yield FrasesItem(phrase=content,
                             author=author,
                             link=response.url)
            yield response.follow(f"{self.url_base}{next_page}", self.get_pages)
        except:
            author = response.url.split('/')[4]
            content = response.xpath(self.locations['phrases']).extract()
            yield FrasesItem(phrase=content,
                             author=author,
                             link=response.url)


{% endhighlight %}

<h4>Explicação</h4>

A linha 10, tem um dicionário das localizações necessárias, isso deixa o código mais legível e facilita
alterações futuras, afinal sites são alterados a todo tempo.
Na linha 14, a função parse é responsável por pegar as informações iniciais da página, elas nortearão nosso caminho site,
optei por pegar todos os autores existentes no site e depois disso basta chamar o método response.follow e passar
a função get_page e o links do autor, isso faz entrar em todas as páginas!
Na linha 22, já dentro da função get_page, temos a primeira linha um try, que vai literalmente tentar executar o bloco de
codigo dentro dela, caso dê algum erro, pula para o except. Dentro do bloco, a variável next_page pega o endereço da
próxima página, como quando a gente chega na última página de citações do autor, essa variável é nosso interruptor, uma
hora nevitavelmente irá dar erro, pois não terá próxima página e então, vai para o except para finalizar.
Observe que a linha 29 faz uma chamada recursiva da função, passando o link da próxima página de citação.

As linhas onde temos yield FrasesItem, chama a configuração de itens, além disso nos possiblita salvar todas as informações
de forma ordenada, em um arquivo [JSON]{:target="\_blank"}.

<h3>O pulo do gato</h3>

Legal, com isso já é possível perambular por todas as frases de todos os autores, mas como faz para salvar um arquivo e
nos livrar da necessidade de ficar consumindo nossa banda toda hora com isso? Tem dois jeitos de fazer isso, podemos usar
a função do próprio framework que cria um JSON padrão, possível de ser lido em qualquer programa de extração:

{% highlight python %}
scrapy runspider pensador.py -o arquivo.json
{% endhighlight %}

Ou podemos usar o arquivo pipelines.py que é responsável por programar o que acontecerá concomitante à coleta,
ou seja, um processo paralelo será executado e isso faz o seu programa voar,
(veja [um pouco sobre isso em bash aqui]{:target="\_blank"}), mas podemos fazer outras coisas usando o pipeline
como por exemplo redirecionar para uma base de dados, mas para simplificar colocaremos tudo em um arquivo json. No arquivo,
altere-o para:

{% highlight python linenos %}
from itemadapter import ItemAdapter
import json


class FrasesPipeline:
    def process_item(self, item, spider):
        line = json.dumps(ItemAdapter(item).asdict(), ensure_ascii=False)
        self.file.write(line)
        return item

    def open_spider(self, spider):
        self.file = open("arquivo.json", 'w')

    def close_spider(self, spider):
        self.file.close()
{% endhighlight %}

<h4>Explicação</h4>
As funções open_spider e close_spider são executadas quando a spider inicia e finaliza um processo de coleta, já a
process_item é responsável por processar os dados extraídos, no caso os itens são transformados primeiro em uma estrutura
de dicionário e depois em um JSON para então serem escritos no arquivo.json

<h4>Informações dos dados coletados no dia 8 de agosto de 2021</h4>

Cerca de 50 autores e 30000 frases foram coletadas

<h2>Utilidade dos dados coletados</h2>

Muitas coisas podem ser feitas sobre os dados coletados, como já dito anteriormente, porém acho que seria interessante
publicar frases de alguns autores no twitter, uma quantidade grande de frases de alguns autores para diminuir o risco de
serem duplicadas e além disso, não colocar o nome do autor na frase (sempre suspeito das fontes).

Utilizando [esta biblioteca que criei]{:target="\_blank"}, podemos criar um novo código, fazendo reúso da ferramenta
anteriormente construída.

primeiro clone o repositório:

{% highlight bash %}
git clone https://github.com/lcsvillela/twitterbot
{% endhighlight %}

Depois de clonar os repositórios, você pode importar a biblioteca e desenvolver em cima.

{% highlight python linenos %}

from twitterbot import twitterbot as twitter
from time import sleep
import random
import json


max_tweet_length = 280
user = "seu nome de usuario"
password = "sua senha"


def maketweet():
    t = twitter(user, password)
    t.login()
    counter = 0

    while counter < 20:
            counter += 1
            text = get_phrase()
            t.set_text(text)
            t.make_tweet()
            sleep(60)
    t.close()


def get_phrase():
    length = 281
    while length > max_tweet_length:
        archive = open("arquivo.json", "r")
        text = json.load(archive)
        item = random.choice(text)
        phrase = item['phrase']
        author = item['author'].replace("_", " ")
        phrase = random.choice(phrase).replace("\n", " ")
        length = len(phrase)
        print(phrase)
    return f'"{phrase}" - {author}'


maketweet()

{% endhighlight %}


<h4>Explicação</h4>

O código utiliza a biblioteca que criei anteriormente, então a gente pode
abstrair a questão de como o programa faz as ações e simplesmente mandar
fazer, além de ser mais objetivo e garantir o isolamento de código.

Da linha 12 a linha 23, temos a instância da classe twitterbot criada na
variável t, sendo assim podemos aplicar os métodos desejados, no caso
t.login(user, password) para realizar o login, t.set_text(phrase) para definir
uma nova frase.

Já na função get_phrase, é carregado o arquivo contendo as informações no
formato json e depois podemos sortear uma frase qualquer de um autor qualquer,
usando o random.choice, que escolhe um item qualquer de um vetor.

<h2>Conclusão</h2>

Com certeza o scrapy é um dos frameworks mais úteis para a tarefa árdua de
extrair informações, porém também é necessário pensar o que faremos com ela, no
caso é importante juntarmos outros frameworks existentes, de juntar o Scrapy com o uso do Selenium possibilita enviar informações de um lugar para o outro que
no caso é o Twitter, mas poderia ser perfeitamente uma API que retorna frases
aleatórias.


[JSON]: https://pt.wikipedia.org/wiki/JSON
[um pouco sobre isso em bash aqui]: https://lcsvillela.github.io/bash-na-velocidade-da-luz.html
[Manchetômetro]: http://manchetometro.com.br
[mineiração de dados e extração de informações]: https://www.alice.cnptia.embrapa.br/bitstream/doc/895476/1/FSMA.pdf
[A extração de dados em massa]: https://www.terra.com.br/noticias/empresas-utilizam-big-data-para-obtencao-de-informacoes-sobre-negocios-e-ter-impacto-positivo-no-mercado,d07f7431108cfdb4789f76fe4fb3bb82e21j1qy9.html
[zettabytes]: https://pt.wikipedia.org/wiki/Zettabyte
[terabytes]: https://pt.wikipedia.org/wiki/Terabyte
[crie o seu]: https://towardsdatascience.com/using-scrapy-to-build-your-own-dataset-64ea2d7d4673
[Scrapy]: https://medium.com/@marlessonsantana/utilizando-o-scrapy-do-python-para-monitoramento-em-sites-de-not%C3%ADcias-web-crawler-ebdf7f1e4966
[Selenium]: https://selenium-python.readthedocs.io/
[framework]: http://www.dsc.ufcg.edu.br/~jacques/cursos/map/html/frame/oque.htm
[Python]: https://www.python.org/downloads/
[esta biblioteca que criei]: https://github.com/lcsvillela/MakeTweetPy/blob/main/MakeTweet.py
[Mac OS]: https://medium.com/dropout-analytics/selenium-and-geckodriver-on-mac-b411dbfe61bc
[Windows]: https://medium.com/ananoterminal/ambientar-selenium-no-windows-3b880fa0e827
