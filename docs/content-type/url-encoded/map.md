# Serialização e deserialização - Formatos - application/x-www-form-urlencoded - Map

É possível representar um formulário de parâmetros usando um `Map<String, ?>`:

```java
public interface MyApi {

  /* em requisições com o content-type application/x-www-form-urlencoded,
  mapas com a chave do tipo String serão automaticamente serializados */

  @Path("/customers") @Post
  @FormURLEncoded
  Customer createCustomer(@BodyParameter Map<String, Object> parameters);

}
```
