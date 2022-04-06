---
layout: post
title:  "Extraindo dados de vacinação contra Covid-19 do Maranhão"
date:   2022-04-06 00:59:40 -0300
categories: python scraping
description: Muitos dados de vacinação do Maranhão estão disponíveis!
image: https://user-images.githubusercontent.com/23728459/161891618-ff8dcd70-ea4c-44f3-8108-16eb2ea44d6c.png
thumbnail: https://user-images.githubusercontent.com/23728459/161893977-42dec7ff-49ef-482b-b867-9956643ca9b4.png
---


<h2>Um pouco sobre Scraping</h2>

Scraping é algo que com certeza movimenta bilhões de dólares por ano na economia mundial, basicamente é o ato de
extrair dados de sites de forma automatizada, falo um pouco sobre neste [primeiro]{:target="\_blank"} texto onde
mostro como extraí milhares de frases do site pensador, e aqui um pouco sobre a ideia de web scraping
neste [segundo]{:target="\_blank"}. Dito isto, mão na massa!

<h2>Como extrair este site?</h2>

O site [Painel Covid do Maranhão] tem uma interface onde tudo tem algum restrição por cookies e códigos gerados aleatoriamente, mas
além disso conta também existem outros problemas, mas vamos ver como podemos analisar o site e fazer sua
extração dos dados de vacinação do estado.

<h3>Analisando</h3>

![imagem1](https://user-images.githubusercontent.com/23728459/154641082-7d1600a0-2781-4dac-9aca-08f94cb0be08.png){: style="float: center; margin-right: 1em;" class="img-responsive" height="100%" width="100%"}

A página inicial do site é esta e temos que clicar em vacinas, tente pegar os endereços do
site de forma direta não funcionará, por que será? Será por quê? Vamos
investigar algumas coisas, pra isso, pressine F12.

![2](https://user-images.githubusercontent.com/23728459/154641104-5c03a01e-af55-41ad-9256-fc9bcab10dfe.png){: style="float: center; margin-right: 1em;" class="img-responsive" height="100%" width="100%"}

"Depois de pressionar F12, você deve algo assim:"


![3](https://user-images.githubusercontent.com/23728459/154641763-7ea0a694-fa9f-47cd-aa98-e28b52af2c88.png){: style="float: center; margin-right: 1em;" class="img-responsive" height="100%" width="100%"}


Note que usando essa ferramenta, podemos clicar em cada requisição, existem alguns tipos de requisições
do protocolo HTTP, podemos ver os dados de cada uma e entender o que o site
está realizando.


![4](https://user-images.githubusercontent.com/23728459/154780742-840aa2d8-8260-4ea8-adaf-ee0ea645f044.png){: style="float: center; margin-right: 1em;" class="img-responsive" height="100%" width="100%"}


Recomendo que gaste um tempo observando os itens, pesquise os que não entender e recarregue a
página algumas vezes para entender como ela realmente funciona, é um processo investigativo
para podermos automatizar as requisições realizadas pelo navegador. Se você for mais raiz,
possível que goste do [mitmproxy]{:target="\_blank"}, esta ferramenta mostra cada requisição
e resposta feita.

Mas porque fazer a análise? Muitos sites utilizam tecnologias diferentes, alguns utilizam
APIs para alimentar o front-end, criam mecanismos de autenticação internos para realizar
requisições e mitigar o [CSRF]{:target="\_blank"}, talvez algum [cookie]{:target="\_blank"} ou
alguma outra coisa que pode ficar no seu pé. Pra descobrir isso, primeiro olho os cabeçalhos,
a requisição e as respostas de cada requisição e vejo o que muda para rastrear onde exatamente surgiu
a informação.

Com o passar do tempo neste site, você vai perceber que apareceu um fator de autenticação nas requisições,
em alguma requisição esse token foi gerado, onde? Fazendo uma pesquisa, encontramos a requisição responsável,
no caso está no arquivo JavaScript "main-es2015.1fd5ebecf22be55d7c2e.js", o nome desse arquivo pode mudar
facilmente, então precisamos de um jeito para encontrá-lo sempre! Veja na imagem abaixo que nos cabeçalhos da
requisição apareceu o campo _Authorization_:

![image](https://user-images.githubusercontent.com/23728459/161399525-c337f322-7779-4541-973b-a4596c0da821.png){: style="float: center; margin-right: 1em;" class="img-responsive" height="100%" width="100%"}

Além disso, depois de analisar as requisições, percebemos que este site é apenas uma casca, o front-end
consome uma API para alimentar as informações que são exibidas. No próprio registro acima, podemos ver que
o domínio de requisição foi alterado.

Com um pouco mais de investigação nessa página, achamos os dados de vacinação por município, que é o que queremos.
Para facilitar e não precisarmos registrar cidade por cidade, podemos extrair todos os números que a identificam
no [IBGE]{:target="\_blank"}, eles são usados para fazer a requisição no endpoint e precisamos identificar onde estão
os dados que queremos. Para isso, precisamos utilizar mais a página, depois de clicar nos dados de Vacinação Estadual,
abrirá uma tela onde tem os dados regionais e municipais, nós queremos o segundo. Então, ao acessá-lo,
vemos que os dados já estão organizados em um JSON, o que nos faz muito felizes.

![7](https://user-images.githubusercontent.com/23728459/161569331-2c010641-559c-44a3-99d3-fa1321b45da7.png){: style="float: center; margin-right: 1em;" class="img-responsive" height="100%" width="100%"}

![8](https://user-images.githubusercontent.com/23728459/161569491-3d8b8d35-8110-4139-865b-087e41346dcb.png){: style="float: center; margin-right: 1em;" class="img-responsive" height="100%" width="100%"}

<h2>Realizando a extração</h2>

{% highlight python linenos %}
import requests
import json
from unidecode import unidecode
import re


class Maranhao:
    def __init__(self):
        url = "https://painel-covid19.saude.ma.gov.br"
        response = requests.get("https://painel-covid19.saude.ma.gov.br")
        element = re.findall(r'src="main.*"', response.text)[0].split('"')[1]
        self.__token = requests.get(url + "/" + element).content.decode()
        self.__token = (
            re.findall(r"token:\".*?\"", self.__token)[0].split(":")[1].replace('"', "")
        )
        self.__headers = {"Authorization": f"Bearer {self.__token}"}
        self.__endpoint = {
            "todas_cidades": "https://painel-covid19-back.saude.ma.gov.br/v1/covid19/municipios/vacinas/resumo",
            "cidade_detalhada": "https://painel-covid19-back.saude.ma.gov.br/v1/covid19/municipios/vacinas/",
        }
        self.__codigo_ibge = []
        self.__data = {}
        self.start()

    def start(self):
        self.get_cidades_ibge()
        data = self.get_cidade()
        self.save(data)

    def make_request(self, url):
        return json.loads(requests.get(url, headers=self.__headers).content)

    def get_cidades_ibge(self):
        cidades = self.make_request(self.__endpoint["todas_cidades"])
        for cidade in cidades:
            self.__codigo_ibge.append(cidade["codigo_ibge"])

    def save(self, data):
        for _data in data:
            city = unidecode(_data["prefeitura"]).replace(" ", "_")
            file = open(f"{city}.json", "w")
            file.write(json.dumps(_data))
            file.close()

    def get_cidade(self):
        _data = []
        for codigo in self.__codigo_ibge:
            cidade = self.make_request(
                self.__endpoint["cidade_detalhada"] + str(codigo)
            )
            _data.append(
                {
                    "prefeitura": cidade["municipio"],
                    "pupulacao-12+": cidade["populacao_vacina_12_17"]
                    + cidade["populacao_vacina_18"],
                    "doses-imunizacao": "8.658",
                    "cobertura-populacional": round(
                        cidade["cobertura_populacional"] * 100, 2
                    ),
                    "doses-distribuidas-total": cidade["doses_recebidas"],
                    "doses-aplicadas-total": cidade["doses_aplicadas"],
                    "cobertura-de-doses-aplicadas": round(
                        cidade["cobertura_geral"] * 100, 2
                    ),
                    "doses-distribuidas-tipo": {
                        "coronavac": {
                            "1a_dose": cidade["vacinas"][0]["doses_recebidas_d1"],
                            "2a_dose": cidade["vacinas"][0]["doses_recebidas_d2"],
                            "dr-cv": cidade["vacinas"][0]["doses_recebidas_d3"],
                            "total-recebidas-cv": cidade["vacinas"][0][
                                "doses_recebidas_total"
                            ],
                        },
                        "astrazeneca": {
                            "1a_dose": cidade["vacinas"][1]["doses_recebidas_d1"],
                            "2a_dose": cidade["vacinas"][1]["doses_recebidas_d2"],
                            "dr-az": cidade["vacinas"][1]["doses_recebidas_d3"],
                            "total-recebidas-az": cidade["vacinas"][1][
                                "doses_recebidas_total"
                            ],
                        },
                        "pfizer": {
                            "1a_dose": cidade["vacinas"][2]["doses_recebidas_d1"],
                            "2a_dose": cidade["vacinas"][2]["doses_recebidas_d2"],
                            "dr-pfz": cidade["vacinas"][2]["doses_recebidas_d3"],
                            "total-recebidas-pf": cidade["vacinas"][2][
                                "doses_recebidas_total"
                            ],
                        },
                        "janssen": {
                            "du": cidade["vacinas"][3]["doses_recebidas_total"]
                        },
                    },
                    "doses-aplicadas-tipo": {
                        "coronavac": {
                            "1a_dose": cidade["vacinas"][0]["doses_aplicadas_d1"],
                            "2a_dose": cidade["vacinas"][0]["doses_aplicadas_d2"],
                            "dr-cv": cidade["vacinas"][0]["doses_aplicadas_d3"],
                            "total-aplicada-cv": cidade["vacinas"][0][
                                "doses_aplicadas_total"
                            ],
                        },
                        "astrazeneca": {
                            "1a_dose": cidade["vacinas"][1]["doses_aplicadas_d1"],
                            "2a_dose": cidade["vacinas"][1]["doses_aplicadas_d2"],
                            "dr-az": cidade["vacinas"][1]["doses_aplicadas_d3"],
                            "total-aplicada-az": cidade["vacinas"][1][
                                "doses_aplicadas_total"
                            ],
                        },
                        "pfizer": {
                            "1a_dose": cidade["vacinas"][2]["doses_aplicadas_d1"],
                            "2a_dose": cidade["vacinas"][2]["doses_aplicadas_d2"],
                            "dr-pfz": cidade["vacinas"][2]["doses_aplicadas_d3"],
                            "total-aplicada-pfz": cidade["vacinas"][2][
                                "doses_aplicadas_total"
                            ],
                        },
                        "janssen": {
                            "du": cidade["vacinas"][3]["doses_aplicadas_total"]
                        },
                    },
                    "cobertura-aplicadas-tipo": {
                        "coronavac": {
                            "1a_dose": cidade["vacinas"][0]["cobertura_d1"],
                            "2a_dose": cidade["vacinas"][0]["cobertura_d2"],
                            "dr-cv": cidade["vacinas"][0]["cobertura_d3"],
                            "total-cobertura-aplicada-cv": cidade["vacinas"][0][
                                "cobertura_total"
                            ],
                        },
                        "astrazeneca": {
                            "1a_dose": cidade["vacinas"][1]["cobertura_d1"],
                            "2a_dose": cidade["vacinas"][1]["cobertura_d2"],
                            "dr-az": cidade["vacinas"][1]["cobertura_d3"],
                            "total-cobertura-aplicada-az": cidade["vacinas"][1][
                                "cobertura_total"
                            ],
                        },
                        "pfizer": {
                            "1a_dose": cidade["vacinas"][2]["cobertura_d1"],
                            "2a_dose": cidade["vacinas"][2]["cobertura_d2"],
                            "dr-pfz": cidade["vacinas"][2]["cobertura_d3"],
                            "total-cobertura-aplicada-pfz": cidade["vacinas"][2][
                                "cobertura_total"
                            ],
                        },
                        "janssen": {"du": cidade["vacinas"][3]["cobertura_total"]},
                    },
                    "grupo_vacina": {
                        "coronavac": {
                            "doses_recebidas_coronavac_d2_indigena": cidade["vacinas"][
                                0
                            ]["doses_aplicadas"][0],
                            "doses_recebidas_coronavac_d3_indigena": cidade["vacinas"][
                                0
                            ]["doses_aplicadas"][1],
                        },
                        "astrazeneca": {
                            "doses_recebidas_coronavac_d2_indigena": cidade["vacinas"][
                                1
                            ]["doses_aplicadas"][0],
                            "doses_recebidas_coronavac_d3_indigena": cidade["vacinas"][
                                1
                            ]["doses_aplicadas"][1],
                        },
                        "pfizer": {
                            "doses_recebidas_coronavac_d2_indigena": cidade["vacinas"][
                                2
                            ]["doses_aplicadas"][0],
                            "doses_recebidas_coronavac_d3_indigena": cidade["vacinas"][
                                2
                            ]["doses_aplicadas"][1],
                        },
                    },
                }
            )
            print(_data)
        return _data


Maranhao()

{% endhighlight %}

Para melhor compreender este código, recomendo: [Programando com classe sem smokin - Parte 1]{:target="\_blank"}

* Linha 1 a 4: Faz a importação das bibliotecas necessárias para realizar a extração dos dados
* Linha 7: Criamos a classe Maranhao 
* Linha 8: Criamos o método que é iniciado quando a classe é instanciada
* Linha 9 a 23: Fazemos as requisições mais básicas, pegamos o token que está no javascript usando regex,
definimos um cabeçalho para as requisições com o token de autenticação e declaramos as variáveis usadas em toda
a classe
* Linha 25 a 28: Criamos o método _start_ que será nosso método principal e executado assim que a classe é instanciada
* Linha 30 e 31: Como sempre fazemos requisições e nosso cabeçalho não muda, o método ajuda a gente a se comunicar apenas
com os endpoints, sem se preocupar tanto com o cabeçalho
* Linha 33 a 36: Fazemos uma requisição para pegar do endpoint todos os identificadores das cidades do IBGE, usaremos para
fazer as requisições
* Linha 38 a 43: A ideia aqui é pegar cada dado extraído e gerar um arquivo JSON para cada cidade
* Linha 45 a 179: Fazemos a extração dos dados da cidade e depois organizamos os dados em um formato que queremos de forma arbitrária,
deixando de modo que seja mais fácil trabalhar com eles de acordo com um sistema já existente, a importância aqui é adaptar o modelo.
Depois de criado o vetor com os dicionários contendo os dados, é retornado o vetor com os dados de todas as cidades!

<h2>Pulo do gato: certificado SSL</h2>

Verifique no [projeto de extração dos dados de vacinação]{:target="\_blank"} que existe um arquivo pem, se você rodou o programa acima,
receberá um erro sobre o certificado. Ele é usado para fazer a comunicação do frontend com o backend, precisamos então copiá-lo.
Com o comando abaixo, podemos extrair o certificado que nos permite se comunicar com o backend:

{% highlight bash %}
echo | openssl s_client -showcerts -servername painel-covid19.saude.ma.gov.br -connect painel-covid19.saude.ma.gov.br:443 2>/dev/null | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > chain.crt
{% endhighlight %}

Depois disso, basta mover o certificado para o local certo.

{% highlight bash %}
sudo mv chain.crt /usr/local/share/ca-certificates/
{% endhighlight %}

<h3>Um outro jeito de realizar a mesma coisa</h3>

Para ver onde você deve instalar o certificado, você também pode utilizar o certifi no python,
o comando a seguir mostra o arquivo que é responsável por armazená-los:

{% highlight bash %}
>>> certifi.where()
'/venv/lib/python3.9/site-packages/certifi/cacert.pem'
{% endhighlight %}




<h2>Conclusão</h2>


Ao executar o programa, você verá diversos arquivos sendo gerados,
um para cada município do Maranhão. Agora podemos armazenar os dados,
analisá-los e exibí-los como quisermos!
![9](https://user-images.githubusercontent.com/23728459/161889087-3b82f88a-312d-47c8-ab5f-aaa0e2b7f077.png){: style="float: center; margin-right: 1em;" class="img-responsive" height="100%" width="100%"}


[Painel Covid do Maranhão]: https://painel-covid19.saude.ma.gov.br/
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
[CSRF]: https://www.ibm.com/docs/pt-br/sva/10.0.0?topic=configuration-prevention-cross-site-request-forgery-csrf-attacks
[mitmproxy]: https://mitmproxy.org/
[cookie]: https://pt.wikipedia.org/wiki/Cookie_(inform%C3%A1tica)
[IBGE]: https://www.ibge.gov.br/
[Programando com classe sem smokin - Parte 1]: https://lcsvillela.com/programe-com-classe-like-a-lady-or-a-sir.html
[projeto de extração dos dados de vacinação]: https://github.com/lcsvillela/MonitorVacinasMaranhao
