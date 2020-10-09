## Sobre o Hystrix

Hystrix é uma biblioteca de latência e tolerância a falhas projetada para isolar pontos de acesso a sistemas remotos, serviços e bibliotecas de terceiros, interromper as falhas em cascata e permitir a resiliência em sistemas distribuídos complexos onde a falha é inevitável.

Importante: a biblioteca Hystrix não é mais mantida pela Netflix, sendo recomendado o uso do [resilience4j](https://github.com/resilience4j/resilience4j)

# Instalação

A partir da versão 2.x.x, os componentes necessários para utilização do Hystrix foram movidos para outro artefato. Para instalá-lo, é necessário colocar a dependência `java-restify-netflix-hystrix`

## Maven

```xml
<dependency>
    <groupId>com.github.ljtfreitas</groupId>
    <artifactId>java-restify-netflix-hystrix</artifactId>
    <version>2.1.0</version>
</dependency>
```

## Gradle

```groovy
compile group: 'com.github.ljtfreitas', name: 'java-restify-netflix-hystrix', version: '2.1.0'
```

# Utilização

Há duas formas de consumir uma API utilizando o Hystrix no Restify: retornando um `HystrixCommand<T>`, ou utilizando a annotation `OnCircuiBreaker`

## HystrixCommand

Considerando a interface `MyApi`:


```java
public interface MyApi {
    @Path("/products")
    @Get    
    HystrixCommand<SearchResult> searchProducts(@QueryParameter("name") String productName);
}
```

E a classe `MyApiFallback`

```java
class MyApiFallback implements MyApi {
    HystrixCommand<SearchResult> searchProducts(String productName) {
        return new MyApiFallbackCommand();
    }

    class MyApiFallbackCommand implements HystrixCommand<SearchResult> {
        // Setup do HystrixCommand
    }
}
```

É necessário criar um producer com as seguintes configurações:

```java
import com.github.ljtfreitas.restify.http.RestifyProxyBuilder;
import com.github.ljtfreitas.restify.http.client.apache.httpclient.ApacheHttpClientRequestFactory;
import com.github.ljtfreitas.restify.http.netflix.client.call.handler.hystrix.HystrixCommandEndpointCallHandlerAdapter;

ApacheHttpClientRequestFactory apacheHttpClientRequestFactory = new ApacheHttpClientRequestFactory(httpClient);

Setter setter = ???

HystrixCommandEndpointCallHandlerAdapter commandEndpointCallHandlerAdapter = new HystrixCommandEndpointCallHandlerAdapter(setter);

return new RestifyProxyBuilder()
    .client(apacheHttpClientRequestFactory)        
    .handlers().add(commandEndpointCallHandlerAdapter).and()        
    .target(MyApi.class, apiHost).build();
```

Este código já executa um `HystrixCommand` com suporte a _circuit breaker_, mas ainda é necessário implementar uma classe de _fallback_. Para fazer isso, basta criar uma classe que implemente a interface da sua api retornando os valores de _fallback_ desejados e criar um objeto do tipo `Fallback` com a classe criada. Exemplo:

```java
import com.github.ljtfreitas.restify.http.client.call.handler.circuitbreaker.Fallback;
import com.github.ljtfreitas.restify.http.RestifyProxyBuilder;
import com.github.ljtfreitas.restify.http.client.apache.httpclient.ApacheHttpClientRequestFactory;
import com.github.ljtfreitas.restify.http.netflix.client.call.handler.hystrix.HystrixCommandEndpointCallHandlerAdapter;

ApacheHttpClientRequestFactory apacheHttpClientRequestFactory = new ApacheHttpClientRequestFactory(httpClient);

Setter setter = ???

Fallback fallback = Fallback.of(new MyApiFallback());

HystrixCommandEndpointCallHandlerAdapter commandEndpointCallHandlerAdapter = new HystrixCommandEndpointCallHandlerAdapter(setter, fallback);

return new RestifyProxyBuilder()
    .client(apacheHttpClientRequestFactory)        
    .handlers().add(commandEndpointCallHandlerAdapter).and()        
    .target(MyApi.class, apiHost).build();
```

## OnCircuitBreaker

Utilizar o `HystrixCommand` é necessário quando você precisa, por exemplo, receber os resultados de forma assícrona utilizando a API de Futures. Na maioria dos casos, retornar somente o objeto com o resultado já é o suficiente. Neste caso, podemos retornar não mais um `HystrixCommand`, mas sim o tipo do payload que iremos receber do servidor; e basta anotarmos o método `searchProducts` da nossa interface com as _annotations_ `OnCircuitBreaker` e `WithFallback`:

```java
public interface MyApi {
    @Path("/products")
    @Get    
    @OnCircuitBreaker
    @WithFallback(value = MyApiFallback.class)
    SearchResult searchProducts(@QueryParameter("name") String productName);
}
```

Dessa form, a nossa classe de _fallback_ fica com uma implementação mais simples:

```java
class MyApiFallback implements MyApi {
    SearchResult searchProducts(String productName) {
        return SearchResult.empty();
    }
}
```

Uma vez alterada a interface da api, temos que registrar um _adapter_ chamado `HystrixEndpointCallHandlerAdapter` juntamente com o `HystrixCommandEndpointCallHandlerAdapter`. Isso é necessário pois o Restify utiliza o `HystrixEndpointCallHandlerAdapter` para transformar o retoro do método em um `HystrixCommand` e o `HystrixCommandEndpointCallHandlerAdapter` para lidar com o `HystrixCommand` criado.

## Recuperando a exceção que causou o fallback

Pode ser necessário recuperar a exceção que causou a chamada do fallback. Para isso, ao invés de criarmos uma classe que implementa a interface `MyApi`, podemos usar qualquer método que possua a mesma assinatura que o método original com a adição de mais um parâmetro do tipo Throwable:

```java
class MyApiFallback {
    SearchResult searchProducts(String productName, Throwable t) {
        // Faz alguma coisa
    }
}

public interface MyApi {
    @Path("/products")
    @Get    
    @OnCircuitBreaker
    @WithFallback(value = MyApiFallback.class, method = "searchProducts")
    SearchResult searchProducts(@QueryParameter("name") String productName);
}
```

Isso irá fazer com que o Restify chame o método passando a exceção que causou o problema.