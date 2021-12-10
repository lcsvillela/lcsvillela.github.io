---
layout: post
title:  "Programando com classe sem smokin - Parte 1"
date:   2021-12-09 22:00:34 -0300
categories: python java oop
description: O que você pensa quando vê uma cadeira? Que tal construir uma usando Python e Java?
image: https://user-images.githubusercontent.com/23728459/145310353-03522411-917c-455f-892e-8576e342ec18.png
thumbnail: https://user-images.githubusercontent.com/23728459/145310541-9e31aae1-2d2b-4f83-8495-d481c928bf40.png
---


![ferramentas](https://media2.giphy.com/media/f51BBzyfFrEPK/giphy.gif?cid=ecf05e473s3f9m42ya138qmkuqwf80t87tfzkar8ss5mfo4l&rid=giphy.gif&ct=g){: style="float: left; margin-right: 1em;" class="img-responsive" height="40%" width="40%"}

Uma vez em uma aula de literatura, um professor perguntou para mim
o que pensava quando olhava para uma cadeira, respondi que pensava em que
seu material é feito, de onde veio, qual parafuso usa, material do tecido
e outras coisas que continuei a falar por algum tempo, ele fez
uma cara de "Hmm" e perguntou para outra pessoa na sala que respondeu o que
ele queria "Penso em me sentar".
As aulas dele eram demais, mas acredito que ele não gostou muito de minha
resposta, porém a origem do meu pensamento sobre a cadeira não é algo
sem fundamento, muitas pessoas perdem com o passar do tempo esse modo de
ver as coisas, porém tudo pode ser modulado/fracionado/atomizado.
Fritjof Capra discorda e concordo em sua discordância, porém é o
pensamento cartesiano (Descartes: fundador da filosofia moderna) que desenha
a forma como compreendemos as coisas ou de como são impossíveis de serem
compreendidas, isso possibilitou a gente a
fazer fusão de átomos, acredito que prove algo, não é mesmo?

<h2> Por que orientado a objeto?</h2>

![comclasse](https://user-images.githubusercontent.com/23728459/145310353-03522411-917c-455f-892e-8576e342ec18.png){: style="float: left; margin-right: 1em;" class="img-responsive" height="40%" width="40%"}
O mercado foi tomado pela Programação Orientada a Objeto
(POO) e muitos odeiam esse paradigma, assim como existe o POO
que tem seus pilares e modo de ver o mundo, existem também outros, como
procedural e funcional. Mas a maior força de POO reside no reúso do software,
imagine só que triste, você é uma pessoa desenvolvedora e cria um codigo que faz
uma cadeira verde, depois cria outro código copiando e colando...uma cadeira
azul, depois você cria outro, mas nesse as mudanças são tantas que você recria
a cadeira do jeito que o cliente quer...ou pior ainda, é desejada uma cadeira
com acessórios adicionais, como porta copo, propulsor à base de neutrinos e
um ajustador de medidas...nesse momento você já deve ter desistido do projeto.
Tudo isso pode ser feito com POO, a flexibilidade do código é absurda e se
você quer adicionar algo no projeto, basta fazer o que é chamado de herança,
fazendo com que a classe resultante da classe origem tenha mais recursos que
a anterior, é a evolução em ação! Para você pensar se você precisa de OOP
ou quaisquer boas práticas em relação à produção de software, veja esse trecho
do livro de Wilson de Pádua "Engenharia de Software - Fundamentos, Métodos e
Padrões": 


*"A Engenharia de Software se preocupa com o software enquanto produto. Estão fora de seu escopo
programas que são feitos unicamente para diversão do programador. Estão fora de seu escopo também
pequenos programas descartáveis, feitos por alguém exclusivamente como meio para resolver um
problema, e que não serão utilizados por outros."*

<h2>Abstraindo uma cadeira</h2>

Pensemos em uma cadeira, ela tem características, certo? Mas apenas as
realmente essenciais, não tem parafuso, porque tem cadeira que é peça única ;)

Vamos listar algumas:
{% highlight nul %}
Material da estrutura
cor da estrutura
cor do estofado
peso máximo suportado
largura do assento
profundidade do assento
material do estofado
altura do assento
altura do encosto
altura do pé
largura do pé
largura do encosto
Marca
Ocupada
{% endhighlight%}
Depois de pegarmos todas essas características, temos que identificar o tipo
da variável, por exemplo:

Altura do assento é do tipo float, que são os números quebrados, por exemplo
1.556, agora sobre material...são coisas que devemos conhecer previamente,
nesse caso é interessante fazermos uma enumeração.
A estrutura que permite esse tipo de declaração tem em C, Python e Java,
sendo assim podemos enumerar as cores, materiais ou qualquer coisa que
quisermos listar e associar um número a isso.
Mas também podemos ter o tipo string, que é uma cadeia de caracteres, no caso
seria a *Marca*. Já o atributo *Ocupada*, é um booleano, permitindo verdadeiro
ou falso.

<h2>Criando a classe</h2>

Depois de termos esse pequeno exercício de abstrair uma cadeira, podemos criar
uma classe que crie cadeiras para nós!

<h4> Em Python</h4>

{% highlight python%}

class Cadeira():
    material_estrutura = "madeira"
    cor_da_estrutura = "vermelha"
    cor_estofado = "amarela"
    peso_maximo_suportado_kg = 120
    largura__assento_metros = 0.8
    profundidade_assento_metros = 0.4
    material_estofado = "tecido"
    altura_assento_metros = 0.45
    altura_encosto_metros = 0.7
    altura_pe_metros = 0.4
    largura_pe_metros = 0.1
    largura_encosto_metros = 0.1
    marca = "minha cadeira"
    ocupada = True
{% endhighlight %}

Via de regra, esta é uma classe em python que cria uma cadeira, com seus
atributos definidos, porém a principal ideia de uma classe é que sirva de
template(forma) para a instanciação (criação) de objetos, da forma que está,
temos um objeto que não é exatamente uma template, porque ele é fixo, para
alterar as propriedades da cadeira, temos que obrigatoriamente nesse caso
alterar o código contido na classe e isso joga no lixo o conceito de
programação orientada a objetos, um template deve ser ajustável!

<h4>Em Java</h4>
Seguindo a mesma ideia anterior:

{% highlight java %}
public class Cadeira{

    public String material_estrutura = "madeira";
    public String cor_da_estrutura = "vermelha";
    public String cor_estofado = "amarela";
    public Int peso_maximo_suportado_kg = 120;
    public Float largura__assento_metros = 0.8;
    public Float profundidade_assento_metros = 0.4;
    public String material_estofado = "tecido";
    public Float altura_assento_metros = 0.45;
    public Float altura_encosto_metros = 0.7;
    public Float altura_pe_metros = 0.4;
    public Float largura_pe_metros = 0.1;
    public Float largura_encosto_metros = 0.1;
    public String marca = "minha cadeira";
    public boolean ocupada = true;

}
   
{% endhighlight %} 
<h2>Fazendo um objeto mais flexível</h2>>
<h4>Em Python</h4>
Para podermos ter uma template de um objeto que possa ser moldado e não
rígido como os anteriores, precisamos de construtores! Eles basicamente são
métodos(pode-se entender método como função pertencente a uma classe)
que são executadas por padrão quando a classe está sendo instanciada
(se transformando em objeto), sendo assim, não precisamos alterar nada dentro
da classe para criar nosso objeto dinâmico.
{% highlight python %}
class Cadeira:
    def __init__(
        self,
        material_estrutura,
        cor_estrutura,
        cor_da_estrutura,
        cor_estofado,
        peso_maximo_suportado_kg,
        largura__assento_metros,
        profundidade_assento_metros,
        material_estofado,
        altura_assento_metros,
        altura_encosto_metros,
        altura_pe_metros,
        largura_pe_metros,
        largura_encosto_metros,
        marca,
        ocupada,
    ):
        self.__material_estrutura = material_estrutura
        self.__cor_da_estrutura = cor_da_estrutura
        self.__cor_estofado = cor_estofado
        self.__peso_maximo_suportado_kg = peso_maximo_suportado_kg
        self.__largura__assento_metros = largura__assento_metros
        self.__profundidade_assento_metros = profundidade_assento_metros
        self.__material_estofado = material_estofado
        self.__altura_assento_metros = altura_assento_metros
        self.__altura_encosto_metros = altura_encosto_metros
        self.__altura_pe_metros = altura_pe_metros
        self.__largura_pe_metros = largura_pe_metros
        self.__largura_encosto_metros = largura_encosto_metros
        self.__marca = marca
        self.__ocupada = ocupada
{% endhighlight %}

Este exemplo em Python carrega um punhado de informações novas, primeiro vamos
observar o *\_\_init\_\_*, ele é o método construtor da nossa função, observe que
ele recebe como parâmetro todos os dados necessários para a instanciação do
nosso objeto.
Outro elemento novo aqui é o *self*, muitos programadores iniciantes se perdem
nele, mas o conceito aqui é mais ou menos assim: Imagine uma distribuidora de
cargas, o nome da distribuidora é SELF e ela distribui água pelo SELF.AGUA,
suco pelo SELF.SUCO e coisa do gênero, então o *self* é um unificador de
atribuitos da classe onde os mesmos podem ser acessados em qualquer método.
Um detalhe pequeno, mas não menos importante, é o underlines (\_)
existente depois do self, isso não é capricho para deixar o nome mais legível,
quando utilizamos \_ no self, tornamos aquele atributo protegido, mas ainda é
acessível fora da classe. Se usarmos dois \_\_ como é o caso do nosso programa,
a situação muda! O atributo é acessível por outro nome!

{% highlight python %}
class Teste():
    self.__teste = 1
    self.teste2 = 2
{% endhighlight %}

depois de instanciar a classe Teste, podemos alterar o atributo *teste2* assim:

{% highlight python%}
teste = Teste()
teste.teste2 = 4
{% endhighlight %}

Mas o atributo *teste* não é acessível desse modo. Temos que fazer:

{% highlight python%}
teste._Teste__teste
{% endhighlight %}

Mas se o atributo é protegido, por que temos acesso e podemos alterá-lo? Isso é
uma questão filosófica da linguagem, seu criador não queria que existisse pedaços
inacessíveis, então em Python tudo é acessível de algum modo.

<h4>Objeto flexível em Java</h4>

{% highlight java %}
public class Cadeira{

    public String material_estrutura;
    public String cor_da_estrutura;
    public String cor_estofado;
    public Int peso_maximo_suportado_kg;
    public Float largura__assento_metros;
    public Float profundidade_assento_metros;
    public String material_estofado;
    public Float altura_assento_metros;
    public Float altura_encosto_metros;
    public Float altura_pe_metros;
    public Float largura_pe_metros;
    public Float largura_encosto_metros;
    public String marca;
    public boolean ocupada;
    
    public Cadeira(String material_estrutura,
                   String cor_da_estrutura,
                   String cor_estofado,
                   Int peso_maximo_suportado_kg,
                   Float largura_assento_metros,
                   Float profundidade_assento_metros,
                   String material_estofado,
                   Float altura_assento_metros,
                   Float altura_encosto_metros,
                   Float altura_pe_metros,
                   Float largura_encosto_metros,
                   String marca,
                   boolean ocupada){
                   
                   this.material_estrutura = material_estrutura;
                   this.cor_estofado = cor_estofado;
                   this.peso_maximo_suportado_kg = peso_maximo_suportado_kg;
                   this.largura_assento_metros = largura_assento_metros;
                   this.profundidade_assento_metros = profundidade_assento_metros;
                   this.material_estofado = material_estofado;
                   this.altura_assento_metros = altura_assento_metros;
                   this.altura_encosto_metros = altura_encosto_metros;
                   this.altura_pe_metros = altura_pe_metros;
                   this.largura_encosto_metros = largura_encosto_metros;
                   this.marca = marca;
                   this.ocupada = ocupada;
                   }

}
{% endhighlight %}
O *this* em Java é o *self* do Python, note que em java a gente pode explicitar no
código de forma verborrágica que um atributo ou um método é protegido, público, privado ou estático.

Desse modo, podemos definir diversas configurações possíveis para nossa cadeira,
sem precisar alterar nada dentro da classe, o que dificulta a manutenção do
programa com o passar do tempo e também a sua extenção.

<h2>Fragmentando as classes - criando peça por peça</h2>

Vamos fatiar nossa classe em três: pés, assento, encosto. Assim podemos criar
cadeiras de todo tipo, inclusive banquinhos (que não tem encosto) e juntar tudo
na classe cadeira, que poderá nos retornar tudo.

<h4> Python - fragmentado e flexível</h4>
{% highlight python %}
class Pe:
    def __init__(
        self,
        altura_pe_metros=0.5,
        largura_pe_metros=0.05,
        material_pe="madeira",
        quantidade_pe=4,
    ):
        self.__data = {}
        self.__data["altura_pe_metros"] = altura_pe_metros
        self.__data["largura_pe_metros"] = largura_pe_metros
        self.__data["material_pe"] = material_pe
        self.__data["quantidade_pe"] = quantidade_pe

    def get_data(self):
        return self.__data


class Encosto:
    def __init__(self, altura_encosto_metros=0.4, largura_encosto_metros=0.05):
        self.__data = {}
        self.__data["altura_encosto_metros"] = altura_encosto_metros
        self.__data["largura_encosto_metros"] = largura_encosto_metros

    def get_data(self):
        return self.__data


class Assento:
    def __init__(
        self,
        largura_assento_metros=0.5,
        profundidade_assento_metros=0.4,
        altura_assento_metros=0.5,
        material_assento="madeira",
        ocupado=True
    ):
        self.__data = {}
        self.__data["largura_assento_metros"] = largura_assento_metros
        self.__data["profundidade_assento_metros"] = profundidade_assento_metros
        self.__data["altura_assento_metros"] = altura_assento_metros
        self.__data["material_assento"] = material_assento
        self.__data["ocupado"] = ocupado

    def get_data(self):
        return self.__data


class Estofado:
    def __init__(self, material_estofado="espuma", cor_estofado="vermelho"):
        self.__data = {}
        self.__data["material_estofado"] = material_estofado
        self.__data["cor_estofado"] = cor_estofado

    def get_data(self):
        return self.__data


class Cadeira:
    def __init__(self):
        self._pieces = {}

    def add_piece(self, piece):
        self._pieces[piece.__class__.__name__] = piece

    def get_data(self):
        for piece in self._pieces.keys():
            print(self._pieces[piece].get_data())
{% endhighlight %}

Isso é uma composição, é um padrão de desenvolvimento bem básico,
existem outros muito interessantes como singleton, factory, observer...
cada um tem um propósito de existência.

<h4>Java - fragmentado e flexível</h4>

Para fazer o código mais ou menos equivalente em Java, vamos precisar
criar 5 arquivos, cada qual com o nome da classe correspondente no arquivo.

Arquivo Cadeira.java:

{% highlight Java %}
import java.io.*;

public class Cadeira{

    public static void main(String args[]){

        Estofado estofado = new Estofado("espuma", "vermelho");
        Pe pe = new Pe("metal", 1.0, 0.1, "vermelha");
        Encosto encosto = new Encosto("madeira", 1.0, 0.1, "vermelho");
        Assento assento = new Assento("amarelo", "madeira", 1.0, 0.4, 0.5, 120.0);
        System.out.println(estofado.get_data()+
                pe.get_data()+
                encosto.get_data()+
                assento.get_data());
       

    }

}

{% endhighlight %}

Arquivo Assento.java
{% highlight Java %}
public class Assento{

    public String material_estrutura;
    public String cor_da_estrutura;
    public Double peso_maximo_suportado_kg;
    public Double largura_assento_metros;
    public Double profundidade_assento_metros;
    public Double altura_assento_metros;

    public Assento(String cor_da_estrutura,
                   String material_estrutura,
                   Double altura_assento_metros,
                   Double profundidade_assento_metros,
                   Double peso_maximo_suportado_kg,
                   Double largura_assento_metros
                   ){
        this.material_estrutura = material_estrutura;
        this.altura_assento_metros = altura_assento_metros;
        this.profundidade_assento_metros = profundidade_assento_metros;
        this.largura_assento_metros = largura_assento_metros;
        this.peso_maximo_suportado_kg = peso_maximo_suportado_kg;
    }

    public String get_data(){
    
        return this.material_estrutura+"\n"+this.altura_assento_metros+"\n"+
            this.profundidade_assento_metros+"\n"+this.largura_assento_metros
            +"\n"+this.peso_maximo_suportado_kg;
    }
}

{% endhighlight %}


Arquivo Encosto.java
{% highlight Java %}
public class Encosto{

    public String material_estrutura;
    public String cor_da_estrutura;
    public Double altura_encosto_metros;
    public Double largura_encosto_metros;
    
    public Encosto(String material_estrutura,
                   Double largura_encosto_metros,
                   Double altura_encosto_metros,
                   String cor_da_estrutura){
        this.material_estrutura = material_estrutura;
        this.largura_encosto_metros = largura_encosto_metros;
        this.altura_encosto_metros = altura_encosto_metros;
        this.cor_da_estrutura = cor_da_estrutura;
    }

    public String get_data(){
        return this.material_estrutura+"\n"+this.largura_encosto_metros+"\n"
            +this.altura_encosto_metros+"\n"+this.cor_da_estrutura;
    }

}
{% endhighlight %}

Arquivo Estofado.java
{% highlight Java %}
public class Estofado{

    public String cor_estofado;
    public String material_estofado;

    public Estofado(String material_estofado,
                    String cor_estofado){
    
        this.material_estofado = material_estofado;
        this.cor_estofado = cor_estofado;
    }

    public String get_data(){
        return this.material_estofado +"\n"+ this.cor_estofado;
    }
}


{% endhighlight %}

Arquivo Pe.java
{% highlight Java %}
public class Pe{

    public Double largura_pe_metros;
    public Double altura_pe_metros;
    public String material_estrutura;
    public String cor_da_estrutura;

    public Pe(String material_estrutura,
              Double altura_pe_metros,
              Double largura_pe_metros,
              String cor_da_estrutura){
        this.material_estrutura = material_estrutura;
        this.altura_pe_metros = altura_pe_metros;
        this.cor_da_estrutura = cor_da_estrutura;
        this.largura_pe_metros = largura_pe_metros;
        
    }

    public String get_data(){
    
        return this.material_estrutura+"\n"+this.altura_pe_metros+
            "\n"+this.cor_da_estrutura+"\n"+this.largura_pe_metros;
    }
}




{% endhighlight %}


Desse modo nossa composição em Java para construir uma cadeira já está pronta! O
ideal é que realizemos, em qualquer linguagem, [composição ao invés de herança]{:target="\_blank"}
pois melhora a flexibilidade do código.

<h2>Conclusão :D</h2>

Programação Orientada a Objetos não é melhor ou pior do que os outros
paradigmas, é uma forma de ver o mundo, que pode beneficiar ou não
a solução de problemas. A principal vantagem desse paradigma, é o reaproveitamento
de código, imagine só ter que reescrever uma classe de um programa que faz
determinada tarefa...isso é desperdício, para isso sim utilizamos algum tipo de
polimorfismo (ou poliformismo, como alguns dizem).


[composição ao invés de herança]: https://en.wikipedia.org/wiki/Composition_over_inheritance

