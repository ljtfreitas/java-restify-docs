# Serialização e deserialização - Formatos - multipart/form-data - MultipartParameters

`MultipartParameters` é um objeto equivalente ao [Parameters](../url-encoded/parameters.md), mas com algumas facilidades para envio de arquivos.

```java
public interface MyApi {

    /* em requisições com o content-type multipart/form-data, argumentos do tipo MultipartParameters serão automaticamente serializados. */

    @Path("/customers/{id}/picture") @Post
    @MultipartFormData
    String uploadPictureToCustomer(@PathParameter id, @BodyParameter MultipartParameters parameters);
}

MyApi myApi = new RestifyProxyBuilder()
    .target(MyApi.class)
        .build();

//MultipartParameters é imutável
MultipartParameters parameters = new MultipartParameters()
    .put("picture_name", "main-picture")
    .put("picture", new File("/path/to/file/picture.jpg"));

Customer customer = myApi.uploadPictureToCustomer("1", parameters);

```