# Requisições assíncronas

O `java-restify` oferece suporte para requisições assíncronas de maneira bastante simples.

Um detalhe de implementação importante é que o `java-restify` diferencia a execução assíncrona do **método** da execução assíncrona da **requisição**.

## Tipos de retorno assíncronos

Conforme comentado na [documentação sobre tipos de retorno de método](../method-return/default-types), alguns tipos, como `Future`, `CompletableFuture`, irão automaticamente forçar que o método seja executado em uma `thread` diferente.

```java
public interface MyApi {

    @Path("/customers/{id}") @Get
    CompletableFuture<Customer> getCustomerById(@PathParameter String id);
}

MyApi myApi = new RestifyProxyBuilder()
    .target(MyApi.class)
        .build();

myApi.getCustomerById("abc123")
    .thenAccept(customer -> ...) //outra thread
```

Outro tipo que pode ser utilizado como retorno do método é o `AsyncEndpointCall`, um objeto fornecido pelo  `java-restify` que representa a execução assíncrona de uma requisição.

```java
public interface MyApi {

    @Path("/customers/{id}") @Get
    AsyncEndpointCall<Customer> getCustomerById(@PathParameter String id);
}

MyApi myApi = new RestifyProxyBuilder()
    .target(MyApi.class)
        .build();

AsyncEndpointCall<Customer> call = myApi.getCustomerById("abc123"); // lazy - a requisição ainda não foi realizada

call.executeAsync()
    .thenAccept(customer -> ...) //outra thread
```

### Callbacks

Outra abordagem é, ao invés de lidar com o retorno do método, é utilizar argumentos de `callback`, usando a anotação `@CallbackParameter`:

Ao utilizar parâmetros anotados com `@CallbackParameter`, o retorno do método **deve** ser `void`.

```java
import com.github.ljtfreitas.restify.http.contract.CallbackParameter;
import com.github.ljtfreitas.restify.http.client.call.async.EndpointCallSuccessCallback;
import com.github.ljtfreitas.restify.http.client.call.async.EndpointCallFailureCallback;
import com.github.ljtfreitas.restify.http.client.call.async.EndpointCallCallback;

public interface MyApi {

    /* callback do tipo java.uil.function.BiConsumer: 
       uma função que recebe o objeto de resposta e a exceção  (se houver)
    */
    @Path("/customers/{id}") @Get
    void getCustomerById(@PathParameter String id, @CallbackParameter BiConsumer<Customer, Throwable> callback);

    /* EndpointCallSuccessCallback permite capturar a resposta deserializada como um objeto.
       Essa interface possui um único método onSuccess(T response) 
    */
    @Path("/customers/{id}") @Get
    void getCustomerById(@PathParameter String id, @CallbackParameter EndpointCallSuccessCallback<Customer> success);

    /* EndpointCallFailureCallback permite capturar a exceção gerada pela requisição HTTP, se houver.
       Essa exceção pode ser um problema de I/O ou uma resposta de erro (4xx, 5xx)
       Essa interface possui um único método onFailure(Throwable throwable):
    */
    @Path("/customers/{id}") @Get
    void getCustomerById(@PathParameter String id, @CallbackParameter EndpointCallFailureCallback failure);

    /* É possível usar parâmetros dos dois tipos
    */
    @Path("/customers/{id}") @Get
    void getCustomerById(@PathParameter String id, @CallbackParameter EndpointCallSuccessCallback<Customer> success, @CallbackParameter EndpointCallFailureCallback failure);

    /* Existe uma terceira interface chamada EndpointCallCallback, que extende EndpointCallSuccessCallback e EndpointCallFailureCallback.
       Essa interface também é uma opção caso você precise dos dois callbacks (sucesso e falha)
    */
    @Path("/customers/{id}") @Get
    void getCustomerById(@PathParameter String id, @CallbackParameter EndpointCallCallback<Customer> callback);
}
```

### Configuração

Os `handlers` responsáveis pela execução de métodos assíncronos utilizam o mesmo `thread pool` configurado para requisições assíncronas. Por padrão, é utilizad um `Executor` criado a partir do método [Executors.newCachedThreadPool](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Executors.html#newCachedThreadPool--)).

Se a configuração padrão não atender as necessidades da sua aplicação, o `Executor` pode ser facilmente customizado:

```java
ExecutorService myExecutor = Executors.newFixedThreadPool(10);

MyApi myApi = new RestifyProxyBuilder()
    .async(myExecutor)
    .target(MyApi.class)
        .build();

// ou

MyApi myApi = new RestifyProxyBuilder()
    .async()
        .using(myExecutor)
    .target(MyApi.class)
        .build();
```

## Clientes HTTP assíncronos

Os exemplos acima demonstram o suporte do `java-restify` para execução assíncrona de **métodos**, usando o tipo de retorno ou argumentos de `callback`. Nesse cenário, o `java-restify` irá apenas executar em outra `thread` os mesmos passos de qualquer chamada de método. O ponto a ser observado aqui é que a requisição HTTP continua a ser uma operação bloqueante, mas sendo realizada em uma `thread` separada.

Isso é útil, mas eventualmente pode ser importante utilizarmos os recursos assíncronos do próprio `client` HTTP, como por exemplo I/O não-bloqueante ou algum suporte especializado fornecido pela própria biblioteca. O `java-restify` também oferece uma API específica para clientes HTTP assíncronos. Os detalhes são explicados mais profundamente na [documentação sobre clientes HTTP](../http-clients/async.md).

Se o `java-restify` estiver configurado para utilizar um cliente HTTP assíncrono, então os recursos específicos da implementação serão utilizados; se não for o caso (como é o padrão), a execução do método (incluindo a requisição HTTP) apenas será feita em uma `thread` separada (conforme detalhado acima).