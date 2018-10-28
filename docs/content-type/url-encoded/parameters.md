# Serialização e deserialização - Formatos - application/x-www-form-urlencoded - Parameters

Outra opção é utilizar uma instância do tipo `Parameters`, um objeto *Map-like* que permite adicionar múltiplos valores por parâmetro:

```java
public interface MyApi {

  /* em requisições com o content-type application/x-www-form-urlencoded,
  argumentos do tipo Parameters serão automaticamente serializados */

  @Path("/customers") @Post
  @FormURLEncoded
  Customer createCustomer(@BodyParameter Parameters parameters);

}

MyApi myApi = new RestifyProxyBuilder()
    .target(MyApi.class)
        .build();

// Parameters é um objeto imutável
Parameters parameters = new Parameters()
    .put("name", "Tiago de Freitas Lima")
    .put("age", "31")
    .put("socialPreferences", "facebook")
    .put("socialPreferences", "twitter");

Customer customer = myApi.createCustomer(parameters);

```
