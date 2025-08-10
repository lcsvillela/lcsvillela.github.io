---
layout: post
title:  "Criando um robô para o seu Instagram"
date:   2021-01-13 16:32:34 -0300
categories: instagram 
image: https://github.com/user-attachments/assets/e58f4722-3014-4239-a632-1ad52b1146a8
thumbnail: https://github.com/user-attachments/assets/e58f4722-3014-4239-a632-1ad52b1146a8
---

Todo ano ocorre o evento [EuroPython]{:target="_blank"}, sendo realizada a apresentação de projetos e mudanças na linguagem [Python]{:target="_blank"}. Em uma dessas palestras, estava o [Tim Großmann]{:target="_blank"} falando sobre o [Instapy]{:target="_blank"} (você pode ver o [projeto aqui]{:target="_blank"}), que consiste em um [framework]{:target="_blank"} para a automação de ações nas redes sociais. O fato é que as redes sociais são justamente o contrário do nome: são redes antissociais, que causam [problemas]{:target="_blank"} e [mais problemas]{:target="_blank"} psicológicos significativos. Ou seja, se podemos automatizar nossas ações, podemos nos afastar de tais problemas e, de quebra, conseguir seguir pessoas de verdade.

Großmann fez o framework de modo a toda ação ser aleatória, para que você não precise se preocupar em alterar as frequências dessas ações, por ele ser uma imitação humana. Você pode ler [este artigo]{:target="_blank"} sobre como ele conseguiu 2500 seguidores em 6 meses - no meu caso, consegui 800 utilizando apenas meu notebook.

<h2>Preparando o ambiente</h2>

Para começar, é necessário instalar o Python em sua máquina. Para isso, entre no site oficial e faça o download para o seu sistema operacional. Você pode ver como instalar no [Windows aqui]{:target="_blank"} e no [Linux aqui]{:target="_blank"}.

Depois de instalar o Python, abra o terminal do Windows ou do Linux e digite o seguinte comando:

{% highlight python %}
pip install instapy
{% endhighlight %}

Depois disso, você vai precisar instalar o Firefox e o GeckoDriver. Veja aqui as instruções [para Windows]{:target="_blank"} e [Linux!]{:target="_blank"}

<h2>Criando um robô simples</h2>

O Instapy é extramente completo. Podemos, por exemplo, seguir perfis que seguem perfis; seguir perfis que são seguidos por perfis; seguir quem curtiu alguma publicação específica e muito mais! Você pode ver a documentação para saber mais [clicando aqui]{:target="_blank"}. A ideia é criar um robô simples, que siga os perfis que seguem um perfil determinado.

Primeiro, precisamos importar as bibliotecas do Python. Para isso, abra um editor de texto qualquer - se você estiver no Windows, abra o Notepad e se estiver no Linux, abra o Vim ;)

Cole no arquivo:

{% highlight python %}
from instapy import InstaPy as insta
from instapy import Settings as settings
from instapy import smart_run
from time import sleep
{% endhighlight %}

Até aqui, apenas importamos as bibliotecas necessárias para iniciarmos a programação do nosso bot. Agora é onde começamos a realizar a mágica!

Depois disso, nosso próximo passo será fazer o login em nossa conta no Instagram. Para isso, precisamos determinar os parâmetros de **username**, **password** e **headless_browser**. Este último define se o seu navegador ficará visível para você durante as ações. Como eu prefiro que não, coloquei True. Se você quiser ver seu navegador realizando as ações, coloque **False**.

{% highlight python %}
settings()
contador = 4
sessao = insta(username="SeuUserSem@"
               password="SuaSenha"
               headless_browser=True)
sessao.login()
{% endhighlight %}

Aqui temos a configuração do tipo de usuários que não queremos seguir: **skip_no_profile_pic** define se vamos ou não
seguir usuários que estão sem foto; **no_profile_pic_percentage** define quantas vezes isso acontecerá - no caso, 100% das vezes.

{% highlight python %}
sessao.set_skip_users(skip_no_profile_pic=True,
                      no_profile_pic_percentage=100,
                      skip_private=True,
                      private_percentage=100)
{% endhighlight %}

Agora, a nossa ideia é fazer a mesma tarefa repetidas vezes, usando a estrutura de decisão while - que, no caso,
sempre executa o código que pertence a ela (isso é definido pelo espaçamento no Python, diferentemente de C ou Java).

Nesse sentido, o comando **sessao.follow_user_followers** primeiro chama a nossa seção definida com nossas credenciais e depois chama o método **follow_user_followers**, que faz com que sigamos usuários que seguem os perfis citados.

No caso, coloquei dois perfis que são referência de conteúdo técnico brasileiro para mim: [PizzaDeDados]{:target="_blank"} e [MenteBinaria]{:target="_blank"}. Logo depois, definimos com **amount** a quantidade de perfis que vamos seguir. Com **randomize** definimos que a ação não será "muito" previsível, dando a impressão que somos humanos.

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

Após isso, o programa está pronto e você só precisará salvar o arquivo. Agora, o pulo do gato é colocarmos isso para ser executado automaticamente agendando tarefas [no Windows]{:target="_blank"} e [no Linux]{:target="_blank"}.

Coloque o agendamento para ocorrer apenas uma vez ao dia em seu computador, pois seguir mais do que 40 pessoas por dia pode fazer você sofrer com penalidades na plataforma do Instagram.

<h2>Conclusão</h2>

A automação pode nos livrar de uma tarefa simples e tediosa, que é seguir perfis no Instagram. Esse é um dos vários jeitos de impulsionar um perfil, tornando-se importante quando você é um profissional de social media com muitos perfis de Instagram para gerenciar, por exemplo.

Se esse for o seu caso, fica o desafio: Use um programa pra acessar vários perfis automatizados. Assim você pode deixar o seu cliente mais satisfeito, pois os usuários que deram o "seguir de volta" existem de verdade e ficarão por tempo indeterminado te seguindo, sendo muito mais qualitativo do que entregar um panfleto de propaganda na rua, não é mesmo?

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
[para Windows]: https://take.net/blog/take-test/instalacao-geckodriver-driver-para-abrir-o-firefox-no-selenium
[Linux!]: https://medium.com/beelabacademy/baixando-e-configurando-o-geckodriver-no-ubuntu-dc2fe14d91c
[clicando aqui]: https://instapy.org/settings/
[PizzaDeDados]:https://www.instagram.com/pizzadedados/
[MenteBinaria]: https://www.instagram.com/mentebinaria_/
