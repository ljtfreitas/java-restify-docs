# Serialização e deserialização - Formatos - application/json - Jackson

## Instalação

Para utilizar o [Jackson](https://github.com/FasterXML/jackson), adicione a dependência `java-restify-json-jackson-converter`. O `converter` `JacksonMessageConverter` será automaticamente registrado.

### Maven

```xml
<dependency>
  <groupId>com.github.ljtfreitas</groupId>
  <artifactId>java-restify-json-jackson-converter</artifactId>
  <version>{version}</version>
</dependency>
```

### Gradle

```groovy
dependencies {
  compile("com.github.ljtfreitas:java-restify-json-jackson-converter:{version}")
}
```

## Utilização

```java
public interface MyApi {
  
  /* requisições e respostas com o content-type application/json
  serão automaticamente serializadas/deserializadas usando o jackson */

  @Path("/customers/{id}") @Get
  Customer findCustomer(@PathParameter String id);

  @Path("/customers") @Post
  @JsonContent
  Customer createCustomer(@BodyParameter Customer customer);
}
```

A implementação da classe `JacksonMessageConverter` utiliza uma instância do [ObjectMapper](https://fasterxml.github.io/jackson-databind/javadoc/2.9/com/fasterxml/jackson/databind/ObjectMapper.html) do Jackson com configurações padrão. A única customização é a *feature* [FAIL_ON_UNKNOWN_PROPERTIES](https://fasterxml.github.io/jackson-databind/javadoc/2.9/com/fasterxml/jackson/databind/DeserializationFeature.html#FAIL_ON_UNKNOWN_PROPERTIES), que é definida como `false`. 

Caso você precise de outras modificações, seu código pode gerar o `ObjectMapper` e instanciar o `JacksonMessageConverter` manualmente:

```java
import com.github.ljtfreitas.restify.http.client.message.converter.json.JacksonMessageConverter;
import com.fasterxml.jackson.databind.ObjectMapper;

ObjectMapper myObjectMapper = new ObjectMapper();
//configura o objectMapper da maneira que desejar...

MyApi myApi = new RestifyProxyBuilder()
  .converters()
    .add(new JacksonMessageConverter(myObjectMapper))
    .and()
  .target(MyApi.class)
    .build();
```
