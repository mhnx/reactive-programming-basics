# Curso "Construa microsserviços reativos usando Spring WebFlux e Spring Boot".

## Seção 1: Começando o curso
O que o curso aborda?
- Introdução a programação reativa
- Vantagens da programação reativa em relação aos modelos de programação tradicionais
- Escrita de código de programação reativa usando o Project Reactor, que é uma biblioteca moderna que implementa os princípios do fluxo reativo (reactive stream).
- Construção de três serviços REST reativos usando o Spring WebFlux.
- Testes unitários e de integração com JUnit5.

Outro curso recomendado é o "Reactive Programming in Modern Java using Project Reactor".

Pré-requisitos:
- Conhecimento em Spring Boot.
- Experiência construindo APIs RESTful.
- Conhecimento em streams e lambdas do Java 8.

## Seção 2: Código-fonte
https://github.com/dilipsundarraj1/reactive-spring-webflux/tree/final

## Seção 3: Por que programação reativa?
- No início da programação os projetos eram criados como monolitos, que são um tipo de aplicação na qual diferentes componentes estão envelopados em um só e são implantadas em um servidor.
- Hoje existe o conceito de microsserviços, onde estes são implantados em ambientes em nuvem (servidores online oferecidos por terceiros como Amazon, Microsoft e Google). Além disso há também o forte conceito de sistema distribuído.
- O atual padrão de mercado exige que as aplicações de software tenham um tempo de resposta na casa dos milissegundos, alta disponibilidade (sem downtime) e escalonamento automático baseado em carga.

Entendendo o funcionamento do Spring MVC
- Spring MVC é um padrão muito utilizado na criação de APIs RESTful.
- Uma API com Spring MVC possui um servidor (Apache Tomcat) embutido que gerencia as chamadas vindas do cliente através da rede. Quando o cliente realiza uma chamada (request), o servidor designa uma thread (um segmento) na thread pool para atender a request e providenciar uma resposta (response) ao cliente.
- O Spring utiliza o padrão de concorrência, ou seja, o modelo de thread por request.
- Esse estilo de construção é chamado de API bloqueante. Porque a medida que o número de requests aumenta mais threads são criadas para atender as requests sobrecarregando o sistema pois em grande escala o tempo de resposta aumenta consideravelmente.
- O Dispatcher Servlet, que é o responsável por direcionar as requests pros Controllers corretos é bloqueante por natureza.
- A medida que o microsserviço precisa se comunicar com outros serviços externos para processar a resposta ao cliente o seu tempo de resposta aumenta a cada serviço externo.

Limitações do Spring MVC
- A tamanho da thread pool do Tomcat embutido no Spring MVC é de 200 threads. Ou seja, a concorrência é de no náximo 200 threads.
- É possível aumentar esse tamanho até um certo limite. Pois uma thread é um recurso computacional que ocupa muito espaço. Uma thread pode ocupar facilmente 1 MB no espaço da heap (memória RAM). Menos espaço na heap pode impactar na performance geral da aplicação. Por isso, ao construir aplicações em que se espera uma grande demanda de requests concorrentes, utilizar o Spring MVC pode não ser uma boa ideia.
- Outra possibilidade que existe para tentar contornar o problema do aumento da quantidade de tempo de resposta de uma request é paralelizar as chamadas da API para os serviços externos de que depende. Em Java, e em outras linguagens, existe um recurso chamado Callback. Há também o Future.

Porque callbacks não funcionam e quais são suas desvantagens?
- Callbacks são basicamente métodos assíncronos que aceitam um callback como parâmetro e o invoca quando a chamada bloqueante é concluída.
- Codar callbacks é complexo e difícil de manutenir. Quando o projeto amadurece e fica difícil de manutenir caracteriza-se o que chamamos de "callback hell".

Agora vamos conhecer a API de Future:
- Lançada no Java 5.
- Esta API permite aos programadores trabalhar melhor com threads escrevendo código assíncrono.
- A principal desvantagem é que não há uma maneira fácil de combinar o resultado de múltiplos futures.
- Outra desvantagem é que geralmente usa-se o método .get() de um future para obter seu resultado e esse método é bloqueante.
- No Java 8 foi lançado o CompletableFuture, que nos permite escrever código assíncrono no estilo funcional. Com ele é mais fácil compor/combinar múltimos futures.
- A única desvantagem do CompletableFuture é que ele não possui uma boa forma de lidar com computação assíncrona quando retorna múltiplos valores. Por exemplo, imagine que você possui um CompletableFuture do tipo List: CompletableFuture<List>. A chamada dele vai esperar que toda a coleção seja construída e completamente disponível para considerar a chamada assíncrona como completa. O que é um problema recorrente em sistemas de companhias. Além dessa desvantagem mencionada, CompletableFuture também não lida bem com valores infinitos, que veremos o que são mais adiante no curso.

Resumindo o que foi dito:
- Concorrência possui limitações no Spring MVC.
- Código bloqueante leva ao uso ineficiente de threads.
- A API de Servlet é bloqueante a nível de servidor por natureza.

Considerando essas limitações acima temos acesso a uma solução melhor: programação reativa.

## Seção 4: O que é programação reativa?
- É um novo paradigma de programação.
- Assíncrono e não-bloqueante.
- Os dados fluem como um fluxo orientado a evento/mensagem.

Como a programação reativa funciona?
- Por debaixo dos panos, quando uma aplicação faz uma request para um banco de dados, é criada uma thread não-bloqueante que solicitará os dados necessários. A diferença é que essa thread não vai esperar a resposta do banco de dados. Essa thread retorna imediatamente avisando a aplicação que a solitação foi feita com sucesso. Assim a thread fica livre para realizar qualquer outro trabalho importante.
- Um segundo request vai ser enviado para o banco de dados avisando que a aplicação está pronta para consumir os dados. Quando o resultado da consulta estiver pronto, este será enviado na forma de um fluxo de eventos, chamado de fluxo reativo. Os dados serão enviados através de uma função onNext() a aplicação:
<- onNext(1)
<- onNext(2)
...
<- onNext(n)
Ao final uma função onComplete() será enviada a aplicação para avisar que todos os dados necessários foram enviados. Todo esse processo é feito pela biblioteca reativa.
Observando-se esse modelo podemos dizer que os dados são empurrados da fonte de dados (data source) para quem a invocou/chamou. Sendo assim, podemos chamar esse modelo de fluxo de dados da programação reativa como modelo baseado em push.
- Não confundir esse fluxo de dados reativo com o fluxo de dados do Java 8.
- É importante lembrar que a programação reativa está diretamente ligada com a programação funcional. Quem tem experiência com streams (fluxo de dados) no Java 8, lambdas, referências de métodos e interfaces funcionais vai encontrar similaridades com a programação reativa uma vez que esta foi construída baseada nesses conceitos. Pode-se dizer ainda que a programação reativa é uma extensão da programação funcional.
- Uma das principais funcionalidades da programação reativa é o suporte a back pressure ao fluxo de dados reativo. Essa funcionalidade permite controlar o fluxo de dados. Vamos entender como.

O que é back pressure?
- Vamos pensar no modelo apresentado anteriormente. Quando a aplicação envia a request ao data source, a thread que enviou a request retorna imediatamente. Depois disso uma nova thread é enviada solicitando ao data source 2 eventos. Uma vez que esses eventos são processados pela aplicação, nós temos duas opções: pedir mais eventos para processar ou cancelar a request.
- Através desse conceito nós podemos contrar o fluxo de envio de dados do data source à aplicação de modo que a aplicação não sofra qualquer tipo de sobrecarga no processamento dos dados/eventos.
- Com o conceito de back pressure em cena, esse modelo de fluxo de dados recebe um nome atualizado: modelo de fluxo de dados baseado em push e pull (empurrar e puxar, respectivamente).

Quando usar programação reativa?
- Quando há necessidade de construir e manuternir aplicações com muita carga e com alta disponibilidade de recursos.
- Por exemplo: uma aplicação que precisa realizar 400 transações por segundo.

Como a aplicação trata as requests vindas dos clients?
- A aplicação precisa de um servidor não-bloqueante, caso contrário a solução reativa não estará completa. Para isso usamos o Netty, um servidor não-bloqueante que usa o modelo de loop de eventos. E para trablhar com a programação reativa faremos uso da biblioteca Project Reactor para escrever código reativo de modo eficiente.
- Para desenvolver o projeto de modo reativo em todas as pontas (do client à aplicação/servidor e deste às outras APIs ou datasources vamos usar o framework Spring WebFlux.

Reactive Streams (Fluxos Reativos)
A especificação dos reactive streams foi desenvolvida pela parceria de big techs como Lightbend, Netflix e Pivotal. A especificação é bem simples e possui 4 interfaces:
- Publisher
- Subscriber
- Subscription
- Processor

Vamos conhecer os métodos que cada interface possui:

**Publisher**

```java
public interface Publisher<T> {
    public void subscribe(Subscriber<? super T> s);
}

```
O Publisher representa o datasource, que pode ser o database, um RemoteService ou qualquer coisa que detenha os dados necessários para a aplicação.

---

**Subscriber**
```java
public interface Subscriber<T> {
    public void onSubscribe(Subscription s);
    public void onNext(T t);
    public void onError(Throwable t);
    public void onComplete();
}

```
O datasource usa os métodos ```onNext()``` para enviar os dados e o ```onComplete()``` para avisar de que a transmissão de dados foi concluída com sucesso.

---

**Subscription**
```java
public interface Subscription<T> {
    public void request(long n);
    public void cancel();
}

```
A aplicação faz uma chamada usando o método ```request()``` para pedir os dados ao datasource.
O método ```cancel()``` é utilizado para interromper a transmissão de dados quando o volume já transmitido é suficiente.

Ou seja, o Subscription é quem se conecta ao datasource, que é Publisher e Subscriber ao mesmo tempo.

---
Reactive Streams - Como funciona tudo junto?

Vamos conhecer o fluxo dos reactive streams em um **cenário de sucesso**, ou seja, onde não há erros ou exceções.

Imagine que temos um Subscriber e um Publisher, sendo o Subscriber o servidor e o Publisher o datasource (um banco de dados ou um RemoteService ou qualquer outro sistema externo).

Todo o fluxo começa com o Subscriber iniciando uma request invocando o método ```subscribe(this)``` do Publisher. Esse é o primeiro passo de toda a interação.
O segundo passo é o Publisher passar o objeto do tipo Subscription invocando o método ```onSubscribe()``` do Subscriber.
Uma vez que o Subscriber tem o objeto do tipo Subscription a sua disposição, o próximo passo é o Subscriber invocar o método ```request(n)``` desse objeto solicitando ao Publisher os dados necessários.
Quando o Publisher receber essa solicitação, os dados serão enviados por argumento invocando o método ```onNext()``` que é parte da interface Subscriber na **forma de eventos**.

Ao final do envio de todos os dados solicitados pelo Subsriber, o Publisher invocará o método ```onComplete()``` do Subscriber para avisar que o envio dos eventos foi concluído com sucesso.

Agora vejamos um **cenário de erro/exceção**:
Imagine que ao invocar o método ```request(n)``` do objeto Subscription ocorra uma exceção em tempo de execução (Runtime Exception). Isso pode ocorrer porque o banco de dados está temporariamente fora do ar. Nesse caso a exceção será enviada como um evento usando o método ```onError()``` do Subscriber.
Essa forma de notificar uma exceção é diferente da forma tradicional de lançar uma exceção. Sendo assim é importante observarmos alguns pontos:
- Na programação reativa as exceções são tratadas da forma forma que os dados: como um evento.
- Quando uma exceção é enviada como um evento, a transmissão de dados é encerrada. O stream de dados morre.

---

**Processor**
```java
public interface Processor<T, R> extends Subscriber<T>, Publisher<R> {
}

```
A ideia do Processor é se comportar como um Subscriber e um Publisher. Ele geralmente não é utilizado no dia a dia do desenvolvedor, mas é importante conhecer essa opção.

---

Flow API (Java 9)
- Detém o contrato para streams reativos mas não possui implementação disponível para o JRE.

## Seção 5: Introdução ao Spring WebFlux
O que é uma API RESTful reativa ou não-bloqueante?
- Uma API RESTful Reativa ou Não-bloqueante tem o papel de prover um meio de comunicação de ponta a ponta não-bloqueante entre o cliente e o servidor.
- Reativo ou não-bloqueante significa que a thread não é bloqueada aguardando outra thread (externa ou não) ter seu processamento concluído para assim poder ter o seu processamento também concluído.
- É quando uma thread não é bloqueada ao lidar com ```HttpRequest``` e ```HttpResponse```.
- Como alcançar isso? Através do Spring WebFlux: um módulo do Spring Framework capaz de implementar o comportamento reativo ou não-bloqueante.
- O servidor embutido Netty é ideal pois permite focar na construção de endpoints e na lógica de negócio para a API.
