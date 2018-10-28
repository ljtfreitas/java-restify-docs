# Programação Reativa - RxJava 2.x

[RxJava](https://github.com/ReactiveX/RxJava) é a implementação Java do [Reactive Extensions](http://reactivex.io/), uma poderosa API para programação reativa.

O `java-restify` fornece suporte para as versões [1.x](rxjava-1.md) e 2.x do RxJava, e os principais objetos podem ser utilizados como retorno de método.

## Instalação

O suporte para o RxJava 2.x está na dependência `java-restify-rxjava-2`. Uma vez presente no `classpath`, os `handlers` serão automaticamente registrados.

### Maven

```xml
<dependency>
  <groupId>com.github.ljtfreitas</groupId>
  <artifactId>java-restify-rxjava-2</artifactId>
  <version>{version}</version>
</dependency>
```

### Gradle

```groovy
dependencies {
  compile("com.github.ljtfreitas:java-restify-rxjava-2:{version}")
}
```

## Utilização

### Tipos suportados

* `Observable`
    
[Observable](http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/Observable.html) é um objeto reativo que representa a emissão de uma **sequência de valores**. 

Ao utilizar o `Observable` como retorno de método, o `java-restify` irá assumir que a resposta da requisição representa uma **coleção**. 
    
Por exemplo, digamos que o `endpoint` a ser consumido retorne um JSON; se a resposta for um *array* (uma coleção), você pode utilizar o `Observable`; se a resposta for um único objeto, utilize um `Single` ou um `Maybe` (ver abaixo).

```java
import io.reactivex;

public interface MyApi {

    @Path("/customers") @Get
    Observable<Customer> getAllCustomers();
}
```

* `Flowable`
    
[Flowable](http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/Flowable.html) é um novo objeto reativo introduzido na versão 2.x do RxJava, que implementa a interface [Publisher](http://www.reactive-streams.org/reactive-streams-1.0.2-javadoc/org/reactivestreams/Publisher.html), da especificação [Reactive Streams](http://www.reactive-streams.org/). Assim como o `Observable`, o `Flowable` representa a emissão de uma **sequência de valores** com suporte a `back-pressure`.

Ao utilizar o `Flowable` como retorno de método, o `java-restify` irá assumir que a resposta da requisição representa uma **coleção**. 

```java
import io.reactivex.Flowable;

public interface MyApi {

    @Path("/customers") @Get
    Flowable<Customer> getAllCustomers();
}
```

* `Single`

[Single](http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/Single.html) é um objeto reativo que representa a emissão de **um único valor**.

O `Single` é um tipo de retorno adequado caso a resposta da requisição represente um único objeto (assim como o `Observable` ou `Flowable` são mais adequados para uma sequência/coleção de objetos); outra possibilidade para esse cenário é o `Maybe` (detalhado mais abaixo).

```java
import io.reactivex.Single;

public interface MyApi {

    @Path("/customers/{id}") @Get
    Single<Customer> getCustomerById(@PathParameter String id);
}
```

* `Maybe`

[Maybe](http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/Maybe.html) é um objeto reativo que representa a emissão de **um único valor** ou **nenhum valor**.

O `Maybe` é um tipo de retorno adequado caso a resposta da requisição represente um único objeto; uma diferença em relação ao `Single` é o fato do `Maybe` permitir valores nulos (por exemplo, uma resposta vazia).

```java
import io.reactivex.Maybe;

public interface MyApi {

    @Path("/customers/{id}") @Get
    Maybe<Customer> getCustomerById(@PathParameter String id);
}
```

* `Completable`

[Completable](http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/Completable.html) é um objeto reativo que **não emite valores**, podendo apenas ser completado sem erros ou emitir um erro. 

O uso desse objeto como retorno de método é adequado quando não é necessário obter o corpo da resposta, mas o seu código deve reagir quando a requisição for concluída sem erros ou em caso de problemas.

```java        
import rx.Completable;

public interface MyApi {

    @Path("/customers") @Post
    Completable createCustomer(@BodyParameter Customer customer);
}
```

### Configuração

Em todos os casos acima, o `java-restify` irá executar a requisição em uma `thread` separada usando o [suporte do RxJava para processamento assíncrono](http://reactivex.io/documentation/scheduler.html), através do objeto [Scheduler](http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/Scheduler.html).

Por padrão, o `Scheduler` será criado a partir do método [Schedulers.io](http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/schedulers/Schedulers.html#io--), que é o mais adequado para requisições HTTP.

Caso essa configuração não atenda às necessidades da sua aplicação, desligue a descoberta automática de `handlers` e registre-os manualmente:

```java
import io.reactivex.Scheduler;
import io.reactivex.schedulers.Schedulers;
import com.github.ljtfreitas.restify.http.client.call.handler.rxjava2.RxJava2CompletableEndpointCallHandlerFactory;
import com.github.ljtfreitas.restify.http.client.call.handler.rxjava2.RxJava2FlowableEndpointCallHandlerAdapter;
import com.github.ljtfreitas.restify.http.client.call.handler.rxjava2.RxJava2MaybeEndpointCallHandlerAdapter;
import com.github.ljtfreitas.restify.http.client.call.handler.rxjava2.RxJava2ObservableEndpointCallHandlerAdapter;
import com.github.ljtfreitas.restify.http.client.call.handler.rxjava2.RxJava2SingleEndpointCallHandlerAdapter;

Scheduler myScheduler = Schedulers.from(Executors.newFixedThreadPool(10));

MyApi myApi = new RestifyProxyBuilder()
  .handlers()
    .discovery()
        .disabled()
    .add(new RxJava2CompletableEndpointCallHandlerFactory(myScheduler))
    .add(new RxJava2FlowableEndpointCallHandlerAdapter<>(myScheduler))
    .add(new RxJava2MaybeEndpointCallHandlerAdapter<>(myScheduler))
    .add(new RxJava2ObservableEndpointCallHandlerAdapter<>(myScheduler))
    .add(new RxJava2SingleEndpointCallHandlerAdapter<>(myScheduler))
    .and()
  .target(MyApi.class)
    .build();
```