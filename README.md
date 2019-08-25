---
Author:
- Amanda Luna
Bibliography:
- 'references.bib'
Date: Agosto 2019
Title: 'Apostila - Programação Concorrente'
---

# Apostila - Programação Concorrente


O que é programação concorrente ?
=================================

Os sistemas de computação modelam o mundo e este contém atores que
executam independentemente, mas se comunicam uns com os outros. Na
modelagem do mundo, muitas (possivelmente) execuções paralelas precisam
ser compostas e coordenadas, e é aí que entra o estudo da
concorrência.[@IntroConcurrency]

A concorrência nada mais é do que a coordenação e gestão destas linhas
independentes de execução[@IntroConcurrency] e pode ocorrer quando
várias cópias da mesma tarefa são executadas ao mesmo tempo, mas no
decorrer de sua execução, essas cópias se comunicam umas com as outras,
via memória compartilhada ou passagem de mensagens.[@WhatIsConcurrency]

Ou seja, programação concorrente fornece uma maneira de tornar eficaz
uso de sistemas paralelos e distribuídos que executam muitas tarefas
simultaneamente.[@StudentsArticle]

Um detalhe importante é que **não** devemos confundir concorrência com
paralelismo, pois concorrência é sobre *lidar* com muitas coisas de uma
só vez, já paralelismo é sobre *fazer* muitas coisas ao mesmo
tempo.[@ConcurrencyVsParallelism]

Threads
=======

Definição
---------

De forma simplista e direta, uma thread é uma linha (thread) de execução
de um processo [@Tanenbaum:2014] ou também pode ser vista como um
subprocesso de um processo. Como threads têm algumas das propriedades
dos processos, às vezes eles são chamados de *processos leves*.

Threads têm exatamente o mesmo espaço de endereçamento, o que significa
que elas também compartilham as mesmas variáveis globais. Tendo em vista
que toda thread pode acessar todo espaço de endereçamento de memória
dentro do espaço de endereçamento do processo, um thread pode ler,
escrever, ou mesmo apagar a pilha de outro thread,[@Tanenbaum:2014] o
que é considerado um grande problema. Para evitar isto, uma das
possíveis soluções é garantir **exclusão mútua**, que nada mais é do que
garantir que apenas uma thread entre nos espaços com recursos
compartilhados.

Estes espaços com recursos compartilhados são chamados de **regiões
críticas**, nas quais podem ocorrer **condições de corrida**. Estas
ocorrem quando múltiplas threads entram nesta região de forma
concorrente (ao mesmo tempo). O resultado final da execução da tarefa
pode ser afetado pelo fluxo de execução das threads.

Problemas que podemos ter
-------------------------

Seguem aqui alguns problemas que poderemos ter quando ocorrem condições
de corrida nas regiões críticas[@Problems]

-   Livelock

    -   Os livelocks ocorrem quando as threads são escalonadas, mas não
        estão fazendo progresso porque estão reagindo continuamente às
        alterações de estado uma da outra. A alta utilização da CPU sem
        nenhum sinal de trabalho real sendo feito é um sinal clássico de
        aviso de um livelock. Os Livelocks são incrivelmente difíceis de
        detectar e diagnosticar.

-   Deadlock

    -   Um deadlock ocorre quando duas ou mais threads esperam uma a
        outra, formando um ciclo e impedindo que todas elas avancem.
        Deadlocks geralmente são introduzidos por desenvolvedores
        tentando resolver condições da corrida.

-   Starvation

    -   Starvation é um atraso indefinido ou bloqueio permanente de uma
        ou mais threads em um aplicativo multithread. Threads que não
        estão sendo escalonadas para serem executadas, mesmo que não
        estejam bloqueadas ou esperando por qualquer outra coisa, estão
        em starvation.

Estados de uma thread
---------------------

A execução de uma thread pode passar por quatro estados: novo,
executável, bloqueado e encerrado. Um exemplo de como funciona a trocas
entre estes estados na JVM pode ser vista na figura abaixo.

![Estados de uma thread na
JVM[]{label="EstadosThread"}](ima004.jpg)

-   A thread está no estado de novo, quando é criada. Ou seja, quando é
    alocada área de memória para ela através do operador new.Ao ser
    criada, a thread passa a ser registrada dentro da JVM, para que a
    mesma posso ser executada.

-   A thread está no estado de executável, quando for ativada. O
    processo de ativação é originado pelo método *start()*. É importante
    frisar que uma thread executável não está necessariamente sendo
    executada, pois quem determina o tempo de sua execução é a JVM ou o
    S.O.

-   A thread está no estado de bloqueado, quando for desativada. Para
    desativar uma thread é necessário que ocorra uma das quatro
    operações a seguir:

    1.  Foi chamado o método *sleep(long tempo)* da thread;

    2.  Foi chamado o método *suspend()* da thread (método deprecado);

    3.  A thread chamou o método *wait()*;

    4.  A thread chamou uma operação de I/O que bloqueia a CPU;

-   Para a thread sair do estado de bloqueado e voltar para o estado de
    executável, uma das seguintes operações deve ocorrer, em oposição as
    ações acima:

    -   Retornar após o tempo especificado, caso a thread estiver
        adormecida;

    -   Retornar através do método *resume()*, caso a thread tiver sido
        suspensa (método deprecado);

    -   Retornar com o método *notify()* (ou *notifyAll()*), caso a
        thread estiver em espera;

    -   Retornar após a conclusão da operação de I/O.

-   A thread está no estado de encerrado, quando encerrar a sua
    execução. Isto pode acorrer pelo término do método *run()*, ou pela
    chamada explícita do método *stop()*. [@ThreadsStates]

Por agora não se preocupe tanto com essas nomenclaturas, detalharemos
elas na seção seguinte.

Introdução a Threads em Java
============================

Existem dois jeitos para se implementar thread em java:

1.  Derivar da classe *Thread* (extends)

2.  Implementar a interface *Runnable*

O segundo jeito é o mais recomendado, pois, ao estender *Thread*, cada
uma das suas threads tem um objeto exclusivo associado a ele, enquanto
implementando *Runnable*, muitas threads podem compartilhar a mesma
instância de objeto. Além disso, quando há necessidade de estender uma
superclasse, implementar a interface *Runnable* é mais apropriado do que
usar a classe *Thread*, pois podemos estender outra classe ao
implementar a interface para criar uma thread, mas, se apenas
esterdermos a classe *Thread*, não poderemos herdar de nenhuma outra
classe.[@RunVsTh]

Começando a trabalhar com threads (Java)
----------------------------------------

Como dito anteriormente, o melhor jeito para começar a brincar com
threads é criando uma nova classe que implementa a interface *Runnable*

Para implementar esta interface, só é necessário implementar um único
método, chamado *run()*, que será o que a sua thread irá executar.

Para criar uma thread, temos o seguinte trecho de código:

    Thread t = new Thread(Runnable target, String name);

Em que o parâmetro `Runnable target` se refere a classe que implementa
*Runnable* e o método *run()* e o parâmetro `String name` ao nome da
thread, que é opcional.

Temos como exemplo o programa apresentado a seguir

    public class ThreadsWorking implements Runnable {
        @Override
        public void run() {
            System.out.println("Minha thread executando");
        }
    }

    class ThreadsExample{
        public static void main(String[] args) {
            Thread minhaThread = new Thread(new ThreadsWorking());
            minhaThread.start();
        }
    }

*Exemplo 1*

A saída esperada para esta execução é apenas a linha

    Minha thread executando

Métodos para trabalhar com Threads (Java)
-----------------------------------------

Segue abaixo uma lista com alguns métodos disponíveis da classe
*Thread*: [@ThreadsStates]

-   `void run()` -- Deve conter o código que se deseja executar, quando
    a thread estiver ativa;

-   `void start()` -- Inicia a thread;

-   `void stop()` -- encerra a thread;

-   `static void sleep(long tempo)` -- deixa thread corrente inativa por
    no mínimo tempo milisegundos e promove outra thread;

-   `static void yield()` -- Deixa a thread em execução temporariamente
    inativa e, quando possível, promove outra thread de mesma prioridade
    ou maior;

-   `void join()` -- Aguarda outra thread para encerrar;

-   `boolean isAlive()` -- retorna true caso uma thread estiver no
    estado executável ou bloqueado. Nos demais retorna false;

-   `void wait()` -- Interrompe a thread corrente e coloca a mesma na
    fila de espera (do objeto compartilhado) e aguarda que a mesma seja
    notificada. Este método somente pode ser chamado dentro de um método
    de sincronizado;

-   `void notify()` -- Notifica a próxima thread, aguardando na fila;

-   `void notifyAll()` -- Notifica todas as threads.

Para mais detalhes e conhecer mais métodos disponíveis, você pode
consultar a
[documentação](https://docs.oracle.com/javase/7/docs/api/java/lang/Thread.html)
de *Thread*

Trabalhando com múltiplas threads (Java)
----------------------------------------

Para entendermos sobre múltiplas threads em java, observemos o código
abaixo.

    public class MyThread implements Runnable {
        private String id;
        Thread t;

        public MyThread(String id) {
            this.id = id;
            this.t = new Thread(this, id);
            t.start();
        }

        @Override
        public void run() {
            try {
                for (int i = 0; i < 3; i++) {
                    System.out.println("Thread " + id + " executando");
                    Thread.sleep(1000);
                }

            } catch (InterruptedException e) {
                System.out.println(id + "interrompida");
            }
            System.out.println("Thread " + id + " terminando");
        }
    }

    class ThreadsExample{
        public static void main(String[] args) {
            MyThread t1 = new MyThread("1");
            MyThread t2 = new MyThread("2");
            MyThread t3 = new MyThread("3");
        }
    }

*Exemplo 2*

O output desse programa foi:

    Thread 3 executando
    Thread 1 executando
    Thread 2 executando
    Thread 2 executando
    Thread 1 executando
    Thread 3 executando
    Thread 1 executando
    Thread 3 executando
    Thread 2 executando
    Thread 2 terminando
    Thread 1 terminando
    Thread 3 terminando

Neste programa, temos a execução de três threads simultâneamente, em que
cada uma delas faz um *sleep()* de 1000 milissegundos e imprime na tela
seu id.

Com o output, podemos perceber que as threads não executam de maneira
sequencial (1,2,3) e sim de maneira concorrente, em que a thread que
pega a CPU primeiro é a que será executada. Também podemos perceber que
o encerramentos destas também não segue um padrão e que não
necessariamente a primeira thread a ser executada é a primeira a
terminar.

Introdução a Threads em C
=========================

Para começar a implementar threads em C, basta inserir o seguinte
comando no seu código:

``` {style="CStyle"}
#include <pthread.h>
```

Começando a trabalhar com threads (C)
-------------------------------------

Para criar uma thread em C, temos o seguinte trecho de código:

``` {style="CStyle"}
int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start_routine)(void*), void *arg);
```

Em que,

-   `pthread_t *thread` - é onde você coloca a *pthreadt* que será
    inicializada;

-   `const pthread_attr_t *attr` - é onde se mantém dados que mudam o
    comportamento da thread. Se você não sabe o que está fazendo, o
    ideal é por *NULL* no lugar;

-   `void *(*start_routine)(void*)` - aqui é onde se coloca a função que
    a thread irá executar quando estiver pronta;

-   `void *arg` - aqui é onde se passam os argumentos para a função que
    vai ser executada (se tiver).

Com isso, poderíamos substituir os parâmetros para que ficassem assim:

``` {style="CStyle"}
int pthread_create(minha_thread, NULL, minha_funcao, argumentos_da_funcao);
```

Obs. Se você está com dificuldades em entender como funcionam os
apontadores, talvez
[isto](https://www.tutorialspoint.com/cprogramming/c_pointers.htm)
possa lhe ajudar.

Agora vejamos um programa semelhante ao *exemplo 1* feito em Java, agora
em C

``` {style="CStyle"}
#include<stdio.h>
#include<pthread.h>

void* run(void* args){
    int id = (int) args;
    printf("Ola, eu sou a thread que voce criou\nMeu ID e %d e eu estou executando\n", id);
    pthread_exit(NULL);
}

int main(int argc, char *argv[]){
    pthread_t minha_thread;
    pthread_create(&minha_thread, NULL, &run, (void *) 1);
    pthread_join(minha_thread, NULL);
    return 0;
}
```

*Exemplo 3*

O output para este programa é

``` {style="CStyle"}
Ola, eu sou a thread que voce criou
Meu ID e 1 e eu estou executando
```

Métodos para trabalhar com threads (C)
--------------------------------------

Segue abaixo ums lista com algumas funções básicas de *pthread*:[@CDocs]

-   `pthread_create(pthread_t *, const pthread_attr_t *,void *(*)(void *), void *)`
    -- Cria uma thread baseado nos parâmetros colocados;

-   `pthread_join(pthread_t thread, void **value_ptr)` -- Suspende a
    execução da thread corrente até que a thread passada como parâmetro
    termine. O segundo parâmetro contém o valor passado em
    *pthread_exit*;

-   `pthread_exit(void *value_ptr);` -- Termina a thread que a chamou e
    disponibiliza o valor de *\*valueptr* para qualquer chamada de
    função *join* que contenha a thread atual.

Agora algumas funções envolvendo mutex:[@CFunc]

-   `pthread_mutex_init(pthread_mutex_t *restrict mutex, const pthread_mutexattr_t *restrict attr);`
    -- Inicia o mutex que foi passado no primeiro parâmetro. O segundo
    parâmetro refere-se aos atributos do mutex, se você não sabe o que
    está fazendo, o ideal é por *NULL*;

-   `pthread_mutex_lock(pthread_mutex_t *mutex);` -- Bloqueia o mutex
    especificado no parâmetro. Se o mutex já estiver bloqueado por outra
    thread, a thread aguarda que o mutex se torne disponível. A thread
    que bloqueou o mutex torna-se sua atual proprietária e permanece
    como proprietária até que a mesmo thread o tenha desbloqueado;

-   `pthread_mutex_unlock(pthread_mutex_t *mutex);` -- Libera o mutex
    especificado no parâmetro. Se uma ou mais threads estiverem
    aguardando para bloquear o mutex,o unlock com que uma dessas threads
    saia do lock com o mutex do parâmetro. Se nenhuma threads estiver
    aguardando o mutex, o mutex será desbloqueado sem proprietário
    atual;

-   `pthread_mutex_destroy(pthread_mutex_t *mutex);` -- Deleta o mutex
    especificado no parâmetro.

Por último, envolvendo variáveis condicionais:[@CCond]

-   `pthread_cond_init(pthread_cont_t *cv, const pthread_condattr_t *cattr);`
    -- Inicia a variável condicional passada no primeiro parâmetro. O
    segundo parâmetro refere-se aos atributos dessa variável
    condicional, se você não sabe o que está fazendo, o ideal é por
    *NULL*;

-   `pthread_cond_wait(pthread_cont_t *cv,pthread_mutex_t *mutex);` -- Esta
    função bloqueia até que a condição seja sinalizada (*signal()*. Ele
    atomicamente libera a trava mutex associada antes de bloquear, e
    atomicamente a adquire novamente antes de retornar;

-   `pthread_cond_signal(pthread_cont_t *cv);` -- Desbloqueia uma thread
    específica;

-   `pthread_cond_broadcast(pthread_cont_t *cv);` -- Desbloqueia todas as
    threads que estiverem bloqueadas;

-   `pthread_cond_destroy(pthread_cont_t *cv);` -- Destrói a variável
    condicional passada no parâmetro.

Uma coisa que você pode estar se perguntando é: Por que, destas funções,
o wait necessita, além da variável condicional, um mutex? A resposta
para isto é que o mutex é usado para *proteger* a variável condicional
quando ocorre o *wait()*. O *wait()* irá \"atomicamente\" desbloquear o
mutex, permitindo que outros acessem a variável de condição para
*signal()*. Então, quando ocorre um *signal()* ou *broadcast()*
envolvendo a variável condicional, uma ou mais threads bloqueadas serão
acordados e o mutex será magicamente bloqueado novamente para essa
thread.

Para mais detalhes e conhecer mais funções disponíveis, você pode
consultar a
[documentação](http://pubs.opengroup.org/onlinepubs/7908799/xsh/pthread.h.html).

Trabalhando com múltiplas threads (C)
-------------------------------------

Para entendermos sobre múltiplas threads em C, observemos a versão em C
do código contido no *exercício 2*.

``` {style="CStyle"}
#include<stdio.h>
#include<pthread.h>
#include <unistd.h>

void* run(void* args){
    int id = (int) args;
    for (int i = 0; i < 3; i++){
        printf("Thread %d executando\n", id);
        sleep(1);
    }  
   
    printf("Thread %d terminando\n", id);
    pthread_exit(NULL);
}

int main(int argc, char *argv[]){
     int i;
    pthread_t pthreads[3];

    for (i = 0; i < 3; i++) {
        pthread_create(&pthreads[i], NULL, &run, (void*) i + 1);
    }

    for (i = 0; i < 3; i++) {
        pthread_join(pthreads[i], NULL);
    }

    return 0;
}
```

*Exemplo 4*

O output gerado foi:

    Thread 1 executando
    Thread 2 executando
    Thread 3 executando
    Thread 1 executando
    Thread 3 executando
    Thread 2 executando
    Thread 3 executando
    Thread 1 executando
    Thread 2 executando

Exclusão mútua
==============

Como já vimos antes, exclusão mútua nada mais é do que garantir que
apenas uma thread entre na região crítica, evitando assim, condições de
corrida. Nas seções abaixo exploraremos soluções em C e Java que
utilizam este recurso para proteger o código.

Produtor/Consumidor
-------------------

Nas situações que vimos até agora, nossas threads não precisaram se
preocupar com que as outras threads estavam fazendo, porém, na grande
maioria dos programas que envolvem concorrência, elas são obrigadas a se
preocupar com isto, um exemplo é a aplicação do produtor/consumidor.

Nesta aplicação,o produtor produz um determinado fluxo de dados que é
consumido pelo consumidor assim que foi produzido. Nisto, podem ocorrer
diversas situações problemáticas, como por exemplo, o consumidor
consumir mais rápido do que o produtor pode produzir ou o produtor
produzir mais rápido do que o consumidor consome.

Dado isto, iremos explorar soluções para que o programa execute um fluxo
correto de produzir/consumir.[@Jacques]

### Produtor/Consumidor (Java)

Neste exemplo, o produtor irá produzir valores de 0 a 9 e colocá-los num
objeto *Caixinha* num intervalo que pode variar entre 0 e 100ms.

    class Produtor implements Runnable {
        private int id;
        private Caixinha caixinha;

        public Produtor(int id, Caixinha caixinha) {
            this.id = id;
            this.caixinha = caixinha;
        }

        @Override
        public void run() {
            try {
                for (int i = 0; i < 10; i++) {
                    this.caixinha.coloca(i);
                    Thread.sleep((long) (Math.random()*100));
                }
            } catch (InterruptedException e) {
                System.out.println(id + "interrompida");
            }
            System.out.println("Produtor " + id + " terminando");
        }
    }

*Exemplo 5*

O consumidor retira (consome) os valores da *Caixinha* assim que se
tornam disponíveis. Se ele tentar consumir antes de ocorrer a produção,
irá retornar -1.

    private int id;
        private Caixinha caixinha;

        public Consumidor(int id, Caixinha caixinha) {
            this.id = id;
            this.caixinha = caixinha;
        }

        @Override
        public void run() {
            int valor = 0;
            for (int i = 0; i < 10; i++) {
                valor = caixinha.retira();
            }
            System.out.println("Consumidor " + this.id + " terminando");
        }
    }

*Exemplo 6*

Vemos que o produtor e o consumidor compartilham dados através do objeto
*Caixinha*, que o torna uma região crítica, suscetível a condições de
corrida.

Com isso,como já citado anteriormente, alguns problemas podem ocorrer:

-   O consumidor consumir mais rápido do que o produtor produz

            ...
            Produtor #1 colocou: 0
            Consumidor #1 retirou: 0
            Consumidor #1 retirou: -1
            Consumidor #1 retirou: -1
            ...
            

-   O produtor produzir mais rápido do que o consumidor consome, em que
    o consumidor deixa de consumir um número

            ...
            Consumidor #1 retirou: 1
            Produtor #1 colocou: 2
            Produtor #1 colocou: 3
            Consumidor #1 retirou: 3
            ...
            

Uma solução para este problema seria garantir que a *Caixinha* só deixe
o produtor colocar algo quando o dado anterior tiver sido consumido.

Para isso, as threads não devem acessar a *Caixinha* simultaneamente.
Isto pode ser impedido se uma thread travar o objeto. Quando o objeto
está travado, um outro thread que chamar um método sincronizado
(*synchronized*) no mesmo objeto vai bloquear até o objeto ser
destravado.

Além disso, as threads devem coordenar seu trabalho, em que o produtor
deve ter uma forma de dizer ao consumidor que um novo número está
disponível para consumo e o consumidor deve ter uma forma de dizer ao
produtor que o número foi consumido, liberando a produção de outro
número. Para isto existem alguns métodos para permitir que threads
esperem por uma condição e notificar outras threads quando uma condição
ocorre (*wait()*, *notify()*, *notifyAll()*).

Existem duas formas de resolver estes problemas:

-   Colocar as travas no objeto *Caixinha*;

-   Colocar as travas no *Produtor* e no *Consumidor*.

Iremos ver a primeira opção.

**Travas no objeto *Caixinha***

As regiões críticas do objeto *Caixinha* são os métodos *coloca()* e
*retira()*, que ambos o produtor e consumidor compartilham seus dados.

    class Caixinha {
        private int valor = -1;

        public synchronized void coloca(int id, int valor){
            ...
        }

        public synchronized int retira(int id){
           ...
        }

        public boolean vazia(){
            return this.valor == -1;
        }
    }

Em que o método *coloca()* recebe como parâmetros o id do produtor e o
valor a ser colocado e o método *retira()* recebe o id do consumidor.

Obs.Talvez o primeiro pensamento pra solucionar o problema seja fazer
utilização do método *vazia()*, verificando se o objeto está vazio ou
não e assim controlar o consumo/produção.

    class Caixinha {
        private int valor;

        public Caixinha() {
            this.valor = -1;
        }

        public synchronized void coloca(int id,int valor){
            if(this.vazia()){
                this.valor = valor;
            }
        }

        public synchronized int retira(int id){
            int valorRetirado = -1;
            if(!this.vazia()){
                valorRetirado = this.valor;
                this.valor = -1;
            }
            return valorRetirado;
        }

        public boolean vazia(){
            return this.valor == -1;
        }
    }

*Exemplo 7*

Isso não funciona! Se por exemplo, não houver nada na *Caixinha*
(*vazia()* == true), o método *retira()* não faz nada, o correto seria
esperar até se ter algo. Outro exemplo de bug seria: se houver algo na
*Caixinha* (*vazia()* == false), o método *coloca()* não faz nada e
perde o valor, o correto seria esperar até poder guardá-lo. Ou seja,
continuamos com os mesmos problemas de antes!

Precisamos que as threads sinalizem umas as outras quando elas podem ou
não continuar e isto é feito utilizando os métodos *wait()* e
*notifyAll()*. Uma possível modificação no código do objeto que
resolveria o problema seria:

    class Caixinha {
        private int valor = -1;

        public synchronized void coloca(int id, int valor){
            while (!this.vazia()){
                try {
                    wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            this.valor = valor;
            System.out.println("Produtor #" + id +  " colocou: " + valor);
            notifyAll();
        }

        public synchronized int retira(int id){
            while (this.vazia()){
                try {
                    wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

            int valorRetirado = this.valor;
            this.valor = -1;
            System.out.println("Consumidor #" + id +  " retirou: " + valorRetirado);
            notifyAll();

            return valorRetirado;
        }

        public boolean vazia(){
            return this.valor == -1;
        }
    }

*Exemplo 8*

Neste código, o método *wait()* libera o lock e espera notificação para
continuar, isto é necessário para que o outro thread possa adquirir o
lock, fazer seu trabalho e acordar o outro com *notifyAll()*. Ao
continuar, o lock é obtido novamente.

O método *notifyAll()* \"acorda\" todos os threads que estão em *wait()*
nesse objeto. Os threads que acordam competem pelo lock, quando um
thread pega o lock, os outros voltam a dormir.

Eis o output do código acima:

    Produtor #1 colocou: 0
    Consumidor #1 retirou: 0
    Produtor #1 colocou: 1
    Consumidor #1 retirou: 1
    Produtor #1 colocou: 2
    Consumidor #1 retirou: 2
    Produtor #1 colocou: 3
    Consumidor #1 retirou: 3
    Produtor #1 colocou: 4
    Consumidor #1 retirou: 4
    Produtor #1 colocou: 5
    Consumidor #1 retirou: 5
    Produtor #1 colocou: 6
    Consumidor #1 retirou: 6
    Produtor #1 colocou: 7
    Consumidor #1 retirou: 7
    Produtor #1 colocou: 8
    Consumidor #1 retirou: 8
    Produtor #1 colocou: 9
    Consumidor #1 retirou: 9
    Consumidor 1 terminando
    Produtor 1 terminando

Com isso, vemos que este código funciona e é uma das opções de solução
para o nosso problema. A segunda opção de solução para o problema pode
ser vista no
[github](https://github.com/thiagomanel/fpc/blob/master/threads/java/prod_cons/src/br/edu/ufcg/lsd/pc/Main.java)
do professor, em que ele utiliza um equivalente ao objeto *Caixinha*
(Data) como trava do *synchronized* nas classes de produtor/consumidor.

### Produtor/Consumidor (C)

Agora veremos como é implementado o produtor/consumidor utilizando a
linguagem C. Para isto utilizaremos mutex e variáveis condicionais, que
são mecanismos de sincronização de threads da linguagem C.

Você pode implementar esta solução utilizando apenas uma variável
condicional, Porém, na solução que será mostrada, teremos um buffer de
tamanho 1, usaremos duas variáveis condicionais (uma para quando o
buffer estiver vazio e outra para quando estiver cheio) e um
mutex.[@Cprodcons]

Seguindo a mesma linha do nosso exemplo em Java, nosso produtor irá
produzir valores de 0 a 9.

``` {style="CStyle"}
void* produtor(void* args){
    for (int i = 0; i < 10; ++i) {
        pthread_mutex_lock(&mutex);
        while(contador == 1){
            pthread_cond_wait(&vazio, &mutex);
        }
        printf("Produtor colocou %d\n", i);
        coloca(i);
        pthread_cond_signal(&cheio);
        pthread_mutex_unlock(&mutex);
        printf("Produtor terminando");
    }
}
```

No produtor temos, a partir a linha 3:

-   O mutex faz um *lock*, travando o trecho de código;

-   Em seguida, é feita uma checagem para saber se o buffer está cheio
    (contador == 1);

-   Se estiver cheio, é realizado um *wait()* até ele ficar vazio
    (contador == 0);

-   Quando estiver vazio, coloca o item no buffer (produz);

-   Sinaliza que o buffer está cheio;

-   Destrava o mutex.

``` {style="CStyle"}
void* consumidor(void* args){
    for (int i = 0; i < 10; ++i) {
        pthread_mutex_lock(&mutex);
        while(contador == 0){
            pthread_cond_wait(&cheio, &mutex);
        }
        int valorPego = retira();
        printf("Consumidor retirou %d\n", valorPego);
        pthread_cond_signal(&vazio);
        pthread_mutex_unlock(&mutex);
        printf("Consumidor terminando");
    }
}
```

No consumidor temos, a partir da linha 3:

-   O mutex faz um *lock*, travando o trecho de código;

-   Em seguida, é feita uma checagem para saber se o buffer está vazio
    (contador == 0);

-   Se estiver vazio, é realizado um *wait()* até ele ficar cheio
    (contador == 1);

-   Quando estiver cheio, retira o item do buffer (consome)

-   Sinaliza que o buffer está vazio;

-   Destrava o mutex.

E aqui são as funções de colocar/retirar

``` {style="CStyle"}
int retira(){
    contador = 0;
    return buffer;
}

void coloca (int valor){
    contador = 1;
    buffer = valor;
}
```

E o main

``` {style="CStyle"}
int main() {
    pthread_t t1;
    pthread_t t2;

    pthread_mutex_init(&mutex, NULL);
    pthread_cond_init(&cheio, NULL);
    pthread_cond_init(&vazio, NULL);

    pthread_create(&t1, NULL, produtor, NULL);
    pthread_create(&t2, NULL, consumidor, NULL);

    pthread_join(t1, NULL);
    pthread_join(t2, NULL);

    return 0;
}
```

O output deste programa foi:

    Produtor colocou 0
    Consumidor retirou 0
    Produtor colocou 1
    Consumidor retirou 1
    Produtor colocou 2
    Consumidor retirou 2
    Produtor colocou 3
    Consumidor retirou 3
    Produtor colocou 4
    Consumidor retirou 4
    Produtor colocou 5
    Consumidor retirou 5
    Produtor colocou 6
    Consumidor retirou 6
    Produtor colocou 7
    Consumidor retirou 7
    Produtor colocou 8
    Consumidor retirou 8
    Produtor colocou 9
    Produtor terminando
    Consumidor retirou 9
    Consumidor terminando

