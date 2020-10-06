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

diz_oi:
  echo "ola"

print_file: teste.txt
  cat teste.txt
~~~

Cada uma dessas diretivas executa as ações que estão sob as respectivas identações, dessa forma ao digitarmos no terminal:

~~~ sh
> make clean
~~~

Todos os arquivos que batem com a expressão regular *.o serão apagados do diretorio que contem o makefile.

Quando precisamos passsar um arquivo especifico para umas diretivas operar, como é o caso do cat ali no exemplo, passamos esse arquivo na frente da declaração da diretiva como no exemplo.

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
