---
layout: post
title:  "Querido Diário - Monitorando o governo de Araraquara/SP"
date:   2022-08-16 01:00:34 -0300
categories: python webscraping scrapy querido-diario
description: Aqui descrevo como contribuí para o projeto Querido Diário, o qual tem o propósito de baixar todos os diários oficiais do Brasil, sendo um ótimo meio de monitorar ações do governo
image: https://user-images.githubusercontent.com/23728459/184834208-b979fac0-2267-4e0b-b194-e0234c5ac71b.jpg 
---

Um tempo atrás, escrevi um pouco sobre como tinha realizado uma
contribuição para coletar os [diários oficiais de Americana]{:target="\_blank"}.
Porém, tem alguns erros e imprecisões por lá e recomendo a leitura
apenas para caráter informativo. Se você quiser seguir algo, este está
com certeza melhor e é para a [cidade de Araraquara]{:target="\_blank"}!

<h2>Preparando o terreno</h2>

<h3>Fork do projeto</h3>

Entre no seu github e clique para [fazer fork]{:target="\_blank"} do projeto,
depois disso acesse seu perfil e copie o link do seu fork, deve estar
algo assim:

{% highlight bash %}
https://github.com/SEU_NOME_DE_USUARIO/querido-diario
{% endhighlight %}

<h3>Clonando o projeto</h3>

É necessário [instalar o git]{:target="\_blank"} antes!
Depois de realizar o fork, agora podemos puxar os dados para a nossa máquina,
para isso use o comando:

{% highlight bash %}
git clone https://github.com/SEU_NOME_DE_USUARIO/querido-diario
{% endhighlight %}

Depois disso, o projeto foi clonado para seu computador, sendo assim, temos
boa parte do ambiente necessário.

Agora, vamos criar um ramo dentro do nosso projeto, para que possamos separar
o código e não fazer bagunça, para isso usamos o comando:

{% highlight bash %}
git checkout -b spider-to-palmas
{% endhighlight %}

Deste modo, estamos criando um ramo que terá o nosso código para a spider que
queremos criar, tendo como base o código no ramo principal. Para listar todos
os ramos e ver onde estamos:

{% highlight bash %}
git branch -a
{% endhighlight %}

<h3>Configurando Python</h3>

É necessário [instalar o Python no Windows]! Na maioria das distribuições
Linux, vem instalado por padrão.
Precisamos criar nosso ambiente virtual no python, para garantir que
nosso sistema operacional não seja afetado pelas bibliotecas que vamos instalar:

{% highlight bash %}
pip3 install virtualenv
virtualenv --python=python3.9 venv
source venv/bin/activate
{% endhighlight %}

Agora com nosso ambiente criado e ativo (deve ter aparecido venv do lado
esquerdo no seu terminal), agora podemos instalar os requerimentos do projeto,
navegue até o arquivo _requirements.txt_.

{% highlight bash %}
pip install -r requirements.txt
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
serão baixados no seu computador, hoje (16 de agosto de 2022) ela está funcinal.
Mas além de executar, a ideia é criar uma spider nova, então notei no arquivo
CITIES.md, que tem todas as cidades e as spiders existentes, que faltam muitas
cidades. Peguei uma lista das 200 maiores cidades do Estado de SP e filtrei as
que estavam faltando. A cidade de Americana é uma delas! Vamos fazer :)

<h2>Criando a spider</h2>

![spider](https://media4.giphy.com/media/aUPfvs5MOXpxm/giphy.gif?cid=ecf05e47h79nh42rkbl6q7s78jivfxc7rme8brewvk8uutwe&rid=giphy.gif&ct=g){: style="float: left; margin-right: 1em;" class="img-responsive" height="40%" width="40%"}

Para criar nossa spider, basta usar o recurso do próprio Scrapy, ele já cria um modelo legal:

{% highlight bash %}
scrapy genspider sp_araraquara diariooficial.americana.sp.gov.br
{% endhighlight %}

Depois disso, você pode abrir o arquivo gerado com seu editor de texto preferido,
no meu caso, vou utilizar o [Vim]{:target="\_blank"}.

{% highlight bash %}
vim sp_araraquara.py
{% endhighlight %}


Você irá se deparar com um arquivo contendo uma configuração básica, vamos
alterá-lo um pouco.

{% highlight python linenos %}
import datetime
from urllib.parse import urlencode

import dateparser

from gazette.items import Gazette
from gazette.spiders.base import BaseGazetteSpider


class SpAraraquaraSpider(BaseGazetteSpider):
    TERRITORY_ID = "3503208"
    name = "sp_araraquara"
    allowed_domains = ["diariooficialcmararaquara.sp.gov.br"]
    start_date = datetime.date(2021, 3, 4)  # First gazette available
    end_date = datetime.datetime.today()
    start_urls = ["https://www.diariooficialcmararaquara.sp.gov.br"]
    search_url = "https://www.diariooficialcmararaquara.sp.gov.br/diario/busca?"
    base_url = "https://www.diariooficialcmararaquara.sp.gov.br"

    def parse(self, response):
        pass

{% endhighlight %}

<h3>Explicação</h3>
Linha 1 a 7: estamos importando diversas bibliotecas, referentes à
manipulação de hora, bibliotecas com recursos padronizados para utilizar
o Scrapy no projeto e outras;

Linha 10: é declarada a classe SpAraraquaraSpider e entre parênteses é informado
que ela herda a classe _BaseGazetteSpider_;

Linhas 11: Define o código IBGE da cidade

Linha 12: Define o nome da nossa spider;

Linha 13: Define o domínio permitido, muito útil para garantir que nada sairá de controle;

Linhas 14 e 15: Configura o primeiro dia que foi publicado diário oficial da
cidade e depois a data atual

Linha 16: Define onde nossa spider vai começar sua atuação;

Linha 17 e 18: Determina a URL que é realizada a pesquisa do diário e uma base
para realizarmos os downloads

Linha 20: Criado o método _parse_, o qual recebe o objeto _response_. O objeto response contem inclusive o
código fonte da página atual que a spider está;


<h2>Criando a Spider</h2>

<h3>Investigação</h3>

Acesse o site dos [diários de Araraquara]{:target="\_blank"}
e pressione F12, vai aparecer
um painel e lá tem uma seção de Rede, onde é possível ver (quase) todas
as mensagem que seu navegador trocou com o site alvo.

![imagem1](https://user-images.githubusercontent.com/23728459/184828319-7fa3534f-97ca-4ffd-97c2-b89cb9e267ad.png){: style="float: center; margin-right: 1em;" class="img-responsive" height="100%" width="100%"}

No formulário à esquerda, tem como filtrar por datas, após colocar a data,
clique em buscar. Podemos encontrar esta requisição:

![imagem1](https://user-images.githubusercontent.com/23728459/184828635-f4232042-b662-497f-80c8-ee799c04e961.png){: style="float: center; margin-right: 1em;" class="img-responsive" height="100%" width="100%"}

Assim descobrimos que podemos interagir diretamente com a URL!

<h2>Desenvolvendo o método parse</h2>

Depois de alterarmos nosso arquivo base, agora devemos criar o método parse,
o qual é o ponto de retorno padrão quando outros métodos não definem o _callback_,
por isso este acaba sendo responsável também por nutrir nossa classe com links!

{% highlight python linenos %}
def parse(self, response):
        initial = self.start_date.strftime("%d/%m/%Y")
        end = self.end_date.strftime("%d/%m/%Y")

        params = {
            "data": initial,
            "dataFinal": end,
            "descricao": "",
            "subcategoria": "",
        }
        url_params = urlencode(params)
        yield response.follow(f"{self.search_url}{url_params}", self.parse_gazette)

{% endhighlight %}
<h3>Explicação</h3>

Linha 2 e 3: Definimos as datas de início e fim no formato de string; 

Linha 5: Configuramos os parâmetros que o site passa via requisição na _URL_; 

Linha 11: codificamos as informações para serem passadas na _URL_;

Linha 12: Usando o yield fazemos com que o comando seguinte entre para a
fila de processamento e chama o método parse_gazette

<h2>Desenvolvendo o método parse_gazette</h2>

Depois de realizar a requisição com os parâmetros que passamos via URL,
agora vamos extrair os dados que retornaram e salvar os diários.

{% highlight python linenos %}
def parse_gazette(self, response):

        gazettes = response.css(".event-card.animated.flipInX")

        for gazette in gazettes:
            card = gazette.css(".event-card")

            edition_number = card.css(".event-data h4 ::text").re_first(r"[0-9]+")
            date = card.re_first(r"[0-9]+/[0-9]+/[0-9]+")
            date = datetime.datetime.strptime(date, "%d/%m/%Y").date()
            url = card.css(".row ::attr(href)").get()
            url = self.base_url + url
            if card.css(".event-edicao p ::text").get() == "Edição Única":
                extra_edition = False
            else:
                extra_edition = True

            yield Gazette(
                date=date,
                file_urls=[url],
                is_extra_edition=extra_edition,
                power="executive",
                edition_number=edition_number,
            )
{% endhighlight %}

<h3>Explicação</h3>
Linha 3: com este comando, extraímos o elemento da página Web utilizando os
seletores de CSS, utilizamos o nome da classe do objeto para isso.
Para encontrar essa informação, basta usar o botão F12 e procurar o elemento
no código HTML e então jogamos essa informação na variável _card_ que
utilizaremos depois.

![imagem1](https://user-images.githubusercontent.com/23728459/184831592-0e44cd51-7c10-49df-809e-7754571acd66.png){: style="float: center; margin-right: 1em;" class="img-responsive" height="100%" width="100%"}

Linhas 5: Com este _for_ vamos passar por todos os elementos Web que
encontramos

Linha 6 a 12: Navegamos utilizando o CSS, utiliza-se Regex para extrair as
informações necessárias para o diário oficial

Linha 18: Damos início à criação do nosso objeto a ser inserido no projeto,
estruturando a informação no modelo do projeto, o
yield adiciona isso à fila de processamento e segue realizando downloads.


<h2>Subindo o código</h2>

Precisamos dizer ao git que iremos adicionar os arquivos alterados:

{% highlight bash %}
git add .
{% endhighlight %}

Damos início ao nosso commit:

{% highlight bash %}
git commit -m "Add spider Araraquara/SP"
{% endhighlight %}

Subimos nosso código para o repositório:
{% highlight bash %}
git push
{% endhighlight %}

![comemoracao](https://media1.giphy.com/media/IwAZ6dvvvaTtdI8SD5/giphy.gif?cid=ecf05e47aahkyosu6docb8kkvbrgyabdymqwzt4j7mwukir9&rid=giphy.gif&ct=g#center){: style="display: block; margin: auto;" class="img-responsive" height="50%" width="50%"}

Depois de fazer tudo isso, ao entrar na página do GitHub, vai
aparecer o botão para realizar o Pull Request, isso encerra uma contribuição com o projeto.
Se você tiver interesse em contribuir ou quiser conversar sobre, basta abrir um issue na página
do GitHub do projeto.


<h2>Conclusões</h2>

Contribuindo com a extração da cidade de Americana, pude aprender bastante
sobre a linguagem Python e também sobre Scrapy, quando expomos nosso código
publicamente a comunidade nos ajuda a realizar o melhor. Algo que já foi
aplicado na coleta da cidade de Araraquara. 

Espero que este relato tenha sido
útil e ajudado você que está iniciando no Scraping!


[fazer fork]: https://github.com/okfn-brasil/querido-diario/fork
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
[xpaths]: https://pt.wikipedia.org/wiki/XPath
[vetor (array)]: https://pt.wikipedia.org/wiki/Arranjo_(computa%C3%A7%C3%A3o)
[deputado devolver o dinheiro]: https://g1.globo.com/distrito-federal/noticia/apos-ser-flagrado-por-app-deputado-devolve-a-camara-r-727-por-13-refeicoes-no-mesmo-dia.ghtml
[instalar o git]: https://git-scm.com/book/pt-br/v2/Come%C3%A7ando-Instalando-o-Git
[instalar o Python no Linux]: https://python.org.br/instalacao-linux/
[no Windows]: https://python.org.br/instalacao-windows/
[diários de Araraquara]: https://www.diariooficialcmararaquara.sp.gov.br
[diários oficiais de Americana]: https://lcsvillela.com/querido-diario-monitorando-governo-com-scrapy.html
[cidade de Araraquara]: https://github.com/okfn-brasil/querido-diario/pull/603
