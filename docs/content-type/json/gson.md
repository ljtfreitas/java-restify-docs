# Serialização e deserialização - Formatos - application/json - Gson

## Instalação

Para utilizar o [Gson](https://github.com/google/gson), adicione a dependência `java-restify-json-gson-converter`. O `converter` `GsonMessageConverter` será automaticamente registrado.

### Maven

```xml
<dependency>
  <groupId>com.github.ljtfreitas</groupId>
  <artifactId>java-restify-json-gson-converter</artifactId>
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
  serão automaticamente serializadas/deserializadas usando o gson */

  @Path("/customers") @Post
  @JsonContent
  Customer createCustomer(@BodyParameter Customer customer);

  @Path("/customers/{id}") @Get
  Customer findCustomer(@PathParameter String id);
}
```

A implementação da classe `GsonMessageConverter` utiliza uma instância do objeto [Gson](https://google.github.io/gson/apidocs/com/google/gson/Gson.html) com configurações padrão. Se você precisar de outras modificações, seu código pode gerar o objeto `Gson` (usando o [GsonBuilder](https://google.github.io/gson/apidocs/com/google/gson/GsonBuilder.html)) e instanciar o `GsonMessageConverter` manualmente:

```java
import com.github.ljtfreitas.restify.http.client.message.converter.json.GsonMessageConverter;
import com.google.gson.Gson;
import com.google.gson.GsonBuilder;

Gson myGson = new new GsonBuilder(); //configura o Gson da maneira necessária...
  .build()

MyApi myApi = new RestifyProxyBuilder()
  .converters()
    .add(new GsonMessageConverter(myGson))
    .and()
  .target(MyApi.class)
    .build();
```
