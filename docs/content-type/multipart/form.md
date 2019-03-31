# Serialização e deserialização - Formatos - multipart/form-data - @MultipartForm

Para representar um formulário "multipart", utilize a anotação `@MultipartForm`. É equivalente à anotação [@Form](../url-encoded/form.md), mas suporta o uso da anotação `@MultipartField`, que pode ser utilizada para explicitar campos que representam arquivos (campos do tipo `File`, `Path` ou `InputStream`).

```java
@MultipartForm
class FormParameters {

  @Field
  String pictureName;

  // campos anotados com @MultipartField também podem ser do tipo Path ou InputStream
  @MultipartField
  File picture;
}

public interface MyApi {

  /* em requisições com o content-type multipart/form-data, objetos anotados com @MultipartForm serão automaticamente serializados */

  @Path("/customers/{id}/picture") @Post
  @MultipartFormData
  String uploadPictureToCustomer(@PathParameter id, @BodyParameter FormParameters parameters);
}
```