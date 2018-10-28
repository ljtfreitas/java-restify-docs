# Retornos de método - Tipos padrão

## Tipos suportados

Por padrão, os seguintes tipos de retorno de método são suportados:

* `String`

```java
public interface MyApi {

    @Path("/customers/{id}") @Get
    String getCustomerAsString(@PathParameter String id);
}
```

* Tipos primitivos ou `wrappers`

```java
public interface MyApi {

    @Path("/integer") @Get
    int integer();

    @Path("/boolean") @Get
    Boolean boolean();

    @Path("/double") @Get
    double double();
}
```

* `byte[]`

```java
public interface MyApi {

    @Path("/customers/{id}") @Get
    byte[] getCustomerAsBytes(@PathParameter String id);
}
```

* `InputStream`

```java
public interface MyApi {

    @Path("/customers/{id}") @Get
    InputStream getCustomerAsStream(@PathParameter String id);
}
```

* `Optional`

```java
public interface MyApi {

    @Path("/customers/{id}") @Get
    Optional<Customer> getCustomerById(@PathParameter String id);
}
```

* `Collection`

```java
public interface MyApi {

    @Path("/customers") @Get
    Collection<Customer> getAllCustomers();
}
```

* `Stream`

```java
public interface MyApi {

    @Path("/customers") @Get
    Stream<Customer> getAllCustomers();
}
```

* `Enumeration`

```java
public interface MyApi {

    @Path("/customers") @Get
    Enumeration<Customer> getAllCustomers();
}
```

* `Iterator`

```java
public interface MyApi {

    @Path("/customers") @Get
    Iterator<Customer> getAllCustomers();
}
```

* `ListIterator`

```java
public interface MyApi {

    @Path("/customers") @Get
    ListIterator<Customer> getAllCustomers();
}
```

* `Iterable`

```java
public interface MyApi {

    @Path("/customers") @Get
    Iterable<Customer> getAllCustomers();
}
```

* `Queue`

```java
public interface MyApi {

    @Path("/customers") @Get
    Queue<Customer> getAllCustomers();
}
```

* `Callable`

[Callable](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Callable.html) é um objeto que representa uma computação qualquer que retorna algum valor. A requisição HTTP será feita de modo *lazy*, eventualmente em outra `thread` (o seu código será responsável pela execução do `Callable`).

```java
public interface MyApi {

    @Path("/customers/{id}") @Get
    Callable<Customer> getCustomerById(@PathParameter String id);
}
```

* `Runnable`

[Runnable](https://docs.oracle.com/javase/8/docs/api/java/lang/Runnable.html) é um objeto que representa uma computação qualquer, e não retorna nenhum valor. A requisição HTTP será feita de modo *lazy*, eventualmente em outra `thread` (o seu código será responsável pela execução do `Runnable`).

```java
public interface MyApi {

    @Path("/customers") @Post
    Runnable createCustomer(@BodyParameter Customer customer);
}
```

* `EndpointCall<>`

Esse objeto representa uma operação HTTP realizada pelo `java-restify`. O seu código será responsável por invocar o método `execute`.

```java
import com.github.ljtfreitas.restify.http.client.call.EndpointCall;

public interface MyApi {

    @Path("/customers/{id}") @Get
    EndpointCall<Customer> getCustomerById(@PathParameter String id);
}
```

* `Headers`

Esse objeto é uma coleção imutável de cabeçalhos. Quando utilizado como retorno de método, irá conter os `headers` da resposta.

```java
import com.github.ljtfreitas.restify.http.client.message.Headers;

public interface MyApi {

    @Path("/customers/{id}") @Head
    Headers getCustomerById(@PathParameter String id);
}
```

* `StatusCode`

Esse objeto representa o status HTTP da resposta.

```java
import com.github.ljtfreitas.restify.http.client.message.response.StatusCode;

public interface MyApi {

    @Path("/customers") @Post
    StatusCode createCustomer(@BodyParameter Customer customer);
}
```

* `EndpointResponse<>`

Esse objeto fornece acesso a todos os dados da resposta (incluindo cabeçalhos e `status code`). Também fornece acesso ao corpo, já deserializado para um objeto.

```java
import com.github.ljtfreitas.restify.http.client.response.EndpointResponse;

public interface MyApi {

    @Path("/customers/{id}") @Get
    EndpointResponse<Customer> getCustomerById(@PathParameter String id);
}
```

## Tipos assíncronos

Alguns tipos de retorno farão com que a requisição seja executada da maneira assíncrona automaticamente. De maneira simplificada, a requisição simplesmente será realizada em uma `thread` separada. O mecanismo é explicado com mais detalhes na [documentação sobre requisições assíncronas HTTP](../async/overview.md). 

O `java-restify` oferece suporte para vários tipos de retorno assíncronos, e os `handlers` para esses objetos são registrados automaticamente. Para tornar essa configuração explícita:

```java
MyApi myApi = new RestifyProxyBuilder()
  .handlers()
    .async() // habilita objetos assíncronos (são habilitados por padrão; use apenas caso deseje tornar a utilização explícita)
  .target(MyApi.class)
    .build();
```

Os seguintes tipos assíncronos são suportados:

* `Future`

```java
public interface MyApi {

    @Path("/customers/{id}") @Get
    Future<Customer> getCustomerById(@PathParameter String id);
}
```

* `CompletableFuture`

```java
public interface MyApi {

    @Path("/customers/{id}") @Get
    CompletableFuture<Customer> getCustomerById(@PathParameter String id);
}
```

* `FutureTask`

```java
public interface MyApi {

    @Path("/customers/{id}") @Get
    FutureTask<Customer> getCustomerById(@PathParameter String id);
}
```

* `AsyncEndpointCall`

Esse objeto é análogo ao `EndpointCall`, e fornece métodos para execução assíncrona.

```java
import com.github.ljtfreitas.restify.http.client.call.async.AsyncEndpointCall;

public interface MyApi {

    @Path("/customers/{id}") @Get
    AsyncEndpointCall<Customer> getCustomerById(@PathParameter String id);
}
```

### @CallbackParameter

Ao invés de lidar com o retorno do método, outra possibilidade é utilizar um parâmetro anotado com `@CallbackParameter`, que represente um `callback` para a execução assíncrona.

Ao utilizar parâmetros anotados com `@CallbackParameter`, o retorno do método **deve** ser `void`.

```java
import com.github.ljtfreitas.restify.http.contract.CallbackParameter;
import com.github.ljtfreitas.restify.http.client.call.async.EndpointCallSuccessCallback;
import com.github.ljtfreitas.restify.http.client.call.async.EndpointCallFailureCallback;
import com.github.ljtfreitas.restify.http.client.call.async.EndpointCallCallback;

public interface MyApi {

    /* callback do tipo java.util.function.BiConsumer: 
       uma função que recebe o objeto de resposta e a exceção (se houver)
    */
    @Path("/customers/{id}") @Get
    void getCustomerById(@PathParameter String id, @CallbackParameter BiConsumer<Customer, Throwable> callback);

    /* EndpointCallSuccessCallback permite capturar a resposta deserializada como um objeto.
       Essa interface possui um único método onSuccess(T response) 
    */
    @Path("/customers/{id}") @Get
    void getCustomerById(@PathParameter String id, @CallbackParameter EndpointCallSuccessCallback<Customer> success);

    /* EndpointCallFailureCallback permite capturar a exceção gerada pela requisição HTTP, se ocorrer.
       Essa exceção pode ser um problema de I/O ou uma resposta de erro (4xx, 5xx)
       Essa interface possui um único método onFailure(Throwable throwable):
    */
    @Path("/customers/{id}") @Get
    void getCustomerById(@PathParameter String id, @CallbackParameter EndpointCallFailureCallback failure);

    /* É possível usar parâmetros dos dois tipos
    */
    @Path("/customers/{id}") @Get
    void getCustomerById(@PathParameter String id,
        @CallbackParameter EndpointCallSuccessCallback<Customer> success,
        @CallbackParameter EndpointCallFailureCallback failure);

    /* Existe uma terceira interface chamada EndpointCallCallback, que extende EndpointCallSuccessCallback e EndpointCallFailureCallback.
       Essa interface também é uma opção caso você precise dos dois callbacks (sucesso e falha)
    */
    @Path("/customers/{id}") @Get
    void getCustomerById(@PathParameter String id, @CallbackParameter EndpointCallCallback<Customer> callback);
}
```

### Configuração

Os `handlers` responsáveis pela execução de métodos assíncronos utilizam o mesmo `thread pool` configurado para requisições assíncronas. A [documentação sobre requisições assíncronas](../async/overview.md) fornece mais detalhes de configuração e customizações.
