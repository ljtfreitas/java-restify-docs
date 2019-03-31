# Serialização e deserialização - Formatos - application/xml - JAX-B

[JAX-B](https://javaee.github.io/jaxb-v2/) é uma especificação Java, incluída no próprio JDK, para processamento de XML, incluindo `binding` de objetos usando um rico conjunto de anotações.

## Instalação

Para utilizar o JAX-B, inclua a dependência `java-restify-xml-jaxb-converter`. O `converter` `JaxBXmlMessageConverter` será automaticamente registrado.

### Maven

```xml
<dependency>
  <groupId>com.github.ljtfreitas</groupId>
  <artifactId>java-restify-xml-jaxb-converter</artifactId>
  <version>{version}</version>
</dependency>
```

### Gradle

```groovy
dependencies {
  compile("com.github.ljtfreitas:java-restify-xml-jaxb-converter:{version}")
}
```

## Utilização

```java 
public interface MyApi {

  /* requisições e respostas com o content-type application/xml
  serão automaticamente serializadas/deserializadas usando o jax-b */

  @Path("/customers/{id}") @Get
  Customer findCustomer(@PathParameter String id);

  @Path("/customers") @Post
  @XmlContent
  Customer createCustomer(@BodyParameter Customer customer);
}
```
