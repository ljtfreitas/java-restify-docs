# Manipulação de erros

O `java-restify` oferece diferentes abordagens para manipulação e recuperação de erros. 

Além dos erros de I/O (comunicação, rede, streams, etc) que podem ocorrer durante a requisiçao HTTP, o `java-restify` converte respostas HTTP 4xx (`client error`) e 5xx (`server error`) para exceções. Esse é o comportamento padrão, mas pode ser customizado para atender diferentes necessidades.

Todas as exceções do `java-restify` extendem `com.github.ljtfreitas.restify.http.client.HttpException`. Exceções ocorridas durante a execução do método ou durante a requisição HTTP sempre serão encapsuladas e propagadas em uma exceção do tipo `HttpException`.

Existem duas especializações dessa exceção: 

* `HttpClientException`, que representa erros de I/O

* `HttpMessageException`, para exceções durante a manipulação da requisição e da resposta. 

`HttpMessageException` possui uma subclasse chamada `EndpointResponseException`, que representa uma resposta HTTP de erro (`status code` 4xx ou 5xx). Essa classe possui várias sub-exceções, cada uma representando um status de erro HTTP específico. Isso permite implementar um controle fino para situações específicas:

```java
public interface MyApi {

    @Path("/customers/{id}") @Get
    Customer getCustomerById(@PathParameter String id);
}

MyApi myApi = new RestifyProxyBuilder()
    .target(MyApi.class)
        .build();

try {
    Customer customer = myApi.getCustomerById("abc123");

} catch (EndpointResponseNotFoundException e) {
    // 404 Not Found

} catch (EndpointResponseUnauthorizedException e) {
    // 401 Unauthorized

} catch (EndpointResponseNotAcceptableException e) {
    // 406 Not Acceptable

} catch (EndpointResponseException e) {
    // qualquer outra resposta de erro HTTP

} catch (HttpClientException e) {
    // erros de I/O

}
```

## Requisições assíncronas

Caso a requisição seja assíncrona, o tratamento de falhas deve ser implementado usando o próprio objeto de retorno ou com o uso de `callbacks`.

Por exemplo, caso o retorno seja um `CompletableFuture`:

```java
public interface MyApi {

    @Path("/customers/{id}") @Get
    CompletableFuture<Customer> getCustomerById(@PathParameter String id);
}

MyApi myApi = new RestifyProxyBuilder()
    .target(MyApi.class)
        .build();

myApi.getCustomerById("abc123")
        .whenComplete((customer, exception) -> {
            // lógica de falha ou sucesso
        });
```

Ou utilizando callbacks:

```java
public interface MyApi {

    @Path("/customers/{id}") @Get
    void getCustomerById(@PathParameter String id, @CallbackParameter EndpointCallSuccessCallback<Customer> success, @CallbackParameter EndpointCallFailureCallback failure);
}

MyApi myApi = new RestifyProxyBuilder()
    .target(MyApi.class)
        .build();

myApi.getCustomerById("abc123", 
        customer -> { /* lógica de sucesso */},
        exception -> { /*lógica de falha */});
```

A interface `EndpointCallFailureCallback` (representada acima como uma expressão *lambda*) tem um único método, `onFailure`, que recebe um `Throwable` como parâmetro, permitindo o tratamento de qualquer exceção; existe uma especialização chamada `EndpointResponseFailureCallback` específica para respostas de erro, que permite uma manipulação fina de cada cenário de erro:

```java
public interface MyApi {

    @Path("/customers/{id}") @Get
    void getCustomerById(@PathParameter String id, @CallbackParameter EndpointCallSuccessCallback<Customer> success, @CallbackParameter EndpointResponseFailureCallback failure);
}

MyApi myApi = new RestifyProxyBuilder()
    .target(MyApi.class)
        .build();

EndpointResponseFailureCallback failure = new EndpointResponseFailureCallback() { // classe abstrata

    // sobrescreva os métodos para cada resposta de erro que deseje manipular

    @Override
    protected void onNotFound(EndpointResponse<String> response) {
        // 404 Not Found
    }

    @Override
    protected void onUnauthorized(EndpointResponse<String> response) {
        // 404 Not Found
    }

    @Override
    protected void onNotAcceptable(EndpointResponse<String> response) {
        // 404 Not Found
    }

    protected void onFailure(EndpointResponse<String> response) {
        // qualquer outra resposta de erro HTTP
    }

    protected void onException(Throwable throwable) {
        // qualquer outra exceção, incluindo erros de I/O
    }
};

myApi.getCustomerById("abc123", 
        customer -> { /* lógica de sucesso */},
        failure);
```

## Recuperação de respostas de erro

Além da captura da exceção, através de `try/catch` ou `callback` de falha, outra abordagem possível é implementar estratégias para recuperação de respostas de erro.

O objeto `EndpointResponseErrorFallback` é responsável pela manipulação de respostas 4xx e 5xx, podendo eventualmente retornar outra resposta. A implementação padrão é o comportamento demonstrado acima: a propagação de exceções específicas por tipo de resposta.

Mas implementações dessa interface podem implementar qualquer comportamento sobre respostas de erro. Por exemplo, um caso especial é o `status code` **404** (`Not Found`); respostas com esse `status` podem ser tratadas como uma resposta de corpo vazio, ao invés de uma exceção.

```java
public interface MyApi {

    // a API devolve 404, caso não encontre um Customer com o id
    @Path("/customers/{id}") @Get
    Customer getCustomerById(@PathParameter String id);
}

MyApi myApi = new RestifyProxyBuilder()
    .error()
        .emptyOnNotFound() // respostas 404 serão consideradas como uma resposta sem corpo
    .target(MyApi.class)
        .build();

// se a resposta for 404, o retorno do método será null
Customer customer = myApi.getCustomerById("xyz123");
```

Uma solução elegante para a situação acima seria utilizar um `Optional` como retorno de método, representando uma resposta potencialmente vazia:

```java
public interface MyApi {

    // a API devolve 404, caso não encontre um Customer com o id
    @Path("/customers/{id}") @Get
    Optional<Customer> getCustomerById(@PathParameter String id);
}

MyApi myApi = new RestifyProxyBuilder()
    .error()
        .emptyOnNotFound() // respostas 404 serão consideradas como uma resposta sem corpo
    .target(MyApi.class)
        .build();

// se a resposta for 404, o retorno do método será um Optional vazio
Optional<Customer> customer = myApi.getCustomerById("xyz123");
```

Também é possível implementar qualquer outra lógica para recuperação de respostas 4xx ou 5xx:

```java
class MyResponseErrorFallback implements EndpointResponseErrorFallback {

    @Override
    public <T> EndpointResponse<T> onError(HttpResponseMessage response, JavaType responseType) {
        /* 
            HttpResponseMessage é um objeto de nível mais baixo, que fornece acesso à resposta HTTP "crua".
            O segundo parâmetro representa o tipo de retorno do método.
            Implemente sua lógica de recuperação para retornar um EndpointResponse compatível com o tipo de retorno esperado, ou lançe uma exceção mais adequada ao seu domínio.
        */
    }
}

MyApi myApi = new RestifyProxyBuilder()
    .error(new MyResponseErrorFallback())
    .target(MyApi.class)
        .build();

// ou

MyApi myApi = new RestifyProxyBuilder()
    .error()
        .using(new MyResponseErrorFallback())
    .target(MyApi.class)
        .build();

```

### EndpointResponse

Conforme discutido na [documentação sobre tipos de retorno](../method-return/default-types.md), o objeto `EndpointResponse` representa a resposta HTTP completa, e pode ser utilizado como retorno de método.

Esse objeto possui alguns métodos que permitem implementar uma lógica de recuperação de falhas, em caso de respostas 4xx ou 5xx.

```java
public interface MyApi {

    @Path("/customers/{id}") @Get
    EndpointResponse<Customer> getCustomerById(@PathParameter String id);
}

MyApi myApi = new RestifyProxyBuilder()
  .target(MyApi.class)
    .build();

EndpointResponse<Customer> response = myApi.getCustomerById("abc123");

// o método recover recebe o tipo de exceção que deve ser manipulada, e uma funcão que retorna uma nova resposta
Customer customer = response.recover(EndpointResponseNotFoundException.class, e -> EndpointResponse.empty(StatusCode.ok()));
```

O `EndpointResponse` deve ser manipulado com cuidado; ao tentar acessar o corpo, em caso de respostas de erro, será lançada a exceção correspondente ao `status code`:

```java
public interface MyApi {

    @Path("/customers/{id}") @Get
    EndpointResponse<Customer> getCustomerById(@PathParameter String id);
}

MyApi myApi = new RestifyProxyBuilder()
  .target(MyApi.class)
    .build();

// digamos que a resposta desse request foi 500 (Internal Server Error)
EndpointResponse<Customer> response = myApi.getCustomerById("abc123");

Customer customer = response.body(); // será lançada uma exceção do tipo EndpointResponseInternalServerErrorException

```
