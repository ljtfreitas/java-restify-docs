# Serialização e deserialização - Formatos - text/plain

O *Content-Type* **text/plain** representa um formato de texto simples.

Adicione a dependência `java-restify-text-converter`, e automaticamente serão registrados componentes para manipular esse tipo de conteúdo.

```java
  public interface MyApi {

    /* em requisições com o content-type text/plain, argumento de tipos primitivos ou wrapper serão automaticamente serializados */
    @Path("/resource") @Post
    @Header(name = "Content-Type", value = "text/plain")
    String createResourceAsInt(@BodyParameter int value);

    /* em respostas com o content-type text/plain, retornos de método de tipos primitivos ou wrapper serão automaticamente deserializados também */
    @Path("/resource") @Get
    Integer resourceAsInt();

    /* o corpo da requisição também pode ser do tipo String */
    @Path("/resource") @Post
    @Header(name = "Content-Type", value = "text/plain")
    String createResourceAsString(@BodyParameter String content);

    /* ou o retorno do método, para obter uma resposta do tipo text/plain */
    @Path("/resource") @Post
    String resourceAsString();
}
```