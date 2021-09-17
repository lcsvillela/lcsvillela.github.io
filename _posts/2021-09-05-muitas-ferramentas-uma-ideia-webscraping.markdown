---
layout: post
title:  "Muitas ferramentas e uma ideia: Webscraping"
date:   2021-09-16 23:00:34 -0300
categories: python webscraping scrapy bash curl
description: A gente não usa motosserra para descascar maça, assim como não usa faca para cortar madeira
image: https://user-images.githubusercontent.com/23728459/133816523-42dc577a-ece7-4bf2-9fa2-160daf8b0334.jpeg
---


![ferramentas](https://media3.giphy.com/media/3oKIPqsXYcdjcBcXL2/giphy.gif?cid=ecf05e47e5jdeaqfllrxuhgmavywsfhcv7senamswlwipkmw&rid=giphy.gif&ct=g){: style="float: left; margin-right: 1em;" class="img-responsive" height="40%" width="40%"}

Quando estamos realizando uma tarefa, podemos ter à nossa disposição
diferentes tipos de ferramentas, muitas vezes caímos no erro de criar algum
tipo de afeto, algo que compromete a resolução do problema. Passamos a
adorar usar Selenium, Scrapy, Curl, Requests ou qualquer que seja a
ferramenta. É uma armadilha, não caia nessa, cada ferramenta tem um
propósito de existência, caso contrário não haveria comunidades e empresas
por trás delas.
A ideia aqui é falar um pouco sobre diferentes ferramentas que podem ser
usadas para realizar Webscraping, aquela prática super legal de extração
de dados da internet, que pode ser em massa ou não. As vezes você só deseja
saber se a informação do texto de uma página por exemplo foi alterado ou
não, ou saber se uma estrutura do site foi modificada, ou se existem
links quebrados no site...enfim.

<h2>Scrapy, a motosserra</h2>

![ferramentas](https://media2.giphy.com/media/SGV9O1fuh2nf5T8FNW/giphy.gif?cid=ecf05e47sshrwd7onioipplvubrh5rw2jdigtk1myhtyop90&rid=giphy.gif&ct=g){: style="float: left; margin-right: 1em;" class="img-responsive" height="40%" width="40%"}

Escrevi dois textos ([primeiro]{:target="\_blank"} e [segundo]{:target="\_blank"}) sobre Scrapy,
um sobre como construí um projeto para
baixar todas as frases de autores do site pensador.com (baixando 30 mil
frases) e outro sobre como contribuí para o projeto OpenSource Querido
Diário, onde automatizei o download de 800 diários oficiais da cidade de
Americana.
Nesses dois projetos, o grau de complexidade é médio, é necessário entrar
em muitas páginas diferentes, as quais muitas vezes nem é humanamente
possível fazer, visto que são milhares de páginas no caso do site pensadores
, onde fazer isso manualmente é impensável. Mas o ponto, é que o Scrapy não
utiliza um navegador, então não existe o tempo de renderização de uma página
e essa é uma diferença importante para com o Selenium. Então, temos aqui uma
afirmação importante: Se você não precisa por exemplo carregar um javascript
que vai gerar o HTML da página, você já tem um ponto a favor do Scrapy!
Outro ponto importante, é a complexidade do projeto, tempo é uma coisa
importante, então acredito que se você precisa navegar em mais de 10 páginas
...acredito que é considerável a utilização do Scrapy (apenas minha opinião,
mas é necessário analisar caso a caso), também é um ponto importante se
a velocidade é muito importante, visto que Scrapy trabalha naturalmente em
[processamento paralelo]{:target="\_blank"}.
É possível que a pergunta "Mas como eu faria login em uma página, como funciona isso afinal?",
nesse caso, é possível fazer login usando o POST,
e o Scrapy tem as ferramentas necessárias para relizar isso.

Podemos pegar o exemplo da página [Quotes to Scrape]{:target="\_blank"}, a qual
tem um mecanismo de login de usuário e senha, com auxílio de um token aleatório.
Então, nossa spider teria que ter algo mais ou menos assim:

{% highlight python %}
import scrapy
from scrapy import FormRequest

class QuotesToScrap(scrapy.Spider):
    name = 'QuotesToScrap'
    url = 'https://quotes.toscrape.com/login'
    start_url = [url]

    def parse(self, response):
        token = response.xpath("//*[@name='csrf_token']/@value").get()
        form = {
                'csrf_token': token,
                'name': 'lcsvillela',
                'password': 'lcsvillela'
        }

        yield FormRequest(url=self.url, formdata=form, callback=self.parse_page)

    def parse_page(self, response):
        #aqui você pode desenvolver o que deseja coletar estando logado!
{% endhighlight %}

Porém, não se iluda, é perfeitamente possível utilizar o Scrapy para
puxar informações de páginas que são carregadas utilizando javascript,
para isso existe a ferramenta [scrapy-splash]{:target="\_blank"}, que consegue renderizar a
página web que você deseja. O truque é bem legal, consiste basicamente
em criar um serviço web que consegue renderizar a página em javascript.


<h2>Selenium, o machado</h2>

![ferramentas](https://media2.giphy.com/media/l2SpUzCleC0Eb7Ow0/giphy.gif?cid=ecf05e47165906jdauj0ti3r5e13a9c47lc7xdzfz2leo8c3&rid=giphy.gif&ct=g){: style="float: left; margin-right: 1em;" class="img-responsive" height="40%" width="40%"}

O Selenium pode ser visto como uma espécie de machado quando pensamos
em extração de informação de páginas na rede, sua criação na realidade, foi
para a realização de testes (TDD por exemplo) em interfaces web. Imagina
poder testar se um elemento está clicável ou não, sensacional né? Porém,
pode ser usado para extração de informação ou automação de ações na internet,
como é [este caso que criei]{:target="\_blank"}, onde automatizei publicação de posts no
twitter. Então, quando queremos fazer testes de interface ou automatizar
alguma ação, o Selenium tem ponto à favor! Porém, o Selenium utiliza um
navegador, então ele faz a renderização de páginas web, ou seja: pode interagir
com um Javascript naturalmente! Essa é a chave do Selenium, ele facilita a interação
com as páginas web, para projetos pequenos ou onde é necessário simular
ação humana, ele é a ferramenta que irá servir melhor!


<h2>Requests, a espada</h2>

![ferramentas](https://media0.giphy.com/media/1hM7SKynfIzqtqXTNR/giphy.gif?cid=ecf05e478bq8uwqt94qw7jm2rxe4ph58y16jc16h0l8o5pjh&rid=giphy.gif&ct=g){: style="float: left; margin-right: 1em;" class="img-responsive" height="40%" width="40%"}


O requests no Python pode ser comparado à uma espada, por quê? Como
tem suporte à Orientação a Objetos, é algo que consegue fazer parte de
contruções mais complexas, porém se não fosse isso, seria uma faquinha
também, o funcionamento é algo muito parecido com o Curl, baseando-se
apenas em requisições HTTP, onde são executadas de forma sequencial.
Podemos usar processamento paralelo? Sim, mas é algo que pode se tornar
muito complexo e difícil de manter, então geralmente é só pra quebrar um
galho mesmo. Muitas vezes o requests é combinado com a ferramenta [BeautifulSoap]{:target="\_blank"},
que tem métodos bem interessantes sobre filtragem de dados. É uma
espécie de coleção de construções usando o requests e outras funcionalidades
da linguagem.

Segue um exemplo usando apenas o requests:

{% highlight python %}
import requests
import re

response = requests.get("https://www.folha.uol.com.br/")
response.text
for line in response.text.split(;):
    if re.search("c-main-headline__title", line):
        result = line
re.findall(r'c-main-headline__title">(.*?)<',result)
{% endhighlight %}

<h2>Curl, a faquinha</h2>


![ferramentas](https://media3.giphy.com/media/XxRfTUFDfNlN6/giphy.gif?cid=ecf05e47o3lm4h8q8ch1yck829gh619h8jo66sm9vmexieqk&rid=giphy.gif&ct=g){: style="float: left; margin-right: 1em;" class="img-responsive" height="40%" width="40%"}
O Curl é um comando do Linux, ele não faz parte da Builtin do Bash, porém
basta instalá-lo no sistema e você poderá fazer diversas requisições HTTP,
entre o Curl e o Scrapy, a principal diferença é que o Scrapy tem uma
estrutura complexa, onde a navegação, extração e tratamento dos dados (por
exemplo colocar os dados em um banco), já é completamente integrado. Além
disso, o Curl não faz a administração dos cookies automaticamente
(coisa que o Scrapy faz).

<h2>Conclusão</h2>

Não nutras sentimentos por nenhuma ferramenta, elas servem para propósitos
específicos e não correspondem seus sentimentos, além disso um comportamento
"purista" também não é saudável. Imagine o trabalho necessário para fazer um projeto
como os do Scrapy que citei, baixando quase 30000 frases do site pensadores utilizando
apenas o requests ou que seja o requests e o BeautifulSoap, é algo como
cortar a grama de um campo usando cortador de unha!

Portanto, construir projetos simples de webscraping com Curl é uma coisa
bem legal e rápida. Deixo aqui um exemplo que faz a mesma coisa do requests ali em cima:
{% highlight bash %}
curl -s https://www.folha.uol.com.br/ | fgrep -i "c-main-headline__title" | sed 's/.*title">//; s/<.*//'
{% endhighlight %}

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
