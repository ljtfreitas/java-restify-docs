# Serialização e deserialização - Formatos - application/x-www-form-urlencoded - @Form

Outra opção é criar um objeto que representa a estrutura de um formulário, anotado com `@Form`:

```java
@Form
class MyForm {

  @Field
  String name;

  @Field("customer_age")
  int age;
}

public interface MyApi {

  /* em requisições com o content-type application/x-www-form-urlencoded,
  objetos anotados com @Form serão automaticamente serializados */

  @Path("/customers") @Post
  @FormURLEncoded
  Customer createCustomer(@BodyParameter MyForm parameters);

}
```

Objetos anotados com `@Form` também podem ser utilizados em requisições do tipo `GET`, usando o serializador `FormObjectParameterSerializer`. O objeto será deserializado em uma `query string`.

```java
import com.github.ljtfreitas.restify.http.client.message.converter.form.FormObjectParameterSerializer;

public interface MyApi {

  @Path("/customers") @Get
  Customer findCustomerByParameters(@QueryParameters(serializer = FormObjectParameterSerializer.class) MyForm myForm);

}

public static void main(String[] args) {

  MyApi myApi = new RestifyProxyBuilder()
      .target(GitHub.class)
          .build();

  MyForm myForm = new FormParameters();
  myForm.name = "Tiago de Freitas Lima";
  myForm.age = 31;

  // name=Tiago+de+Freitas+Lima&customer_age=31
  Customer customer = myApi.findCustomerByParameters(myForm);
}
```
