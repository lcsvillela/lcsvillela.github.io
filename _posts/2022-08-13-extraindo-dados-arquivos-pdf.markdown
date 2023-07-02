---
layout: post
title:  "Extração de dados em arquivos PDF com Python"
date:   2022-10-01 23:00:34 -0300
categories: PDF python data-minning
description: Dados e metadados nos ajudam a entender a informação, mas como extrair e estruturar?
image: https://github.com/lcsvillela/lcsvillela.github.io/assets/23728459/85b7b8d1-35a1-4970-a660-95cd39410e12 
thumbnail: https://github.com/lcsvillela/lcsvillela.github.io/assets/23728459/85b7b8d1-35a1-4970-a660-95cd39410e12
---

<h2>Um pouco sobre</h2>

Muitas vezes utiliza-se um sistema chamado ETL (Extract Transform Load), para podermos extrair
as informações de lugares da internet, transformar a informação para colocar em um formato
que desejamos e por fim carregar a informação em algum local de armazenamento e consulta.

Em um ETL, muitas vezes é necessário realizar a extração de dados de arquivos PDF, o que
pode ser em certo sentido desafiador a depender do arquivo utilizado e da complexidade
de organização da formatação interna do arquivo.

<h2>Qual caminho para extrair? Como funciona?</h2>

Extrair informações de um PDF é como se estivéssemos realizando
"web scraping" para PDF, portanto o nosso código será dependente da
estrutura do arquivo, o que faz com que tornar a informação desestruturada
de um arquivo em informação estruturada se torne um grande desafio.

A estrutura do arquivo, como você já deve ter percebido, não tem um padrão.
Isso faz com que o nível de dificuldade de extração seja muito variável,
do mesmo modo que ocorre em extrações de páginas web.

<h3>Preparando o ambiente</h3>

Utilizaremos a biblioteca Fitz no Python para realizar a extração, primeiro
vamos criar nosso ambiente virtual, verifique sua versão do Python:

{% highlight bash %}
pip install virtualenv
virtualenv --python=3.10 venv
{% endhighlight %}

Depois de criar o ambiente virtual, vamos ativá-lo:

{% highlight bash %}
source venv/bin/activate
{% endhighlight %}

Agora podemos instalar a biblioteca sem contaminar nosso
sistema operacional:

{% highlight bash %}
pip install PyMuPDF
{% endhighlight %}

Baixe o arquivo PDF que utilizaremos nesta demonstração, será o arquivo da
[chamada do vestibular deste ano da UNESP]

<h3>Exibindo todo o conteúdo do PDF</h3>

{% highlight python linenos %}
import fitz
import sys
import json

def abrir(arquivo):
    arquivo = open(arquivo, 'rb').read()
    arquivo = fitz.Document(stream=arquivo, filetype="pdf")

def extrair(arquivo):
    abrir(arquivo)

    informacao = {}
    curso = ""

    for page in arquivo:
        for lines in page.get_text('dict')['blocks']:
            for line in lines.get('lines', []):
                print(line)

if __name__ == "__main__":
    extrair(sys.argv[1])

{% endhighlight %}

Linha 1: importamos a biblioteca fitz


Linha 5: Criamos uma função que abre o arquivo especificado quando executamos o código 


Linha 6: o arquivo é aberto no modo leitura e binário, pois não é um
arquivo texto e o PDF pode ter outros arquivos dentro, como por exemplo
imagens que mantêm o seu formato original.


Linha 7: O arquivo é aberto utilizando o fitz, determinamos a fonte do
fluxo de informação e o tipo.


Linha 15: A variável __arquivo__ contêm todas as páginas do arquivo e podemos
iterar em cima dela utilizando o __for__


Linha 16: A variável __pages__ nos dá a possibilidade de iterar nas linhas
do arquivo (que não necessariamente são linhas, pois um arquivo PDF não é
construído dessa forma) e podemos extrair o formato em dicionário. O que nos
dá acesso a muitas informações.
Mas assim, podemos ter acesso a todos os dados do PDF, mesmo
o que não é texto e então veremos muita informação inicialmente
incompreensível, como por exemplo imagens em hexadecimal.


Linha 18: simplesmente exibe a o conteúdo da página


<h3>Extraindo e finalizando</h3>

Um método efetivo, porém trabalhoso, pode ser utilizado para extrair PDFs,
podemos analisar os tipos de fontes, o tamanho, dependendo
até outras características
apresentadas. Para isso, analise a estrutura a seguir:

{% highlight python %}
{'spans': [{'size': 6.891113758087158, 'flags': 20, 'font': 'CIDFont+F2', 'color': 132100, 'ascender': 0.89111328125, 'descender': -0.21630859375, 'text': '-', 'origin': (736.7427978515625, 420.8341979980469), 'bbox': (736.7427978515625, 414.77301025390625, 739.0677490234375, 422.30548095703125)}], 'wmode': 0, 'dir': (1.0, 0.0), 'bbox': (736.7427978515625, 414.77301025390625, 739.0677490234375, 422.30548095703125)}
{% endhighlight %}

Analisando essas estruturas do arquivo, podemos ver o padrão dos títulos que
iniciam os capítulos e adicioná-los em um vetor, em outro vetor, adicionamos
todas as estruturas e fazemos uma recombinação entre ambos.

No caso, o arquivo que estamos utilizando é algo muito mais simples e sua extração pode ser
feita da seguinte forma:

{% highlight python linenos %}
import fitz
import sys
import json

def abrir(arquivo):
    arquivo = open(arquivo, 'rb').read()
    arquivo = fitz.Document(stream=arquivo, filetype="pdf")
    
def extrair(arquivo):
    abrir(arquivo)

    informacao = {}
    curso = ""

    for page in arquivo:
        for lines in page.get_text('dict')['blocks']:
            for line in lines.get('lines', []):
                if 'integral' in line['spans'][0]['text'] or 'diurno' in line['spans'][0]['text'] or 'matutino' in line['spans'][0]['text'] or 'noturno' in line['spans'][0]['text']:
                    curso = line['spans'][0]['text'].replace(" ", "-")
                    if curso not in informacao.keys():
                        informacao[curso] = []
                if line['spans'][0]['text'].isupper() and len(list(informacao.keys())):
                    informacao[curso].append(line['spans'][0]['text'].replace('*', ''))

    resultado = open(sys.argv[1].split('.')[0]+".json", "w")
    json.dump(informacao, resultado)
    resultado.close()


if __name__ == "__main__":
    extrair(sys.argv[1])

{% endhighlight %}

O que mudou do código anterior para esse foi apenas a inserção de uma ideia que possibilita analisarmos
o conteúdo do dicionário que representa um elemento no PDF e caso o texto seja em CAIXA ALTA, que é o que nos
interessa para extrair os nomes, podemos montar uma estrutura de dicionário e armazenar em um vetor, depois disso
teremos todos os nomes das pessoas que passaram no vestibular da UNESP no ano de 2023 classificado em um
formato JSON com o nome do curso, possibilitando pesquisar na estrutura posteriormente.

Linha 18: Adicionado condição que identifica na linha as palavras integral, noturno e diurno, que sempre
aparecem nos nomes dos cursos

Linha 19: Nome da identificação do trecho que determina algum curso, muda um pouco por conta de
algum identificador interno

Linha 20: Verifica se o curso já foi adicionado no vetor

Linha 21: Caso não tenha sido adicionado ao vetor, cria um novo espaço

Linha 22: Algo que especifica a característica de um nome de uma pessoa na lista do vestibular é que todas
as letras são em caixa alta

Linha 25 a 27: Salvo os dados novos no arquivo


<h3>Conclusão</h3>

A extração de dados de arquivos PDF podem ser pouco triviais, este arquivo de exemplo tem poucos problemas, mas
podemos ter situações em que textos "fantasmas", textos quebrados em várias estruturas de dicionários diferentes,
organizações de informações em tabelas, imagens e todo tipo de dificuldade imaginada. Algo que seria um exemplo
de arquivo interessante de se extrair informações de forma perfeita são os diários oficiais de município, que
além de não serem padronizados, podem ser literalmente um formato qualquer dentro do PDF.

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
[chamada do vestibular deste ano da UNESP]: https://documento.vunesp.com.br/documento/stream/MTUxNjQ3OQ%3d%3d
