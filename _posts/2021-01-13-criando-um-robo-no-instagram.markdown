---
layout: post
title:  "Criando um robô para o seu Instagram"
date:   2021-01-13 16:32:34 -0300
categories: instagram 
---

![image tooltip here](https://camo.githubusercontent.com/4cd0bf58a8cffad17820826a99facc2821a8c50d/68747470733a2f2f692e696d6775722e636f6d2f734a7a665a734c2e6a7067){: style="float: left" class="img-responsive" height="30%" width="30%"}

Todo ano ocorre o evento [EuroPython], sendo realizado a apresentação de projetos e mudanças na linguagem [Python]. Uma dessas palestras, estava o [Tim Großmann] falando sobre seu projeto [Instapy] (você pode ver o [projeto aqui]), que consistia em um [framework] para a automação de ações nas redes sociais. O fato é que as redes sociais são na realidade justamente o contrário do nome, são redes antisociais e que causam [problemas] e [mais problemas] psicológicos significativos, ou seja, se podemos automatizar as nossas ações na "rede social", então podemos nos afastar desses problemas e de quebra conseguir seguir pessoas de verdade.

Großmann fez o framework de modo que tudo seja aleatório e vocẽ não precisa se preocupar em alterar as frequências de ações, ou seja, é uma imitação humana, você pode ler [este artigo] de como ele conseguiu 2500 seguidores em 6 meses, no meu caso, consegui 800 utilizando apenas meu notebook. Mas vamos ao que realmente importa:

<h2>Preparando o ambiente</h2>

Antes de prosseguir, é necessário instalar o python em sua máquina, para isso, entre no site oficial e faça o download para o seu sistema operacional. Você pode ver como instalar no [Windows aqui] e no [Linux aqui].

Depois de instalar o Python em sua máquina, abra o terminal do Windows ou do Linux e digite o seguinte comando:

{% highlight python %}
pip install instapy
{% endhighlight %}

Depois disso, você vai precisar instalar o firefox e o geckodriver, veja aqui as instruções.

<h2>Criando um robô simples</h2>

O instapy é extramente completo, podemos por exemplo seguir perfis que seguem perfis, seguir perfis que são seguidos por perfis, seguir quem curtiu alguma publicação específica e muito mais! Você pode ver a documentação para saber mais clicando aqui.

A ideia é criar um robô simples, que siga os perfis que seguem um perfil determinado, primeiro, precisamos importar as bibliotecas do python. Abra um editor de texto qualquer, se você estiver no Windows, abra o notepad e se estiver no Linux abra o vim ;)

Coloque no arquivo:

{% highlight python %}
from instapy import InstaPy as insta
from instapy import Settings as settings
from instapy import smart_run
from time import sleep
{% endhighlight %}

Até aqui, apenas importamos as bibliotecas necessárias para iniciarmos a programação do nosso bot, agora é onde começamos a realizar a mágica!

Depois disso, nosso próximo passo será fazer o login em nossa conta no instagram, para isso precisamos determinar os parãmetros
de username, password e headless_browser. Este último define se o seu navegador ficará visível para você durante as ações,
prefiro que não, então coloquei True, se você quiser ver seu navegador realizando as ações, coloque False!

{% highlight python %}
settings()
contador = 4
sessao = insta(username="SeuUserSem@"
               password="SuaSenha"
               headless_browser=True)
sessao.login()
{% endhighlight %}

Aqui temos a configuração do tipo de usuários que não queremos seguir, skip_no_profile_pic define se vamos ou não
seguir usuários que estão sem foto e no_profile_pic_percentage define quantas vezes isso acontecerá, no caso 100% das vezes.

{% highlight python %}
sessao.set_skip_users(skip_no_profile_pic=True,
                      no_profile_pic_percentage=100,
                      skip_private=True,
                      private_percentage=100)
{% endhighlight %}

Agora a nossa ideia é fazer a mesma tarefa repetidas vezes, usando a estrutura de decisão while, que no caso
sempre executa o código que pertence a ela (isso é definido pelo espaçamento no python, diferente de C ou Java). Nesse sentido,
o comando sessao.follow_user_followers primeiro chama a nossa seção definida com nossas credenciais e depois chama o método
follow_user_followers, que faz com que sigamos usuários que seguem os perfis citados. No caso, coloquei dois perfis que são
referência de conteúdo técnico brasileiro para mim: [PizzaDeDados] e [MenteBinaria]. Logo depois, definimos com amount
a quantidade de perfis que vamos seguir, com randomize definimos que a ação não será previsível, dando a impressão que somos
humanos.
Depois disso, a nossa seção é finalizada e o programa espera por 1200 segundos (20 minutos) e subtrai o contador para reiniciar esse ciclo.

{% highlight python %}
while contador != 0:
    sessao.follow_user_followers(['pizzadedados','mentebinaria_'],
    amount=10,
    randomize=True,
    sleep_delay=120)
    sessao.end(threaded_session=True)
    sleep(1200)
    contador-=1
{% endhighlight %}

Abaixo, podemos ver como o código do nosso bot ficou:

{% highlight python %}
from instapy import InstaPy as insta
from instapy import Settings as settings
from instapy import smart_run
from time import sleep

settings()
contador = 4
sessao = insta(username="SeuUserSem@"
               password="SuaSenha"
               headless_browser=True)
sessao.login()
sessao.set_skip_users(skip_no_profile_pic=True,
                      no_profile_pic_percentage=100,
                      skip_private=True,
                      private_percentage=100)
while contador != 0:
    sessao.follow_user_followers(['pizzadedados','mentebinaria_'],
    amount=10,
    randomize=True,
    sleep_delay=120)
    sessao.end(threaded_session=True)
    sleep(1200)
    contador-=1
{% endhighlight %}

Após isso, o programa está pronto, você só precisa salvar o arquivo. Agora, o pulo do gato é colocarmos isso para ser executado automaticamente agendando tarefas [no Windows] e [no Linux].

<h2>Conclusão</h2>

A automação pode nos livrar de uma tarefa simples e tediosa, que é seguir perfis no instagram, 
um dos vários jeitos de impulsionar um perfil, portanto isso se torna importante
 quando você é por exemplo um profissional de [social media] com muitos perfis 
de instagram para gerenciar (fica o desafio: Use um programa pra acessar vários 
perfis automatizados), assim você pode deixar o seu cliente mais satisfeito, 
pois os usuários que deram o seguir de volta existem de verdade e 
ficarão por tempo indeterminado te seguindo, muito mais qualitativo do 
que entregar um panfleto de propaganda na rua, não é mesmo?


[EuroPython]: https://wiki.python.org/moin/EuroPython
[Python]: https://www.youtube.com/watch?v=uOgDa1rlqjE
[Tim Großmann]: https://twitter.com/timigrossmann
[instapy]: https://www.youtube.com/watch?v=bGJ41J5b_90
[projeto aqui]: https://github.com/timgrossmann/InstaPy
[framework]: https://pt.wikipedia.org/wiki/Framework
[problemas]: https://www.sciencedirect.com/science/article/abs/pii/S0924977X19313331
[mais problemas]: https://www.sciencedirect.com/science/article/pii/S0747563217300493
[este artigo]: https://medium.com/free-code-camp/my-open-source-instagram-bot-got-me-2-500-real-followers-for-5-in-server-costs-e40491358340
[Windows aqui]: https://python.org.br/instalacao-windows/
[Linux aqui]: https://python.org.br/instalacao-linux/
[no Windows]: https://youtu.be/WWWoFsaLjK8
[no Linux]: https://www.youtube.com/watch?v=zianAMUWlYA
