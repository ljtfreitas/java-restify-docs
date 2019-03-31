# Tipos de retorno de método - Guava

[Guava](https://github.com/google/guava/wiki) é um framework muito conhecido pelos desenvolvedores Java, que fornece vários objetos e utilitários para diversas necessidades. O `java-restify` fornece suporte para uso de alguns objetos do Guava como retorno de método.

## Instalação

O suporte para o Guava está na dependência `java-restify-guava`. Uma vez presente no `classpath`, os `handlers` serão automaticamente registrados.

### Maven

```xml
<dependency>
  <groupId>com.github.ljtfreitas</groupId>
  <artifactId>java-restify-guava</artifactId>
  <version>{version}</version>
</dependency>
```

### Gradle

```groovy
dependencies {
  compile("com.github.ljtfreitas:java-restify-guava:{version}")
}
```

## Utilização

### Tipos suportados

* `Optional`

```java
import com.google.common.base.Optional;

public interface MyApi {

    @Path("/customers/{id}") @Get
    Optional<Customer> getCustomerById(@PathParameter String id);
}
```

* `ListenableFuture` (assíncrono)

[ListenableFuture](https://github.com/google/guava/wiki/ListenableFutureExplained) é um objeto que permite o registro de `callbacks` para processamentos assíncronos. Métodos com esse tipo de retorno serão executados em uma `thread` separada automaticamente.

```java
import com.google.common.util.concurrent.ListenableFuture;
import com.google.common.util.concurrent.FutureCallback;
import com.google.common.util.concurrent.Futures;
import com.google.common.util.concurrent.MoreExecutors;

public interface MyApi {

    @Path("/customers/{id}") @Get
    ListenableFuture<Customer> getCustomerById(@PathParameter String id);
}

MyApi myApi = new RestifyProxyBuilder()
  .target(MyApi.class)
    .build();

ListenableFuture<Customer> future = myApi.getCustomerById("abc123");

FutureCallback<Customer> callback = new FutureCallback<FutureCallback>() {
    
    void onSuccess(Customer result) {
    };

    void onFailure(Throwable t) {
    };
};

Futures.addCallback(future, callback, MoreExecutors.directExecutor());
```

Outra opção é, ao invés de lidar com o retorno do método, utilizar um argumento do tipo [FutureCallback](https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/FutureCallback.html), que deve estar anotado com `@CallbackParameter`:

```java
import com.google.common.util.concurrent.ListenableFuture;
import com.google.common.util.concurrent.FutureCallback;
import com.google.common.util.concurrent.Futures;
import com.google.common.util.concurrent.MoreExecutors;

public interface MyApi {

    @Path("/customers/{id}") @Get
    void getCustomerById(@PathParameter String id, @CallbackParameter FutureCallback<Customer> callback);
}

MyApi myApi = new RestifyProxyBuilder()
  .target(MyApi.class)
    .build();

myApi.getCustomerById("abc123", new FutureCallback<FutureCallback>() {
    
    void onSuccess(Customer result) {
    };

    void onFailure(Throwable t) {
    };
};
```

* `ListenableFutureTask` (assíncrono)

[ListenableFutureTask](https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ListenableFutureTask.html) é equivalente ao `ListenableFuture`, mas com uma API baseada no `FutureTask` do Java.

```java
import com.google.common.util.concurrent.ListenableFutureTask;
import com.google.common.util.concurrent.FutureCallback;
import com.google.common.util.concurrent.Futures;
import com.google.common.util.concurrent.MoreExecutors;

public interface MyApi {

    @Path("/customers/{id}") @Get
    ListenableFutureTask<Customer> getCustomerById(@PathParameter String id);
}
```

### Configuração

No caso dos tipos assíncronos, a configuração padrão utiliza um [cached thread pool](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Executors.html#newCachedThreadPool-java.util.concurrent.ThreadFactory-) isolado dos demais `handlers` assíncronos do `java-restify`. Caso você precise de customizações, desligue a descoberta automática de `handlers` e registre-os manualmente:

```java
import com.google.common.util.concurrent.MoreExecutors;
import com.google.common.util.concurrent.ListeningExecutorService;
import com.github.ljtfreitas.restify.http.client.call.handler.guava.ListenableFutureEndpointCallHandlerAdapter;
import com.github.ljtfreitas.restify.http.client.call.handler.guava.ListenableFutureCallbackEndpointCallHandlerAdapter;
import com.github.ljtfreitas.restify.http.client.call.handler.guava.ListenableFutureTaskEndpointCallHandlerAdapter;

Executor myExecutor = Executors.newFixedThreadPool(10);

ListeningExecutorService myExecutorService = MoreExecutors.listeningDecorator(myExecutor);

MyApi myApi = new RestifyProxyBuilder()
  .handlers()
    .discovery()
        .disabled()
    .add(new ListenableFutureEndpointCallHandlerAdapter<>(myExecutorService))
    .add(new ListenableFutureCallbackEndpointCallHandlerAdapter<>(myExecutorService))
    .add(new ListenableFutureTaskEndpointCallHandlerAdapter<>(myExecutorService))
    .and()
  .target(MyApi.class)
    .build();
```