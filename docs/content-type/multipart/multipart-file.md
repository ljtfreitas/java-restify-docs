# Serialização e deserialização - Formatos - multipart/form-data - MultipartFile

`MultipartFile` é um objeto que representa um arquivo que será serializado no corpo da requisição, como um campo de um formulário.

```java
public interface MyApi {

    /* em requisições com o content-type multipart/form-data, argumentos do tipo MultipartFile serão automaticamente serializados */

    @Path("/customers/{id}/picture") @Post
    @MultipartFormData
    String uploadPictureToCustomer(@PathParameter id, @BodyParameter MultipartFile file);

}

MyApi myApi = new RestifyProxyBuilder()
    .target(MyApi.class)
        .build();

// "picture" será o nome do campo enviado na requisição
MultipartFile picture = MultipartFile.create("picture", new File("/path/to/file/picture.jpg"));

Customer customer = myApi.uploadPictureToCustomer("1", picture);

```