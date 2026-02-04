---
layout: post
title:  "Filmes de Porto Alegre no Cinema em Poa com BotCity"
date:   2026-02-04 03:00:34 -0300
categories: bots, scrapping
description: Criando um bot no Telegram para saber quais filmes estão passando com a BotCity
image: 
thumbnail: 
---

<h2>Por que a BotCity?</h2>

A [BotCity]{:target="\_blank"} é uma empresa brasileira, que teve a ideia de criar um
framework que junta diversos tipos de automações em apenas um framework, mas mais que isso,
oferece um gerenciador gráfico e web para que seja possível criar suas
automações e tem soluções que facilitam o monitoramento e gerenciamento das automações usadas.

Isso facilita muito para gerenciar um projeto, pois atualizar um projeto é sempre algo trabalhoso e no caso a
preocupação se torna apenas os pacotes da BotCity.

No caso, vamos usar para verificar os filmes que estão sendo divulgados no site do [Cinema em POA]{:target="\_blank"}.

[0 - Preparando o ambiente](#0) \\
[1 - Código Completo](#0) \\
[2 - Como executar pela BotCity?](#1)\\
[3 - Conclusão](#6)

<h2 id="0">0 - Preparando o ambiente</h2>

Antes de tudo, precisamos criar nosso ambiente virtual, utilizando o comando virtualenv e então ativá-lo:

{% highlight None %}
virtualenv --python=3 venv
source venv
{% endhighlight %}

Depois disso, podemos clonar o repositório onde o código está:

{% highlight None %}
git clone https://github.com/lcsvillela/cinema-em-poa-botcity.git
{% endhighlight %}

E então instalar no nosso ambiente venv as dependências:

{% highlight None %}
pip install -r requirements.txt
{% endhighlight %}

<h2 id="1">1 - Análise do Código Completo</h2>

No trecho abaixo importamos as bibliotecas que iremos utilizar:

{% highlight Python linenos %}
from botcity.plugins.http import BotHttpPlugin
from botcity.plugins.telegram import BotTelegramPlugin
from bs4 import BeautifulSoup as bs
from requests.exceptions import ConnectionError
from datetime import datetime
import os
from botcity.maestro import *
from time import sleep
{% endhighlight %}

No código abaixo, criamos uma classe chamada CinemaEmPoa, que tem a sua inicialização definida com três
parâmetros, o self que é um objeto que é acessível em toda a classe, a url que iremos utilizar e o token
de acesso do Telegram (obviamente este token não funciona, use o seu).

Aproveitamos para declarar as variáveis que precisaremos em algum momento utilizando o self para que sejam
globais e também executamos métodos quando a classe é instanciada para que seja executado o bot. Podemos
observar a linha 14 extraindo os dados do site do Cinema em Poa, na 15 o método que cria a estrutura da mensagem
e por fim o início da execução completa com o método start.

{% highlight Python linenos %}
class CinemaEmPoa:
    def __init__(
        self,
        url="https://cinemaempoa.com.br/program",
        token="8123421335:ABFwVGLoyMxTpPXWTtG7Ugl136L\_1W8s9wE",
    ):
        self.http = BotHttpPlugin(url)
        self.telegram = telegram = BotTelegramPlugin(token=token)
        self.page = None
        self.message\_films = None
        self.films = {}
        self.last\_time = None
        self.last\_update\_id = self.load\_offset()
        self.get\_data()
        self.create\_message()
        self.start()
{% endhighlight %}

O telegram não faz o controle da fila de requisições que o bot recebe, então em alguns frameworks precisamos
nós mesmos fazer o controle, salvando o último ID processado em um arquivo, no caso o last\_id.data:

{% highlight Python linenos %}
    def save\_offset(self, offset):
        with open("last\_id.data", "w") as f:
            f.write(str(offset))

    def load_offset(self):
        if os.path.exists("last_id.data"):
            with open("last_id.data", "r") as f:
                return int(f.read())
        return None
{% endhighlight %}

Aqui apenas pegamos o código da página e colocamos no parser do BeautifulSoup para
facilitar a extração futura dos dados:

{% highlight Python linenos %}
    def get\_page(self):
        try:
            source = self.http.get().content
            self.page = bs(source, "html.parser")
        except ConnectionError:
            print("error: connection")
{% endhighlight %}

Quando abrimos a página do Cinema em POA e vamos na programação, vemos que temos a estrutura de
datas e depois filmes do dia, então a ideia aqui é basicamente extrair os blocos que englobam a data e os
filmes relacionados naquela data e então iterar com um for para conseguir estruturar
a informação em dois níveis: um dicionário tem a chave da data e ele então armazena
um vetor de dicionários que tem as informações de cada filme.

{% highlight Python linenos %}
    def get\_films(self):
        _dates = self.page.find_all(class_="col-md-8")

        for _date in _dates:
            _date_text = _date.find(class_="mb-2 fs-5").getText()
            self.films[f"{_date_text}"] = []
            _films = _date.find_all(class_="mb-1")
            for _film in _films:
                _location = _film.getText().strip().split(" ")[-1]
                _title_film = _film.getText().strip().split(" ")[1:-1]
                _hour = _film.getText().strip().split(":")[0]
                _title_film = " ".join(_title_film)
                self.films[f"{_date_text}"].append(
                    {"location": _location, "title": _title_film, "hour": _hour}
                )
{% endhighlight %}

Como temos a informação dos filmes estruturada, podemos agora organizá-la do jeito que queremos,
usando o join podemos juntar todas as strings que temos e ter uma mensagem para o usuário.

{% highlight Python linenos %}
    def create\_message(self):

        self.message_films = ""
        for films in self.films.keys():
            for film in self.films[f"{films}"]:
                location = film["location"]
                title = film["title"]
                hour = film["hour"]

                self.message_films += " ".join(
                    [films, "-", hour, "-", title, "-", location, "\n"]
                )
{% endhighlight %}

Aqui temos a execução da get\_data, que ao fim da execução, armazena a data de sua execução, que utilizaremos
para economizar nossa banda.

Na execução do método bot\_telegram, temos na linha 8 o comando que verifica as novas requisições e utiliza
o ID que salvamos anteriormente para que não fique repetindo as requisições anteriores.

Na linha 10 verificamos se temos alguma requisição, caso contrário retornamos ao ponto de chamada e não
é executado o resto do código, na linha 13, temos um for que iterará por todas as requisições, verificando
o seu conteúdo e casando as strings de comando que foram especificadas.

{% highlight Python linenos %}
    def get\_data(self):
        self.get\_page()
        self.get\_films()
        self.last\_time = datetime.now()

    def bot_telegram(self):

        updates = self.telegram.bot.get_updates(offset=self.last_update_id)

        if len(updates) == 0:
            return

        for update in updates:
            message = update.message
            chat_id = message.chat.id
            text = message.text

            if text == "/start":
                self.telegram.bot.send_message(
                    chat_id,
                    "Este é o bot do Cinema em POA feito com o BotCity! Agora você pode ver todos os filmes que passarão na cidade :D\n/filmes - para ver a lista completa de filmes\n/ajuda - veja o que este bot pode fazer!\n\n Mais informações em www.lcsvillela.com",
                )

            elif text == "/status":
                self.telegram.bot.send_message(
                    chat_id, "O sistema está rodando perfeitamente! 🚀"
                )

            elif text == "/ajuda":
                self.telegram.bot.send_message(
                    chat_id, "Use /filmes para ver todos os filmes que estão passando!"
                )

            elif text == "/filmes":
                self.telegram.bot.send_message(chat_id, self.message_films)

            self.last_update_id = update.update_id + 1
            self.save_offset(self.last_update_id)
{% endhighlight %}

O inicializador tem este método, que tem um loop indefinido e que verifica se faz mais de um dia que
o site do Cinema em POA foi extraído, executa sempre o método do bot do telegram e espera um segundo para o
próximo loop.

{% highlight Python linenos %}
    def start(self):

        while True:
            if (datetime.now() - self.last_time).days:
                self.get_data()
                self.create_message()
            self.bot_telegram()
            sleep(1)
{% endhighlight %}

<h2 id="2">2 - Como Executar pela BotCity? </h2>

<iframe width="705" height="301" src="https://www.youtube.com/embed/IsFYU6Kfv5g" title="Telegram bot do Cinema em POA com BotCity" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

<h2 id="3">3 - Conclusão</h2>

A BotCity é uma plataforma completa, que oferece diversos tipos de RPA possíveis em apenas um framework e
possibilita o gerenciamento e monitoramento, de forma intuitiva para que usuários possam executar automações
existentes. Para mais informações, veja a [documentação da BotCity]{:target="\_blank"}

[documentação da BotCity]: https://documentation.botcity.dev/pt/
[Cinema em POA]: https://cinemaempoa.com.br/
[BotCity]: https://www.botcity.dev/br
[se você tiver brio]: https://www.youtube.com/watch?v=TRPBY_lxJfE
[pesquisadores do Vale do Silício]: https://www.bbc.com/portuguese/geral-48586734
[esse estudo do MIT]: https://youtu.be/C38xlWnkezQ?t=281
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
