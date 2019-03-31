# Serialização e deserialização - Formatos - application/octet-stream

O *Content-Type* **application/octet-stream** representa um conteúdo "desconhecido", ou um conteúdo para o qual não exista um *mime-type* específico. 

No `java-restify`, em requisições com esse tipo de conteúdo, o argumento será **escrito usando a serialização padrão do Java**. A leitura de respostas é implementada de maneira equivalente: o corpo da resposta HTTP será convertido para o retorno do método usando a **deserialização padrão do Java**.

Para utilizar esse *Content-Type*, basta adicionar a dependência `java-restify-octet-converter`.

```java 
  public interface MyApi {

    /* em requisições com o content-type application/octet-stream, argumentos do tipo byte[] serão automaticamente serializados */
    @Path("/resource") @Post
    @SerializableContent
    String createResourceAsByteArray(@BodyParameter byte[] bytes);

    /* respostas com o content-type application/octet-stream também podem ser deserializadas para byte[] */
    @Path("/resource") @Get
    byte[] resourceAsByteArray();

    /* em requisições com o content-type application/octet-stream, argumentos do tipo InputStream serão automaticamente serializados */
    @Path("/resource") @Post
    @SerializableContent
    String createResourceAsInputStream(@BodyParameter InputStream inputStream);

    /* respostas com o content-type application/octet-stream também podem ser deserializadas para InputStream */
    @Path("/resource") @Get
    InputStream resourceAsInputStream();

    /* em requisições com o content-type application/octet-stream, argumentos do tipo Serializable serão automaticamente serializados */
    @Path("/resource") @Post
    @SerializableContent
    String createResourceAsSerializable(@BodyParameter Resource customer);

    /* respostas com o content-type application/octet-stream também podem ser deserializadas para objetos do tipo Serializable */
    @Path("/resource") @Get
    Resource resourceAsSerializable();
  }

  class Resource implements Serializable { //serialização padrão do Java
  }
```
