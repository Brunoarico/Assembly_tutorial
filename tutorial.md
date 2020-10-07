# Assembly

Assembly é uma linguagem de mnemônicos ou seja, um conjunto de símbolos (MOV, JMP, JE, etc) que encodam diretamente binário, sendo assim não existe um processo de "compilação" como conhecemos para linguagens usuais como C ou java e sim um processo direto de tradução desses símbolos para binários.

Como estamos trabalhando na ordem dos binários propriamente ditos isto está na camada do software mais próxima ao hardware, assim esse processo de tradução  dependera para qual arquitetura do processador que estamos utilizando.

Um binário compilado para uma máquina de 64 bits não serve em uma máquina de 32 bits, assim como um binário compilado em arquitetura i386 não funciona em um processador de arquitetura arm.

Portanto sempre que vamos compilar um binário temos que estudar para qual máquina iremos gerar. Para criterio do projeto teremos como foco as máquinas de 32 bits.

Um fato curioso e que usarmemos nesse ep é que binários compilados para máquinas de 32 bits podem rodar em maquinas de 64 bits, claro que utilizando desses 64 bits apenas 32 bits.
A maioria dos computadores modernos utiliza arquitetura de 64 bits, e muito provavelmente a máquina que estamos usando agora é 64 bits e teremos que, por meio dela, compilar um arquivo para arquitetura 32 bits. Para isso usaremos bibliotecas dos sistemas 32 bits junto ao GCC para gerar nossos arquivos, como a **gcc-multilib**.

Para instalar, em linux:

~~~ sh
sudo apt-get install gcc-multilib

~~~

# Assembler

Quando falamos em linguagem Assembly utilizamos um programa que fica responsável por gerar o nosso binário a partir dos mnemônicos da linguagem, como dito anteriormente esse processo não é uma compilação e, como o proprio nome diz, do inglês "Assembler" = montagagem, esse programa se chama **assembler**.
Existem diversos assemblers entre eles o da intel (NASM) e o da GNU (GAS), ambos utilizam diferentes memônicos para o processo de montagem, particularmente recomendo o NASM. (Que foi o que utilizei para estudar as possibilidades do projeto)

# Makefile

Makefiles são arquivos de configuração, essencialmente são semelhantes a programas tipo shell (os famosos aquivos .sh), porém com uma sintaxe mais bem organizada e com um formato mais "inteigente de serem implementados" do ponto de vista de quem esta criando esse arquivo.

O make file é um arquivo composto de diretivas tais como:

~~~
clean:
  rm *.o

diz_oi: clean
  echo "ola"

~~~

Cada uma dessas diretivas executa as ações que estão sob as respectivas identações, dessa forma ao digitarmos no terminal:

~~~ sh
> make clean
~~~

Todos os arquivos que batem com a expressão regular *.o serão apagados do diretorio que contem o makefile.

E se executarmos:
~~~ sh
make diz_oi  
~~~
Inicialmente ele limpa todos os arquivos *.o* do diretorio e então printa "ola"

# Parte 1

## Números Primos

Entre os métodos mais simples de se calcular se um número é primo ou não podemos utilizar o de ir testando a divisibilidade do número a ser testado por todos os números inferiores a ele (começando esse teste a partir do 2, pela indefinição da divisão por 0 e por 1 sempre dividir qualquer número). Inclusive, dessa forma em nosso programa assumimos que 0 e 1 são primos.

Por curiosidade esse algoritmo é relativamente ok em tempo de execução, sendo O(n), com tempo proporcional a ordem de grandeza do primo a ser testado. Existe um método de otimização para esse algoritmo que o deixa proporcional a O(sqrt(n)), que afirma que se *a⋅b=N* onde *1<a≤b<N*
então *N=ab≥a²* assim *a²≤N* então *a≤sqrt(N)*. Segundo essa coisa podemos garantir que se formos até a raiz quadrada do numero N testado e não achamos nenhum divisor então N é primo.
Digo que é por curiosidade aqui pois a raiz quadrada é um tanto quanto complicada de ser implementada em assembly, portanto usaremos o algoritmo que é linear. Caso se sintam encorajados de implementar a raiz em assembly é um otimo exercício para entender como a máquina opera mais a fundo.

## C
Um arquivo em C que pode ser tomado como base para a implementação é o seguinte:

``` C
#include <stdio.h>


int is_prime(int n) {
  //Colocar codigo aqui
}

int main(int argc, char *argv[]) {
  //colocar codigo aqui
  return 0;
}

```

Prestar atenção em algumas coisas: a ordem de declaração das funções importa, logo se a **main** chama a **is_prime** a **is_prime** deve ser declarada antes da **main**.

Outro ponto importante a se notar é em relação as bibliotecas declaradas. A stdlib é a biblioteca de funções padrão de entrada e saida do C, logo precisaremos declará-la no início. Ela nos fornecerá a função **printf** para imprimir coisas no terminal.

Outro ponto são os argumentos passados via terminal; quem nos permite acesso a eles por meio do programa em C é o **argc** e o **argv**. O **argc** armazena a quantidade de argumentos declarados **considerando o nome do programa chamado como um argumento**, logo se temos:

~~~ sh
./sieve 101
~~~

**argc** armazenará o valor 2, e assim **argv** armazenará: [['s','i','e','v','e','\0'], ['1', '0', '1','\0']]

 Logo tudo que é lido do terminal vem na forma de string inclusive o primo lido. Para passarmos para a função **is_prime** necessitamos convertê-lo para int.
 Recomendo fortemente que essa função seja implementada 'bare metal', sem bibliotecas, pois mais para frente não poderemos utilizar nenhuma biblioteca.

 Um ótimo esqueleto para começo do código desta parte do EP é esse:

 ``` C
 #include <stdio.h>


 int is_prime(int n) {
   //Colocar codigo aqui
 }

 int char2int (char *array) {
   //Colocar codigo aqui
 }

 int main(int argc, char *argv[]) {
   //colocar codigo aqui
   return 0;
 }

 ```

 Esse arquivo será compilado via linha de código no terminal utilizando o compilador de arquivos C, o GCC. Recomendo utilizar as seguintes flags de compilação:

~~~ sh
 -O3 -W -Wall -ansi -pedantic
~~~

Consultem a documentação do gcc ou o google (recomendo essa opção) para entender o significado de cada uma delas.


 Um makefile que pode servir de base para esta parte é recomendado ter uma linha de chamada que convencionalmente se chama: **all**, que realiza todos os processos necessários para a compilação, limpeza dos arquivos objeto que serão gerados deixando apenas o executavel e os arquivos fonte no diretorio.
 **clean** que faz a limpeza de todos os arquivos objetos e executaveis no diretorio.
 **sieve** que chama o GCC e compila de fato os arquivos.

 No makefile podemos definir variaveis globais usando, por exemplo:

 ~~~ sh

 CFLAGS= -O3 -W -Wall -ansi -pedantic

 all: sieve

 sieve: sieve.c
 	$(CC) $(CFLAGS) -o sieve sieve.c

 clean:
 	rm -rf *.o sieve

 ~~~

 este é um otimo inicio para um makefile


 # Parte 2

Aqui começa realmente a implementação de coisas em Assembly. Nesta parte temos que entender como que o GCC compila as coisas.
O compilador basicamente converte um código C em um codigo de mais baixo nivel, o assembly, o qual internamente usa um assembler para gerar os binarios que correspondem aos dados que estão encodados no programa. Este arquivo gerado é o que chamamos de arquivo objeto. Um programa é composto de um ou mais arquivos objetos, como por exemplo as bibliotecas que o compoe tambem tem arquivos objetos correspondentes. Assim quando o compilador compila, ate esse momento temos uma sequência de arquivos objeto (.o) que precisam ser ligados, esse processo de ligação chama-se linkage. E quem cuida disso é um utilitario que o próprio GCC vai gerenciar que é o **ld**.

Ou seja quando executamos a linha de compilação:

~~~
gcc -o sieve.c sieve
~~~

O arquivo *sieve.c* sera convertido para linguagem de máquina (Assembly) e então para binário que estará disponível em um *sieve.o*, esse sieve.o se ligara aos *.o* das bibliotecas padrão e então gerara o binario do arquivo executabel *sieve*. O GCC faz esse processo ser basicamente transparente, mas quando trabalhamos com arquivos *.s* necessitamos fazer esse processo manualmente.

Suponha que tenho o seguinte arquivo em C, *soma.c*:

~~~ C
#include <stdio.h>

int add(int a, int b);

int main(int argc, char **argv) {
  printf("%d\n", add(2, 6));
  return 0;
}

~~~

A função add tem sua declaração feita, mas não esta implementada, ao tentarmos compilar esse arquivo com o GCC ele dira que nós não temos nada chamado **add** pra ele colocar no lugar ali. Ou seja, no arquivo *.o* gerado ele tera a declaração mas não a referência para o conteudo do que essa função faz. O que podemos fazer é um "puxadinho no arquivo" incluindo um *.o* na compilação que possua o que essa função faz. Ai que entra o arquivo *.s*.

Suponha que temos o arquivo *sub.s*:
(não nos atentemos as funcionalidades dos mnemônicos do assembly ainda)

~~~
global add

section .data

section .text

add:
    push ebp                        
    mov ebp, esp                    

    mov   eax, [esp+4]   
    add   eax, [esp+8]

    pop ebp
    ret
~~~

Perceba que temos uma tag chamada *add* no arquivo e ela esta identificado no começo como global, e fiz questão de chamar o arquivo de *sub* para reforçar que o que vale é o conteudo e não o nome do arquivo. Inclusive ele poderia se chamar *bsdbjfkbs.ahsdahkl* que daria tudo certo, neste nível da computação apenas os bits importam.

Agora, essencialmente, temos tudo que necessitamos para o programa compilar.
Inicialmente temos que montar o nosso arquivo e quem fara essa tarefa para gerar o nosso *sub.o* é o NASM:

~~~ sh
nasm -f elf sub.s
~~~

Automagicamente será gerado um arquivo *sub.o*
(Outra curiosidade que esse processo de tradução dos mnemonicos pode ser executado manualmente para arquiteturas mais simples como 8 bits, recomendo para quem quiser entender onde o software acaba e o hardware começa a fazer esse processo. Existem emuladores de cpu 8bits como a do z80 para testar o código)

Agora podemos fazer a ligação de todas as partes e para isso podemos usar a linha:

~~
gcc -o soma add.o soma.c -m32
~~

Na execução desta linha temos que o GCC verifica que o arquivo *sub.o* já é um objeto, e converte o *soma.c* para *soma.o*, e os linka, gerando o output de nome soma.
A flag **-m32** serve para identificar que a arquiteturapara a qual sera compilada é de 32 bits, e a flag **-o** é para definir o nome do output que será *soma*.  

Para executar o binário basta de um terminal executando dentro do diretorio em questão:

~~
./soma
~~

 Esse processo aqui apresentado é analogo ao que será executado com o *is_prime.s* e o *sieve_asm.c*. Disuctiremos detalhes da implementação em assembly a seguir.
 No makefile certifique-se de seguir a sequência correta de compilação. Primeiro tem que ser feito o *is_prime.o* para aí sim compilar o *sieve_asm.c*.
