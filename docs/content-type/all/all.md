# Serialização e deserialização - Formatos - wildcard

Além dos objetos para formatos de texto, esses são os únicos deserializadores incluídos no artefato padrão do `java-restify`, e permitem apenas **ler** respostas. Qualquer conteúdo é suportado, e a resposta pode ser convertida para `byte[]`, `InputStream` ou `String`.

```java
public interface MyApi {
  
  // respostas de qualquer tipo podem ser deserializadas para os formatos abaixo
  
  @Path("/customers/{id}") @Get
  byte[] findCustomerAsByteArray(@PathParameter String id);

  @Path("/customers/{id}") @Get
  InputStream findCustomerAsInputStream(@PathParameter String id);

  @Path("/customers/{id}") @Get
  String findCustomerAsString(@PathParameter String id);
}
```
