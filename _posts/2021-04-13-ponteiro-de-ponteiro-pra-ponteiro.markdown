---
layout: post
title:  "Ponteiros e indireção múltipla"
date:   2021-04-13 19:32:34 -0300
categories: programacao_C
image: https://user-images.githubusercontent.com/23728459/117733288-b51b3700-b1c7-11eb-865b-9217721f558f.jpg

---

*Aviso: Você precisa saber o básico de C para entender este texto, 
várias pessoas tem dificuldade em ponteiros e espero que isso seja útil,
 recomendo os livros [C Completo e Total]{:target="_blank"},
[Curso de programação em C da UFMG]{:target="_blank"},
 [O Fantástico mundo da linguagem C]{:target="_blank"}, se quiser se aprofundar,
 siga com o [curso da USP]{:target="_blank"}*

![ponteiro](https://user-images.githubusercontent.com/23728459/117733288-b51b3700-b1c7-11eb-865b-9217721f558f.jpg){: style="float: left" class="img-responsive" height="60%" width="60%"} Um assunto muitas vezes ignorado na graduação ou mesmo em livros que passamos batido, é a indireção múltipla. 
Um recurso da linguagem C/C++ que permite 
criarmos até mesmo estruturas de dados 
complexas que sejam dinâmicas ou 
códigos mais enxutos e elásticos, 
onde podemos fazer vários formatos na estrutura de dado,
algo bastante abstrato.

Um ponteiro é um tipo de variável que não aponta
 para o valor de uma variável em si, mas sim para um endereço de memória.
O mais interessante, é que o ponteiro é uma variável que tem um endereço, então
um ponteiro de ponteiro armazena endereços de ponteiros e o 
limite disso depende do [compilador]{:target="_blank"} que você está utilizando.

Porém, a ideia central aqui é criar um ponteiro de ponteiro para um ponteiro, ou seja, criar mais níveis de apontamento,
contribuindo para que seu código fique ainda mais dinâmico.

{% highlight C linenos%}
#include <stdlib.h>
#include <stdio.h>

void  main (){

    char *ponteiro = NULL;
    char **ponteiro_de_ponteiro = NULL;
    char ***ponteiro_de_ponteiro_para_ponteiro = NULL;
    char letra = 'L';

    ponteiro = &letra; //o parâmetro & indica que o que será passado é
    //o endereço da variável letra e não seu conteúdo
    ponteiro_de_ponteiro = &ponteiro;
    ponteiro_de_ponteiro_para_ponteiro = &ponteiro_de_ponteiro;

    printf ("A letra é: %c\n", ***ponteiro_de_ponteiro_para_ponteiro);
}

{% endhighlight %}

Quando executamos esse código, a saída resultante é "A letra é: L", porém não utilizamos a referência da variável letra,
mas sim o endereço físico onde a variável está armazenada, algo que não é muita novidade, porque se vocẽ está aqui,
 provavelmente já conhece ponteiros. Este é um exemplo básico de indireção múltipla, onde usamos ponteiros para referenciar
  outros ponteiros. Mas olhando desta forma, parece ser algo até mesmo trivial, não é mesmo?
  
<h3>Apontando pra ponteiros em massa - matrizes</h3>

Começa a ficar interessante, quando começamos a pensar na possibilidade de apontar para outros ponteiros em massa, sendo
assim, poderíamos consumir quanta memória desejássemos e ainda por cima, teríamos uma estrutura onde poderíamos organizar
nossas informações! Aqui um outro exemplo:

{% highlight C linenos%}
#include <stdio.h>
#include <stdlib.h>

void main (int argc, char *argv[]){

    char letra = 'L';
    char **ponteiro_de_ponteiro = NULL; //precisamos apenas de um
    //ponteiro para ponteiro nesse caso
    int tamanho, i, j;

    tamanho = atoi(argv[1]);
    
    //Assim podemos armazenar ponteiros de ponteiros
    ponteiro_de_ponteiro = (char**)malloc (sizeof(char*)*tamanho);

    for (i = 0; i < tamanho; i++){
    	//assim criamos blocos de caractere dentro dos ponteiros
        ponteiro_de_ponteiro[i] = (char*) malloc (sizeof(char)*tamanho);
    }

	//com o auxílio desse for, conseguimos preencher toda a matriz
    for (i = 0; i < tamanho; i++){
        for (j = 0; j < tamanho; j++){
         ponteiro_de_ponteiro[i][j] = letra;
        }        
    }
    
    //Aqui é um teste de prova de que está preenchido
    for (i = 0; i < tamanho; i++){
        for (j = 0; j < tamanho; j++){
            printf ("%c", ponteiro_de_ponteiro[i][j]);
        }
    }
}

{% endhighlight %}
<h3>Ponteiros de ponteiros para ponteiros - N dimensões</h3>

Um professor numa aula da graduação, passou o seguinte problema para a sala: A gente deveria criar
um programa que armazenasse o o nome do aluno, o número de matrícula, a turma, três provas e o ano
da turma, além disso, calcular uma média simples de cada aluno, a média da turma e o comparativo
entre os anos da turma, tudo isso, armazenando 5 anos anos de turma com no mínimo 3 turmas para cada ano.

Diante deste problema, comecei a pensar um jeito mais simples de resolvê-lo, não queria fazer uma coisa simples
e trabalhosa, queria fazer uma coisa que fosse mais complexa de se pensar, mas ao mesmo tempo mais fácil de
implementar. Então, recorri à indireção múltipla! Você pode ver o [código completo aqui]{:target="_blank"}.

Porém, podemos ver também o seguinte exemplo de forma mais simples:

{% highlight c linenos %}
#include <stdio.h>
#include <stdlib.h>

int main (int argc, char *argv[]){

    char letra = 'L';
    char ***ponteiro_de_ponteiro_para_ponteiro = NULL;
    int tamanho, i, j, k;

    tamanho = atoi(argv[1]);
    ponteiro_de_ponteiro_para_ponteiro = \
        (char***)malloc (sizeof(char**)*tamanho);

    for (i = 0; i < tamanho; i++){
        ponteiro_de_ponteiro_para_ponteiro[i] = \
            (char**) malloc (sizeof(char*)*tamanho);
        for (j = 0; j < tamanho; j++){
            ponteiro_de_ponteiro_para_ponteiro[i][j] = \
                (char*) malloc (sizeof(char)*tamanho);
        }
    }

    for (i = 0; i < tamanho; i++){
        for (j = 0; j < tamanho; j++){
            for(k = 0; k < tamanho; k++)
         ponteiro_de_ponteiro_para_ponteiro[i][j][k] = letra;
        }        
    }
    
    for (i = 0; i < tamanho; i++){
        for (j = 0; j < tamanho; j++){
            for (k = 0; k < tamanho; k++)
            printf ("%c", ponteiro_de_ponteiro_para_ponteiro[i][j][k]);
        }
    }
    printf ("\n");

    return 1;
}

{% endhighlight %}

Mas afinal, o que está acontecendo? Na linha 7, quando declaramos um ponteiro de ponteiro para ponteiro, usamos \*\*\*, o que
significa que é uma informação tridimencional, a quantidade de * significa exatamente isso! Depois, na linha 15 e 18,
preenchemos o espaço de ponteiro para ponteiro e ponteiros, observe que o primeiro armazenamento faz uma conversão
usando (char\*\*\*), depois (char\*\*) e por fim (char\*), isso mostra qual tipo de informação será armazenada em cada bloco.
Então, podemos nos perguntar:

_Quantos níveis a informação que quero armazenar tem?_

Depois disso, podemos criar blocos de memória com o design que quisermos e o melhor de tudo: eles serão dinâmicos,
não precisamos ficar ajustando o código para ler algo com tamanho diferente. Uma das situações que isso pode se colocar
de forma muito útil, é o armazenamento em massa de ponteiros para arquivos de um sistema operacional, para [estruturas de
dados]{:target="_blank"} (lista simplesmente encadeada, árvore binária, árvore B ou afins), ou mesmo, pasmem: armazenar o endereço de funções
do seu programa em C :)

<h3>Conclusão</h3>

A utilização de indireção múltipla, muitas vezes tratada como nota de rodapé ou simplesmente ignorada, é uma função da
linguagem C/C++ que possibilita organizar e fazer coisas muito belas,
como por exemplo organizar dados de forma multidimensional.

[código completo aqui]: https://github.com/lcsvillela/minhas-atividades-ibilce/blob/main/EDI/Exercicio%20turmas%20escola.c
[Curso de programação em C da UFMG]: http://www2.dcc.ufmg.br/disciplinas/pc/source/introducao_c_renatocm_deeufmg.pdf
[O Fantástico mundo da linguagem C]: https://fiorix.files.wordpress.com/2014/04/o-fantc3a1stico-mundo-da-linguagem-c.pdf
[curso da USP]: https://www.ime.usp.br/~pf/algoritmos/
[C Completo e Total]: https://www.inf.ufpr.br/lesoliveira/download/c-completo-total.pdf
[compilador]: https://pt.wikipedia.org/wiki/Compilador
[estruturas de dados]: https://pt.wikipedia.org/wiki/Estrutura_de_dados
