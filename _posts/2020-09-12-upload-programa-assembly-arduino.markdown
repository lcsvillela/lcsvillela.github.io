
---
layout: post
title:  "Upload de um programa em assembly para o Arduíno"
date:   2020-08-04 16:32:34 -0300
categories: assembly arduino atmel328p
image: https://user-images.githubusercontent.com/23728459/117732812-e47d7400-b1c6-11eb-9f8e-e72afbea897e.png
---

Fiz um código para o processador Atmel328p e não achei nenhum conteúdo de qualidade em português de como afinal realizar o upload para o processador usado no arduíno que tenho, portanto escrevi um simples shell script que resolveu o problema.

Para fazer manualmente, foi utilizado as ferramentas avra e avrdude, pois não conseguia executar os programas proprietários da Atmel por algum motivo (máquina virtual do windows). Então decidi fazer tudo na mão!

Em um arquivo texto, criei o seguinte código que faz um led piscar no meu arduíno com o 328p

{% highlight assembly %}
.device atmega328p ; especifica que o dispositivo é o atmega328p
.org 0x0000 ; determina o início do bloco de memória

.equ DDRB = 4 ; determina a porta de saída
.equ PortB = 5 ; determina a porta de saída

rjmp main ; pula pro label main:
 
main:
ldi r16, 0xFF ; aqui, pegamos o número hexa 0xFF que é 255 em decimal e
;11111111 em binário, ou seja, todos os bits estão ativados e
;salvamos essa informação no registrador r16
out DDRB, r16 ; aqui determina o pino utilizado como saída (1 saída e 0 entrada)
loop:
sbi PortB, 0x4 ; seta como 1 o pino 0x4 da PortB
cbi PortB, 0x4 ; seta como 0 o pino da 0x4 da PortB
rjmp loop
{% endhighlight %}

Depois que fizemos esse código simples, podemos transformar ele usando o avrdude e aí teremos o nosso .hex que poderemos fazer o upload do código para o atmel no nosso arduíno.

Em distribuições baseadas em Debian, podemos instalar usando o apt:

{% highlight bash %}
sudo apt install avrdude avra
{% endhighlight %}

Depois da instalação você pode criar um arquivo com os comandos abaixo, onde o avra é o assembler que vamos utilizar e o avrdude faz o upload, no caso estou usando um chip da atmel328p e para especificar isso pro avrdude é a opção -p, recomendo o manual do [avra] e do [avrdude].

{% highlight bash %}
#! /bin/bash
avra -fI $1 $2
NOME=$(echo $1 | cut -d . -f 1)
avrdude -c arduino -p m328p -P /dev/$2 -e -F -b 115200 -U flash:w:$NOME.hex
rm *cof* *obj* *hex
{% endhighlight %}

O que este código acima faz? Basicamente o $1 é uma variável ambiente, onde temos uma fila, no caso da primeira linha o $0 é a string avra, o $2 é -fI e o $3 é o $1, porém, quando executamos o script usando o "./script.sh arquivo.asm" por exemplo, é criada uma nova fila e a $1 nesse caso é o arquivo.asm, que será transformado em binário. Se quiser saber mais sobre shell script, recomendo o livro [Shell Script Profissional]

Após de salvar o arquivo, dê as permissões para que seja executado com os comandos e em seguida execute:

{% highlight bash %}
chmod +x nome-arquivo.sh
./nome-do-arquivo.sh programa.asm /dev/ttyUSB0
{% endhighlight %}

Observação: Veja o endereço do seu arduíno usando o Arduíno IDE, no meu caso era /dev/ttyUSB0, mas o seu pode ser diferente!




[avrdude]: https://linux.die.net/man/1/avrdude
[avra]: https://linux.die.net/man/1/avrdude
[Shell Script Profissional]: https://s3.novatec.com.br/capitulos/capitulo-9788575221525.pdf
