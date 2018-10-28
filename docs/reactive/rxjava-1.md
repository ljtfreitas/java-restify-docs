# Programação Reativa - RxJava 1.x

[RxJava](https://github.com/ReactiveX/RxJava) é a implementação Java do [Reactive Extensions](http://reactivex.io/), uma poderosa API para programação reativa.

O `java-restify` fornece suporte para as versões 1.x e [2.x](rxjava-2.md) do RxJava, e os principais objetos podem ser utilizados como retorno de método. 

## Instalação

O suporte para o RxJava 1.x está na dependência `java-restify-rxjava`. Uma vez presente no `classpath`, os handlers serão automaticamente registrados.

### Maven

```xml
<dependency>
  <groupId>com.github.ljtfreitas</groupId>
  <artifactId>java-restify-rxjava</artifactId>
  <version>{version}</version>
</dependency>
```

### Gradle

```groovy
dependencies {
  compile("com.github.ljtfreitas:java-restify-rxjava:{version}")
}
```

## Utilização

### Tipos suportados

* `Observable`
    
[Observable](http://reactivex.io/RxJava/1.x/javadoc/rx/Observable.html) é um objeto reativo que representa a emissão de uma **sequência de valores**. 

Ao utilizar o `Observable` como retorno de método, o `java-restify` irá assumir que a resposta da requisição representa uma **coleção**. 
    
Por exemplo, digamos que o `endpoint` a ser consumido retorne um JSON; se a resposta for um *array* (uma coleção), você pode utilizar o `Observable`; se a resposta for um único objeto, utilize um `Single` (ver abaixo).

```java
import rx.Observable;

public interface MyApi {

    @Path("/customers") @Get
    Observable<Customer> getAllCustomers();
}
```

* `Single`

[Single](http://reactivex.io/RxJava/1.x/javadoc/rx/Single.html) é um objeto reativo que representa a emissão de **um único valor**.

O `Single` é o retorno adequado caso a resposta da requisição represente um único objeto (assim como o `Observable` é o mais adequado para uma sequência/coleção de objetos).

```java
import rx.Single;

public interface MyApi {

    @Path("/customers/{id}") @Get
    Single<Customer> getCustomerById(@PathParameter String id);
}
```

* `Completable`

[Completable](http://reactivex.io/RxJava/1.x/javadoc/rx/Completable.html) é um objeto reativo que **não emite valores**, podendo apenas ser completado sem erros ou emitir um erro. 

O uso desse objeto como retorno de método é adequado quando não é necessário obter o corpo da resposta, mas o seu código deve reagir quando a requisição for concluída sem erros ou em caso de problemas.

```java        
import rx.Completable;

public interface MyApi {

    @Path("/customers") @Post
    Completable createCustomer(@BodyParameter Customer customer);
}
```

### Configuração

Em todos os casos acima, o `java-restify` irá executar a requisição em uma `thread` separada usando o [suporte do RxJava para processamento assíncrono](http://reactivex.io/documentation/scheduler.html), através do objeto [Scheduler](http://reactivex.io/documentation/scheduler.html). 

Por padrão, o `Scheduler` será criado a partir do método [Schedulers.io](http://reactivex.io/RxJava/1.x/javadoc/rx/schedulers/Schedulers.html#io), que é o mais adequado para requisições HTTP.

Caso essa configuração não atenda às necessidades da sua aplicação, desligue a descoberta automática de `handlers` e registre-os manualmente:

```java
import rx.Scheduler;
import rx.schedulers.Schedulers;
import com.github.ljtfreitas.restify.http.client.call.handler.rxjava.RxJavaObservableEndpointCallHandlerAdapter;
import com.github.ljtfreitas.restify.http.client.call.handler.rxjava.RxJavaCompletableEndpointCallHandlerFactory;
import com.github.ljtfreitas.restify.http.client.call.handler.rxjava.RxJavaSingleEndpointCallHandlerAdapter;

Scheduler myScheduler = Schedulers.from(Executors.newFixedThreadPool(10));

MyApi myApi = new RestifyProxyBuilder()
  .handlers()
    .discovery()
        .disabled()
    .add(new RxJavaObservableEndpointCallHandlerAdapter<>(myScheduler))
    .add(new RxJavaCompletableEndpointCallHandlerFactory(myScheduler))
    .add(new RxJavaSingleEndpointCallHandlerAdapter<>(myScheduler))
    .and()
  .target(MyApi.class)
    .build();
```