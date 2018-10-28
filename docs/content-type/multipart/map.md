# Serialização e deserialização - Formatos - multipart/form-data - Map

Uma opção é utilizar um `Map<String, ?>`. Se o valor for do tipo `File`, `Path` ou `InputStream`, ele será serializado como um `MultipartFile`.

A chave do mapa **deve** ser do tipo `String`.

```java
public interface MyApi {

  /* em requisições com o content-type multipart/form-data, mapas com a chave do tipo String serão automaticamente serializados. */

  @Path("/customers/{id}/picture") @Post
  @MultipartFormData
  String uploadPictureToCustomer(@PathParameter id, @BodyParameter Map<String, Object> parameters);
}

MyApi myApi = new RestifyProxyBuilder()
    .target(MyApi.class)
        .build();

Map<String, Object> parameters = new HashMap<>();
parameters.put("picture_name", "main-image");
parameters.put("picture", new File("/path/to/file/picture.jpg"));

Customer customer = myApi.uploadPictureToCustomer("1", parameters);
  
```