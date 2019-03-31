# Serialização e deserialização - Formatos - application/json - JSON-B

[JSON-B](http://json-b.net/) é uma especifição Java para `binding` entre JSON e objetos.

## Instalação

Para utilizar, inclua a dependência `java-restify-json-jsonb-converter`. O `converter` `JsonBMessageConverter` será automaticamente registrado.

### Maven

```xml
<dependency>
  <groupId>com.github.ljtfreitas</groupId>
  <artifactId>java-restify-json-jsonb-converter</artifactId>
  <version>{version}</version>
</dependency>
```

### Gradle

```groovy
dependencies {
  compile("com.github.ljtfreitas:java-restify-json-jsonb-converter:{version}")
}
```

## Utilização

```java
public interface MyApi {

  /* requisições e respostas com o content-type application/json 
  serão automaticamente serializadas/deserializadas usando o json-b */

  @Path("/customers") @Post
  @JsonContent
  Customer createCustomer(@BodyParameter Customer customer);

  @Path("/customers/{id}") @Get
  Customer findCustomer(@PathParameter String id);
}
```

A implementação da classe `JsonBMessageConverter` utiliza uma instância do objeto [Jsonb](https://static.javadoc.io/javax.json.bind/javax.json.bind-api/1.0/index.html?java.json.bind-summary.html) com configurações padrão. Se você precisar de outras modificações, seu código pode gerar o objeto `Jsonb` ou uma instância de [JsonbConfig](https://static.javadoc.io/javax.json.bind/javax.json.bind-api/1.0/javax/json/bind/JsonbConfig.html), e instanciar o `JsonBMessageConverter` manualmente:

```java
import com.github.ljtfreitas.restify.http.client.message.converter.json.JsonBMessageConverter;
import javax.json.bind.JsonbConfig;

JsonbConfig customJsonbConfig = new JsonbConfig()
    .withNullValues(true); //para fins de exemplo; existem várias configurações disponíveis

MyApi myApi = new RestifyProxyBuilder()
  .converters()
    .add(new JsonBMessageConverter(customJsonbConfig))
    .and()
  .target(MyApi.class)
    .build();
```

```java
import com.github.ljtfreitas.restify.http.client.message.converter.json.JsonBMessageConverter;
import javax.json.bind.Jsonb;
import javax.json.bind.JsonbConfig;
import javax.json.bind.JsonbBuilder;

JsonbConfig customJsonbConfig = new JsonbConfig()
    .withNullValues(true); 

Jsonb myJsonb = JsonbBuilder.create(customJsonbConfig);

MyApi myApi = new RestifyProxyBuilder()
  .converters()
    .add(new JsonBMessageConverter(myJsonb))
    .and()
  .target(MyApi.class)
    .build();
```
