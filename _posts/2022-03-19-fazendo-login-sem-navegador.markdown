---
layout: post
title:  "Acessando uma conta em um site sem navegador"
date:   2022-03-19 01:00:34 -0300
categories: python scrapy requests
description: Muitas vezes existe uma camada de segurança em sites ou mesmo em dispositivos locais, faça a autenticação e seja feliz!
image: https://user-images.githubusercontent.com/23728459/159110003-31f82a83-843b-4249-95f6-7e0598bf4c6a.png 
thumbnail: https://user-images.githubusercontent.com/23728459/159109781-13740661-9255-4ebf-891b-e492b86ff91a.png
---

<h2>Fazendo login com o Requests</h2>

<h3>Logando no site do restaurante da universidade</h3>
Para interagir com sistemas web, precisamos entender como os mesmos funcionam, o navegador
que usamos utiliza o protocolo [HTTP]{:target="\_blank"} para se comunicar, é interessante que sentemos em uma
cadeira e estudemos o funcionamento do protocolo, porém...tudo é requisição e resposta nesse
tipo de comunicação com alguns tipos de ação/verbos, como GET, DELETE, PUT e POST.
Para fazermos o login em um site, precisamos de uma sessão, para isso é gerado um [cookie]{:target="\_blank"},
que nada mais é que uma sequência de caracteres que identifica nossa sessão de forma única
e serve para registrar e fazer nossa identificação online.
Para gerenciar uma sessão, devemos portanto ter um cookie e o Requests vem com um método
incluso para fazer isso, muitas vezes é necessário ter uma sessão em um site mesmo sem
usuário e senha.

Segue um exemplo usando requests para fazer o login e extrair as informações de um usuário
do site do SRU (Sistema de Restaurante Universitário) da UNESP/IBILCE:

{% highlight python linenos%}

import requests
import re
import sys

class SRU:
    def __init__(self, username, password):
        self.__username = username
        self.__password = password
        self.__session = requests.Session()
        self.__data = {}
        self.start()

    def start(self):
        self.login()
        self.get_id()
        self.get_informations()
        _data = self.get_data()
        print(_data)

    def login(self):
        response = self.__session.get("https://sru.ibilce.unesp.br/login")
        csrf = re.findall(r'name="_csrf" value=".*"', response.text)[1].split('"')[3]
        self.__session.post("https://sru.ibilce.unesp.br/login", data={"username": self.__username, "password": self.__password, "_csrf": csrf})
        
    def get_id(self):
        response = self.__session.get("https://sru.ibilce.unesp.br/cliente/buscar-usuario-logado")
        ID = response.json()["idUsuario"]
        self.__data["ID"] = ID

    def get_informations(self):
        response = self.__session.get("https://sru.ibilce.unesp.br/usuario/dados-pessoais")
        self.__data["name"] =  re.findall(r'placeholder="Insira seu nome" required="true" type="text" value=".*"/>', response.text)[0].split('"')[7]
        self.__data["email"] = re.findall(r'placeholder="Insira seu e-mail" type="email" required="true" value=".*"/>', response.text)[0].split('"')[7]
        self.__data["cpf"] = re.findall(r'name="cpf" class="form-control" readonly="readonly" type="text" value=".*"/>', response.text)[0].split('"')[9]

    def get_data(self):
        return self.__data

SRU(sys.argv[1], sys.argv[2])

{% endhighlight %}

* Da linha 1 a 3, estamos importando as bilbiotecas utilizadas no nosso programa
* Na Linha 5, estamos declarando uma classe, se quiser saber mais sobre, leia um
pouco mais sobre [como programar com classe]
* Da linha 6 a 11, estamos utilizando um método especial do python, o \_\_init\_\_ que é executado
quando a classe é instanciada. Nele estamos definindo os atributos de nosso programa, no caso,
senha, nome de usuário, nossa sessão web e um dicionário para armazenar os nossos dados
* Linha 13 a 18 defimos nosso método principal, que fará todas as tarefas que queremos
* Linha 20 a 23 fazemos o método login, responsável por enviar nossas informações para o site,
imitando o comportamento realizado quando utilizamos o navegador. Isso pode ser descoberto verificando
as requisições feitas apertando o botão F12 no seu teclado e indo na parte de Rede ao acessar qualquer site.
Note que não apenas usuário e senha são enviados para o servidor, mas também uma sequência de caracteres que
funciona como validador de uma sessão. Essa sequência sempre aparece ao abrirmos a página em seu código fonte e
vários sites utilizam coisas desse gênero
* Linha 25 a 28 nos comunicamos com o site para extrair o ID de usuário, é o número que muitos utilizam
para poder se identificar no restaurante universitário
* Linha 30 a 34 podemos também extrair todos os dados do nosso usuário
* Linha 36 e 37 serve para retornar as informações extraídas
* Linha 39 estancia a classe e faz a passagem de parâmetros via linha de comando

Para executar esse programa, basta fazer python nome_arquivo.py usuário senha, ele te retornará algo como:

{'ID': 666, 'name': 'Lucas Villela', 'email': 'lucasvillela@email.br', 'cpf': '6666666666'}

Isso, além de tudo, nos prova que estávamos logados no site e podemos fazer o que bem entender nele ;)

<h3>Logando no Quotes to Scrap</h3>

O site quotes to scrap é um local com várias frases legais, criado para servir de exemplo para
diversos web scraping e também para publicar frases (Óbvio). Quando entramos no
[Quotes to Scrap]{:target="\_blank"} temos uma página de login, basta enviar os dados e ser feliz.

Podemos logar facilmente utilizando Requests, depois de investigar um pouco o funcionamento do site:


{% highlight python linenos%}
import requests
import re
import sys

class Quotes:
    def __init__(self, username="lcsvillela", password="lcsvillela"):
        self.__username = username
        self.__password = password
        self.__session = requests.Session()
        self.start()

    def login(self):
        response = self.__session.get("https://quotes.toscrape.com/login")
        csrf = re.findall(r'name="csrf_token" value=".*"/>', response.text)[0].split('"')[3]
        self.__session.post("https://quotes.toscrape.com/login", data={"username": self.__username, "password": self.__password, "csrf_token": csrf})

    def get_info(self):
        response = self.__session.get("https://quotes.toscrape.com")
        re.findall(r'<a href="[^"]*">\(Goodreads page\)</a>',response.text)

    def start(self):
        self.login()
        self.get_info()
        
        
Quotes()
{% endhighlight %}

Para provar que nosso programa está logado no site, vamos extrair os links
do GoodReads page (só aparecem quando estamos logados),
ao executar o programa os links são exibidos.

<h2>Fazendo login com scrapy</h2>

Já fiz vários textos por aqui falando sobre o Scrapy, como extraí [30000 frases]{:target="\_blank"},
mas sobre o login, basta fazermos as requisições assim como o Requests faz, porém no Scrapy:

{% highlight python linenos%}
from scrapy import Spider
from scrapy.http import FormRequest
from scrapy.utils.response import open_in_browser


class Quotes(Spider):
    name = 'quotes'
    start_urls = ('http://quotes.toscrape.com/login',)

    def parse(self, response):
        token = response.xpath('//*[@name="csrf_token"]/@value').get()
        return FormRequest.from_response(response,
                                         formdata={'csrf_token': token,
                                                   'password': 'lcsvillela',
                                                   'username': 'lcsvillela'},
                                         callback=self.parse_info)

    def parse_info(self, response):
        pass
{% endhighlight %}

<h3>Conclusão</h3>

Tudo é HTTP e podemos fingir que somos um navegador, existem sites que inviabilizam o uso
dessas ferramentas pelo tempo de trabalho, mas sempre é possível fazer algo do gênero. Até mesmo
usar requests para passar por cima do RecapchaV2 do Google! 


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
[município de Rio Claro]: https://www.rioclaro.sp.gov.br/diariooficial/index.php?acao=go
[30000 frases]: https://lcsvillela.github.io/nutrindo-se-da-internet-com-scrapy.html
[Web Scraping]: https://lcsvillela.github.io/muitas-ferramentas-uma-ideia-webscraping.html
[Americana SP]: https://lcsvillela.github.io/querido-diario-monitorando-governo-com-scrapy.html
[Rio Claro]: https://pt.wikipedia.org/wiki/Rio_Claro
[HTTP]: https://pt.wikipedia.org/wiki/Hypertext_Transfer_Protocol
[Quotes to Scrap]: https://quotes.toscrape.com/
[cookie]: https://pt.wikipedia.org/wiki/Cookie_(inform%C3%A1tica)
