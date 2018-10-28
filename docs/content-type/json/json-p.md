# Serialização e deserialização - Formatos - application/json - JSON-P

[JSON-P](https://javaee.github.io/jsonp/) é uma especificação Java que fornece uma API para processamento de JSON, bastante simples e interessante. O JSON-P não faz `binding` entre o JSON e classes, mas fornece objetos que representam a estrutura do JSON.

## Instalação

Para utilizar o JSON-P, inclua a dependência `java-restify-json-jsonp-converter`. O `converter` `JsonPMessageConverter` será automaticamente registrado. 

### Maven

```xml
<dependency>
  <groupId>com.github.ljtfreitas</groupId>
  <artifactId>java-restify-json-jsonp-converter</artifactId>
  <version>{version}</version>
</dependency>
```

### Gradle

```groovy
dependencies {
  compile("com.github.ljtfreitas:java-restify-json-jsonp-converter:{version}")
}
```
Esse `converter` permite utilizar apenas objetos da API do JSON-P (objetos do tipo [JsonStructure](https://static.javadoc.io/javax.json/javax.json-api/1.1.4/javax/json/JsonStructure.html)).


## Utilização

```java
import javax.json.JsonObject;

public interface MyApi {

  /* requisições e respostas com o content-type application/json 
  serão automaticamente serializadas/deserializadas com o json-p */

  @Path("/customers/{id}") @Get
  JsonObject findCustomer(@PathParameter String id);

  @Path("/customers") @Post
  @JsonContent
  void createCustomer(@BodyParameter JsonObject customer);
}
```

A implementação da classe `JsonpMessageConverter` utiliza os objetos [JsonReaderFactory](https://static.javadoc.io/javax.json/javax.json-api/1.1.4/javax/json/JsonReaderFactory.html) e [JsonWriterFactory](https://static.javadoc.io/javax.json/javax.json-api/1.1.4/javax/json/JsonWriterFactory.html) com configurações padrão (que dependem do provider JSON-P que estiver sendo utilizado). Se você precisar de mais customizações, você pode gerar um mapa de configurações e instanciar manualmente o `JsonPMessageConverter`:

```java
import com.github.ljtfreitas.restify.http.client.message.converter.json.JsonPMessageConverter;

// customizações dependem do provider JSON-P
// mais detalhes em: https://static.javadoc.io/javax.json/javax.json-api/1.1.4/javax/json/Json.html#createReaderFactory-java.util.Map- e https://static.javadoc.io/javax.json/javax.json-api/1.1.4/javax/json/Json.html#createWriterFactory-java.util.Map-
Map<String, ?> myConfiguration = new HashMap<>();

MyApi myApi = new RestifyProxyBuilder()
  .converters()
    .add(new JsonPMessageConverter(myConfiguration))
    .and()
  .target(MyApi.class)
    .build();
```
