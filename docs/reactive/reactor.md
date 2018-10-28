# Programação Reativa - Project Reactor

[Project Reactor](https://projectreactor.io/) é um framework reativo que implementa a especificação [Reactive Streams](http://www.reactive-streams.org/). O principal foco do `Project Reactor` é o suporte a aplicações assíncronas de baixa latência e alta performance, com uso de *back-pressure*.

O `java-restify` fornece suporte para os dois principais objetos do Reactor, `Flux` e `Mono`, que podem ser utilizados como retorno de método.

## Instalação

O suporte para o Reactor está na dependência `java-restify-reactor`. Uma vez presente no `classpath`, os `handlers` serão automaticamente registrados.

### Maven

```xml
<dependency>
  <groupId>com.github.ljtfreitas</groupId>
  <artifactId>java-restify-reactor</artifactId>
  <version>{version}</version>
</dependency>
```

### Gradle

```groovy
dependencies {
  compile("com.github.ljtfreitas:java-restify-reactor:{version}")
}
```

## Utilização

### Tipos suportados

* `Flux`
    
[Flux](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html) é um objeto reativo que representa a emissão de uma **sequência de valores** (0-N).

Ao utilizar o `Flux` como retorno de método, o `java-restify` irá assumir que a resposta da requisição representa uma **coleção**. 
    
Por exemplo, digamos que o `endpoint` a ser consumido retorne um JSON; se a resposta for um *array* (uma coleção), você pode utilizar o `Flux`; se a resposta for um único objeto, utilize um `Mono` (ver abaixo).

```java
import reactor.core.publisher.Flux;

public interface MyApi {

    @Path("/customers") @Get
    Flux<Customer> getAllCustomers();
}
```

* `Mono`

[Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html) é um objeto reativo que representa a emissão de **um único valor** (0-1).

O `Mono` é o retorno adequado caso a resposta da requisição represente um único objeto (assim como o `Flux` é o mais adequado para uma sequência/coleção de objetos). Esse objeto também suporta retornos nulos (como uma resposta vazia).

```java
import reactor.core.publisher.Mono;

public interface MyApi {

    @Path("/customers/{id}") @Get
    Mono<Customer> getCustomerById(@PathParameter String id);
}
```

### Configuração

Em todos os casos acima, o `java-restify` irá executar a requisição em uma `thread` separada usando o [suporte do Reactor para processamento assíncrono](https://projectreactor.io/docs/core/release/reference/#schedulers), através do objeto [Scheduler](https://projectreactor.io/docs/core/release/api/reactor/core/scheduler/Scheduler.html). 

Por padrão, o `Scheduler` será criado a partir do método [Schedulers.elastic](https://projectreactor.io/docs/core/release/api/reactor/core/scheduler/Schedulers.html#elastic--), que é o mais adequado para requisições HTTP.

Caso essa configuração não atenda às necessidades da sua aplicação, desligue a descoberta automática de `handlers` e registre-os manualmente:

```java
import reactor.core.scheduler.Scheduler;
import reactor.core.scheduler.Schedulers;
import com.github.ljtfreitas.restify.http.client.call.handler.reactor.FluxEndpointCallHandlerAdapter;
import com.github.ljtfreitas.restify.http.client.call.handler.reactor.MonoEndpointCallHandlerAdapter;

Scheduler myScheduler = Schedulers.fromExecutor(Executors.newFixedThreadPool(10));

MyApi myApi = new RestifyProxyBuilder()
  .handlers()
    .discovery()
        .disabled()
    .add(new FluxEndpointCallHandlerAdapter<>(myScheduler))
    .add(new MonoEndpointCallHandlerAdapter<>(myScheduler))
    .and()
  .target(MyApi.class)
    .build();
```
