# Clientes HTTP - Requisições assíncronas

Os objetos `EndpointRequestExecutor` e `HttpClientRequestFactory` têm assinaturas de métodos **síncronas**: recebem um objeto que representa uma requisição e retornam uma resposta. Para requisições assíncronas, o `java-restify` apenas utiliza esses objetos em uma `thread` separada e as coisas funcionam conforme o esperado.

Mas eventualmente o próprio *client* HTTP irá fornecer algum mecanismo para requisições assíncronas (como I/O não-bloqueante ou algum objeto especializado, por exemplo), e não seria possível criar implementações de `EndpointRequestExecutor` ou `HttpClientRequestFactory` que utilizassem esses recursos. 

Para suportar esses casos de uso, existem variações assíncronas desses objetos: `AsyncEndpointRequestExecutor` e `AsyncHttpClientRequestFactory`, que utilizam `CompletableFuture` nas assinaturas de método, permitindo implementar o uso nativo de bibliotecas HTTP assíncronas.

O `java-restify` irá identificar se você está utilizando um *client* HTTP assíncrono, e as adaptações necessárias para chamadas de método síncronas/assíncronas serão feitas de maneira transparente.

## Configuração

O `thread pool` utilizado para requisições assíncronas é o mesmo configurado para execução de métodos assíncronos; por padrão é um `Executor` criado a partir do método [Executors.newCachedThreadPool](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Executors.html#newCachedThreadPool--)).

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
