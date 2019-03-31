# Serialização e deserialização - Formatos - text/html

O *Content-Type* **text/html** representa um conteúdo HTML.

Adicione a dependência `java-restify-text-converter`, e automaticamente será registrado um componente para lidar com esse tipo conteúdo. O tipo `String` é o único objeto suportado para serialização e deserialização.

```java  
public interface MyApi {

    /* em requisições com o content-type text/html, argumento de tipos String serão automaticamente serializados */
    @Path("/resource") @Post
    @Header(name = "Content-Type", value = "text/html")
    String createResourceAsHtml(@BodyParameter String html);

    /* em respostas com o content-type text/html, retornos de método de tipos String serão deserializados também */
    @Path("/resource") @Get
    String resourceAsHtml();
}
```