---
layout: post
title:  "Tweetando com Selenium em 56 linhas!"
date:   2021-04-15 20:00:34 -0300
categories: programacao_python selenium
---
*Veja o [código completo]{:target="_blank"} e uma [demonstração]{:target="_blank"}!*

O [Selenium]{:target="\_blank"} é um [framework]{:target="\_blank"} que consegue automatizar ações no seu navegador,
então podemos utilizá-lo para fazer tarefas online em sites complexos, 
que executam [Javascript]{:target="_blank"} e outras tecnologias, que outros frameworks como o
[Scrapy]{:target="_blank"} e [Beautiful Soup]{:target="_blank"} não conseguem, pois tratam apenas de [HTML]{:target="_blank"}.

<h3>Configurando o ambiente no Linux</h3>
*Veja para [Windows] ou [Mac OS]*

Primeiro, é necessário que você crie um ambiente virtual do [Python]{:target="_blank"} em seu
sistema, desse modo, você consegue isolar os pacotes instalados e garantir
que a execução ocorra normalmente, sem ter chance de entrar em conflito com
outros pacotes instalados do Python. Em seu terminal faça:

{% highlight bash %}
virtualenv --python=python3.7 venv
source venv/bin/activate
{% endhighlight %}

Desse modo, aparecerá (venv) no canto mais à esquerda da tela, isso significa
que o ambiente virtual está sendo utilizado e que podemos instalar pacotes
do Python de forma isolada.

{% highlight bash %}
pip install selenium
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

<h3>Mão na massa do Selenium</h3>

Tenha em mente que realizaremos diversos passados para realizar uma publicação no Twitter utilizando o Selenium,
por isso, é muito importante segmentarmos o código o máximo possível, onde teremos vários [métodos]{:target="_blank"}. Como nosso projeto
é algo simples, teremos apenas uma [classe]{:target="_blank"}. Por mais simples que seja, é importante aplicarmos 
[POO]{:target="_blank"}, pois podemos fazer o reúso do software, o que aumenta cada vez mais a
velocidade de implementação de tarefas.

<h2> Primeiro passo: Criando o esboço</h2>

Primeiro vamos apenas importar as bibliotecas utilizadas:

{% highlight python linenos%}
from selenium import webdriver
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By
from sys import argv
{% endhighlight %}

Agora podemos criar nossa classe e os métodos que vamos precisar:

{% highlight python linenos %}

class Tweet():
    def __init__(self, username, password,
                 text="Fui automatizado! Confira: https://lcsvillela.github.io/publicando-tweet-com-python.html"):
        self.__username = username
        self.__password = password
        self.__text = text
        self.__driver = webdriver.Firefox()
        self.__wait = WebDriverWait(self.__driver, 30)
        self.__actions = ActionChains(self.__driver)
        self.__locations = {'user': 'session[username_or_email]',
                            'password': 'session[password]',
                            'button_login': '//form/div/div[3]',
                            'button_tweet': "//*[@data-testid='tweetButtonInline']"}
        self.start()
	
    def login_twitter(self):
        pass
	
    def make_tweet(self):
        pass
	
    def get_field_text(self):
        pass
    
    def start(self):
        self.login_twitter()
        self.make_tweet()

tweet = Tweet(argv[1], argv[2], argv[3])
{% endhighlight %}

Na linha 2 criamos nosso \__init\__, é um método que é executado assim que a classe é instanciada, então podemos colocar os
atributos definidos por lá, algo que nem sempre é feito, é o "self.\__" que ajuda muito na leitura e identificação rápida
de um atributo no seu programa, sem o risco de confundir com um método! Nele, definimos o recebimento de alguns
parâmetros, sendo que um deles pode ser omitido no momento de instanciação da classe, que é o __text__, que conta com
um valor padrão.

Nas linhas de 4 a 6, fazemos com que os atributos recebam os valores das variáveis, isso faz com que nossa classe seja
isolada do resto do programa, sem que seja possível alterar os atributos diretamente, o que pode gerar problemas.

Nas linhas 7 a 9, iniciamos a nossa instância do nevagador que será automatizado, definimos um padrão para o
WebDriverWait e para as ActionChains (lembre-se do [DRY]{:target="_blank"}).

Nas linhas 10 a 13, cria-se uma [estrutura de dicionário]{:target="_blank"} em Python, para centralizar as informações dos identificadores
dos campos de usuário, senha, campo de texto e botão do tweet, [veja aqui como descobrir]{:target="_blank"} essa informação na página.
Na 13 é onde nosso programa é executado.

Nas linhas 16, 19 e 22 , criamos nossos métodos, por enquanto, eles não fazem nada (com exceção do 25 que chama os outros)
e é para isso que serve o *pass* nesse caso, apenas fechar um buraco de um método ou função que será implementado. A útilma
linha é algo que torna conveniente o uso do programa, não precisamos alterar o nome de usuário, senha e texto para
o tweet toda hora, basta chamar o programa com os dados na frente nessa ordem!

Mas você já pode executar nosso programa! Ele não retornará nada, mas [muitas coisas já aconteceram]{:target="_blank"} :)

{% highlight python %}
python tweet.py usuario senha "O que você quiser publicar"
{% endhighlight %}

<h2> Segundo passo: Fazendo o Login</h2>

Para chegar nas informações do Xpath, CSS Selector, CSS Path ou em algum outro jeito que seja possível identificar de
forma única um elemento de uma página web, você precisa se debruçar no código fonte da página, no caso do twitter,
a empresa faz com que elementos de sua página sejam aleatórios, tenham restrições de interação e outros jeitos de tentar
impedir a gente de automatizar uma das maiores redes sociais do mundo.

Porém, a parte de login é a mais tranquila, nosso método fica:

{% highlight python linenos %}
def login_twitter(self):
    self.__driver.get("https://twitter.com/login")
    self.__wait.until(EC.presence_of_element_located((By.NAME, self.__location['username'])))
    self.__wait.until(EC.presence_of_element_located((By.NAME, self.__location['password'])))
    field_user = self.__driver.find_element_by_name(self.__location['username'])
    field_user.send_keys(self.__username)
    field_pass = self.__driver.find_element_by_name(self.__location['password'])
    field_pass.send_keys(self.__password)
    self.__driver.find_element_by_xpath(self.__location['button_login']).click()
{% endhighlight %}

A segunda linha, o webdriver utiliza sua função get para acessar a página, na terceira e quarta, definimos que seja
aguardado o carregamento dos elementos que iremos interagir, é pouco recomendado a utilização da biblioteca *time.sleep*,
pois não existem garantias de tempo.
Nas linhas 5 e 6, localizamos o elemento para inserir o nome do usuário e logo depois enviamos os dados, o memso é feito
com o campo de senha nas linhas 7 e 8. Por fim, realizamos um click no botão de login. Fácil, não é mesmo?

<h2> Terceiro passo: Fazendo o tweet</h2>

Aqui foi necessário criar dois métodos para dar conta de um problema, porque na realidade, são dois problemas diferentes:

0 - Ao analisar cuidadosamente, percebe-se que o campo de inserção de texto para o tweet tem uma ID, algo muito valioso
quando estamos utilizando o Selenium, pois um Xpath pode mudar do dia para a noite, mas um ID geralmente fica um bom tempo.
Porém o ID deste elemento do Twitter tem uma parte estática e outra variável.
1 - Realizar a inserção do texto e o clique no botão de tweet

{% highlight python %}
def get_field_text(self):
    code = self.__driver.page_source.split('"')
    return [x for x in code if 'placeholder-' in x][0]
{% endhighlight %}

A primeira linha nos faz o favor de trazer o código fonte da página, depois com o método split, fatiamos o código todo e
assim obtemos um vetor. De posse desse vetor, na [linha 2] fazemos um laço for que filtra os elementos do vetor que
interessam e assim obtemos o nosso ID.

{% highlight python linenos%}
def make_tweet(self):
    self.__wait.until(EC.presence_of_element_located((By.XPATH, self.__location['button_tweet'])))
    ID = self.get_field_text()
    self.__wait.until(EC.element_to_be_clickable((By.ID, ID)))
    field_text = self.__driver.find_element_by_id(ID)
    self.__actions.move_to_element(field_text).click(field_text).send_keys(self.__text).perform()
    self.__driver.find_element_by_xpath(self.__location['button_tweet']).click()
{% endhighlight %}

Na linha dois, novamente utilizamos as ferramentas de análise para esperar a página carregar com sucesso, na linha três
executamos o método que busca o ID e armazenamos ele, logo em seguida na linha quatro, utilizamos o verificador da situação da página
para saber se o elemento é clicável. Na linha cinco fazemos a localização do elemento, na linha 6 utilizamos a ferramenta
de ações do Selenium, que nos possibilita emular movimentos mais complexos, sendo assim, enviamos nosso texto para o campo e
na última linha, finalmente clicamos no botão para tweetar!

<h2> Conclusões, observações e recomendações</h2>

Incrível o que em apenas algum tempo podemos fazer, não é? De posse deste programa, podemos publicar qualquer coisa no Twitter
de forma automatizada, não precisamos nos expor a uma ferramenta que causa [problemas psicológicos]{:target="_blank"},
pois podemos fazer isso pelo terminal e até mesmo agendada utilizando o cron!

As ideias afloram, mas algo que é de suma importância é a necessidade de criarmos um [código limpo]{:target="_blank"}, que
facilite sua manutenção e potencialize sua reuzabilidade! Este código é uma biblioteca que permite a publicação no twitter,
uma das várias funções da rede social, para extender este código basta realizar um import dela e utilizar as propriedades
da programação orientada a objetos. Quem sabe...pode ser [nosso twitterpy]{:target="_blank"}?

Veja como ficou o código completo:
{% highlight python linenos%}
''' Criado por www.lcsvillela.github.io
www.instagram.com/lcsvillela
www.twitter.com/lcsvillela
www.github.com/lcsvillela '''

from selenium import webdriver
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By
from selenium.webdriver import ActionChains
from sys import argv


class Twitter():
    def __init__(self, username, password,
                 text="Fui automatizado! Confira: www.github.io/publicando-tweet-com-python.html"):
        self.__username = username
        self.__password = password
        self.__driver = webdriver.Firefox()
        self.__text = text
        self.__wait = WebDriverWait(self.__driver, 30)
        self.__actions = ActionChains(self.__driver)
        self.__location = {'username': 'session[username_or_email]',
                           'password': 'session[password]',
                           'button_login': '//form/div/div[3]',
                           'button_tweet': "//*[@data-testid='tweetButtonInline']"}
        self.start()

    def login_twitter(self):
        self.__driver.get("https://twitter.com/login")
        self.__wait.until(EC.presence_of_element_located((By.NAME, self.__location['username'])))
        self.__wait.until(EC.presence_of_element_located((By.NAME, self.__location['password'])))
        field_user = self.__driver.find_element_by_name(self.__location['username'])
        field_user.send_keys(self.__username)
        field_pass = self.__driver.find_element_by_name(self.__location['password'])
        field_pass.send_keys(self.__password)
        self.__driver.find_element_by_xpath('//form/div/div[3]').click()

    def make_tweet(self):
        self.__wait.until(EC.presence_of_element_located((By.XPATH, self.__location['button_tweet'])))
        ID = self.get_field_text()
        self.__wait.until(EC.element_to_be_clickable((By.ID, ID)))
        field_text = self.__driver.find_element_by_id(ID)
        self.__actions.move_to_element(field_text).click(field_text).send_keys(self.__text).perform()
        self.__driver.find_element_by_xpath(self.__location['button_tweet']).click()

    def get_field_text(self):
        code = self.__driver.page_source.split('"')
        return [x for x in code if 'placeholder-' in x][0]

    def start(self):
        self.login_twitter()
        self.make_tweet()


Twitter(argv[1], argv[2], argv[3])

{% endhighlight %}


[Selenium]: https://selenium-python.readthedocs.io/
[framework]: http://www.dsc.ufcg.edu.br/~jacques/cursos/map/html/frame/oque.htm
[Javascript]: https://pt.wikipedia.org/wiki/JavaScript
[Beautiful Soup]: https://arquivos.info.ufrn.br/arquivos/20190630744d8f67211189dcd25218c93/Apresentao_Scraping_de_Dados.pdf
[Scrapy]: https://medium.com/@marlessonsantana/utilizando-o-scrapy-do-python-para-monitoramento-em-sites-de-not%C3%ADcias-web-crawler-ebdf7f1e4966
[HTML]: https://pt.wikipedia.org/wiki/HTML
[Python]: https://pt.wikipedia.org/wiki/Python
[Geckodriver]: https://medium.com/beelabacademy/baixando-e-configurando-o-geckodriver-no-ubuntu-dc2fe14d91c
[Windows]: https://medium.com/ananoterminal/ambientar-selenium-no-windows-3b880fa0e827
[Mac OS]: https://medium.com/dropout-analytics/selenium-and-geckodriver-on-mac-b411dbfe61bc
[métodos]: https://pt.wikipedia.org/wiki/M%C3%A9todo_(programa%C3%A7%C3%A3o)
[POO]: https://www.ime.usp.br/~mms/mac1222s2019/3%20-%20PDA%20-%20Python%20-%20introducao%20a%20programa%C3%A7%C3%A3o%20orientada%20a%20objetos.pdf
[DRY]: https://pt.wikipedia.org/wiki/Don%27t_repeat_yourself
[classe]: https://docs.python.org/pt-br/3/tutorial/classes.html
[muitas coisas já aconteceram]: http://pythonclub.com.br/debugging-em-python-sem-ide.html
[linha 2]: https://web.archive.org/web/20180309053826/http://www.secnetix.de/olli/Python/list_comprehensions.hawk
[problemas psicológicos]: https://www.theatlantic.com/technology/archive/2017/07/how-twitter-fuels-anxiety/534021/
[nosso twitterpy]: https://lcsvillela.github.io/criando-um-robo-no-instagram.html
[estrutura de dicionário]: https://panda.ime.usp.br/pensepy/static/pensepy/11-Dicionarios/dicionarios.html
[veja aqui como descobrir]: https://escoladedados.org/tutoriais/xpath-para-raspagem-de-dados-em-html/
[código limpo]: https://becode.com.br/clean-code/
[código completo]: https://github.com/lcsvillela/TwitterPy
[demonstração]: https://youtu.be/HmKHaXMvjoQ