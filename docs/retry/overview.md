# Retry

O `java-restify` oferece suporte para `retry` de requisições, síncronas ou assíncronas.

Por padrão, o suporte para `retry` é desabilitado, e deve ser explicitamente habilitado.

```java
MyApi myApi = new RestifyProxyBuilder()
    .retry()
        .enabled()
    .target(MyApi.class)
        .build();
```

## Configuração

As configurações podem ser realizadas no próprio `builder`, e serão aplicadas para todos os métodos da interface:

```java
MyApi myApi = new RestifyProxyBuilder()
    .retry()
        .enabled()
        .configure()
            .attempts(3) // número de tentativas (padrão: 1 (sem retry))
            .timeout(Duration.ofMillis(10000)) // timeout máximo para as tentativas, pode ser informado também em milisegundos (opcional)
            .backOff() // configuração de backoff
                .delay(Duration.ofMillis(2000)) // período entre cada tentiva, pode ser informado também em milisegundos (padrão é 1000 milisegundos, aplicável apenas se attempts > 1)
                .multiplier(1.5) // fator de multiplicação de tempo entre cada tentativa (padrão é 1, sem efeito prático)
                .and()
            .when(IOException.class)
			.when(HttpStatusCode.INTERNAL_SERVER_ERROR, HttpStatusCode.BAD_GATEWAY)
    .target(MyApi.class)
        .build();
```

Outra configuração necessária é o tipo de situação onde o `retry` deve ser executado. Novamente, o comportamento padrão é **não** aplicar nenhum tipo de retentativa, a não ser para os cenários explicitamente configurados:

```java
MyApi myApi = new RestifyProxyBuilder()
    .retry()
        .enabled()
        .configure()
            // ... outras configurações que serão aplicadas quando...
            .when(IOException.class) // ocorrerem exceções do tipo IOException
			.when(HttpStatusCode.INTERNAL_SERVER_ERROR, HttpStatusCode.BAD_GATEWAY) // respostas HTTP com esses status codes
    .target(MyApi.class)
        .build();
```

É possível configurar vários cenários de erro usando do método `when`, quantos forem necessários. Existem outras sobrecargas que permitem ajustes ainda mais finos:

```java
import com.github.ljtfreitas.restify.http.client.retry.RetryCondition.EndpointResponseRetryCondition;

// condição de retry utilizando uma instância de EndpointResponse
EndpointResponseRetryCondition condition = response -> response.body().equals("fail") && response.headers().get("X-Fail").isPresent();

MyApi myApi = new RestifyProxyBuilder()
    .retry()
        .enabled()
        .configure()
            // ... outras configurações que serão aplicadas quando...
            .when(condition) // a condição for satisfeita
    .target(MyApi.class)
        .build();
```

```java
import com.github.ljtfreitas.restify.http.client.retry.RetryCondition.HeadersRetryCondition;

// condição de retry utilizando a coleção de headers da resposta
HeadersRetryCondition condition = headers -> headers.get("X-Should-Retry").isPresent();

/* shortcuts:

    condition = HeadersRetryCondition.contains("X-Should-Retry");
    
    condition = HeadersRetryCondition.contains(Header.of("X-Should-Retry", "true")); // para verificar também o valor do header
*/

MyApi myApi = new RestifyProxyBuilder()
    .retry()
        .enabled()
        .configure()
            // ... outras configurações que serão aplicadas quando...
            .when(condition) // a condição for satisfeita
    .target(MyApi.class)
        .build();
```

```java
import com.github.ljtfreitas.restify.http.client.retry.RetryCondition.StatusCodeRetryCondition;

// condição de retry utilizando o status code da resposta
StatusCodeRetryCondition condition =  status -> status.isInternalServerError();

/* shortcuts:

    condition = StatusCodeRetryCondition.any(HttpStatusCode.INTERNAL_SERVER_ERROR, HttpStatusCode.BAD_GATEWAY);

    condition = StatusCodeRetryCondition.any4xx();

    condition = StatusCodeRetryCondition.any5xx();
*/
MyApi myApi = new RestifyProxyBuilder()
    .retry()
        .enabled()
        .configure()
            // ... outras configurações que serão aplicadas quando...
            .when(condition) // a condição for satisfeita
    .target(MyApi.class)
        .build();
```

```java
import com.github.ljtfreitas.restify.http.client.retry.RetryCondition.ThrowableRetryCondition;

// condição de retry utilizando a exceção
ThrowableRetryCondition condition =  e -> (e instanceof IOException);

/* shortcuts:

    condition = ThrowableRetryCondition.any(IOException.class);

    condition = ThrowableRetryCondition.ioFailure();
*/
MyApi myApi = new RestifyProxyBuilder()
    .retry()
        .enabled()
        .configure()
            // ... outras configurações que serão aplicadas quando...
            .when(condition) // a condição for satisfeita
    .target(MyApi.class)
        .build();
```

### @Retry

Também é possível aplicar configurações mais específicas no nível do método, usando a anotação `@Retry`. Essa anotação tem parâmetros equivalentes às configurações disponíveis no `RestifyProxyBuilder`.

Essa anotação só é lida e processada se o `retry` for habilitado, conforme demonstrado acima.

```java
public interface MyApi {

    @Path("/customers/{id}") @Get
    @Retry(
        attempts = 3, // número de tentativas (padrão: 1 (sem retry))
        timeout = 10000, // timeout máximo para as tentativas em milisegundos (opcional)
	    on4xxStatus = true, // retry para status codes 4xx (padrão false)
	    on5xxStatus = true, // retry para status codes 5xx (padrão false)
	    onIOFailure = true, // retry para erros do tipo IOException (padrão false)
	    status = {HttpStatusCode.INTERNAL_SERVER_ERROR, HttpStatusCode.BAD_GATEWAY}, // lista de status codes para retry (padrão é nenhum)
	    exceptions = SocketException.class, // lista de exceções para retry (padrão é nenhuma)
        backoff = @BackOff( // configurações de backoff
			        delay = 2000, // período entre cada tentativa, em milisegundos (padrão é 1000 milisegundos, aplicável apenas se attempts > 1)
				    multiplier = 1.5 // fator de multiplicação de tempo entre cada tentativa (padrão é 1, sem efeito prático)
			    )
        )
    Customer findCustomer(@PathParameter String id);
```

Se a anotação `@Retry` estiver presente no topo da interface, a configuração será aplicada a todos os métodos.

### Requisiçoes assíncronas

Todas as configurações demonstradas acima também se aplicam para requições assíncronas. A implementação padrão utiliza um [ScheduledExecutorService](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ScheduledExecutorService.html), criado com uma configuração [single-thread](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Executors.html#newSingleThreadScheduledExecutor--).

Caso essa configuração não atenda as necessidades da sua aplicação, é bastante simples configurar um novo `ScheduledExecutorService`:

```java
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;

public interface MyApi {

    // CompletableFuture = requisição assíncrona
    // as retentativas também serão executas em thread separadas

    @Path("/customers/{id}") @Get
    @Retry(attempts = 3, on5xxStatus = true, backoff = @BackOff(delay = 2000))
    CompletableFuture<Customer> findCustomer(@PathParameter String id); 
}

ScheduledExecutorService myScheduler = Executors.newScheduledThreadPool(10);

MyApi myApi = new RestifyProxyBuilder()
    .retry()
        .enabled()
        .async()
            .scheduler(myScheduler)
    .target(MyApi.class)
        .build();
```
