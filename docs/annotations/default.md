# Anotações

Para consumir uma API com o `java-restify`, precisamos especificar os detalhes dos **contratos**: qual o `path`, o verbo HTTP utilizado, o formato do corpo da requisição e da resposta, cabeçalhos, etc. Isso é feito com o uso de **anotações**.

## Definição do endpoint

### @Path

A anotação `@Path` é utilizada para construção do `path` do `endpoint`. É obrigatória em todos os métodos. Se estiver presente no topo da classe, o conteúdo será concatenado à anotação do método.

```java
@Path("/base/api") //aplicado a todos os métodos
public interface MyApi {

  @Path("/resource")
  String getResource(); //o endpoint será "/base/api/resource"
}

public interface MyApi {

  @Path("/resource")
  String getResource(); //o endpoint será "/resource"
}
```

### @PathParameter

É possível construir o `path` dinâmicamente, baseado nos parâmetros do método.

```java  
public interface MyApi {

  @Path("/resource/{id}/{name}")
  String getResource(String id, String name);
}
```

No exemplo acima, o `path` tem duas partes variáveis, chamadas **id** e **name**, e o `endpoint` será construído no momento da invocação do método, usando o valor fornecido para cada argumento. O `binding` será realizado utilizando os **nomes** dos argumentos do método.

> Para permitir que os nomes dos parâmetros dos métodos sejam obtidos através de *reflection*, seu código deve ser compilado com a flag **-parameters**

Caso prefira não utilizar o `binding` pelo nome dos argumentos, ou quiser associar a varíavel a um nome diferente do nome do parâmetro, é possível utilizar a anotação `@PathParameter`:

```java
public interface MyApi {

  @Path("/resource/{id}/{name}")
  String getResource(@PathParameter("id") String identity, @PathParameter("name") String resourceName);
}
```

É permitido incluir essa anotação nos argumentos do método mesmo sem customizar o nome, tornando explícito que esses parâmetros fazem parte do `path`:

```java
public interface MyApi {

  @Path("/resource/{id}/{name}")
  String getResource(@PathParameter String id, @PathParameter String name);
}
```

> Se nenhuma anotação for adicionada ao parâmetro, ele será considerado um `@PathParameter`.

## Métodos HTTP

Também é obrigatório informar qual método HTTP deve ser utilizado. As anotações existentes são:

```java
public interface MyApi {

  @Get
  @Path("/resource") 
  String get();

  @Post
  @Path("/resource") 
  String post();

  @Put
  @Path("/resource") 
  String put();

  @Delete
  @Path("/resource") 
  String delete();

  @Patch
  @Path("/resource") 
  String patch();

  @Head
  @Path("/resource") 
  String head();

  @Options
  @Path("/options") 
  String options();

  @Trace
  @Path("/options") 
  String trace();
}
```    

Todas as anotações são acima são **meta-anotações**; elas apenas encapsulam a anotação `@Method`. Caso deseje utilizar algum outro método HTTP qualquer, também é possível utilizar essa anotação diretamente.

```java
public interface MyApi {

  @Method("GET")
  @Path("/resource") 
  String get();
}
```

## Cabeçalhos

### @Header

A anotação `@Header` pode ser utilizada para definição de `headers` da requisição.

```java
@Header(name = "X-Custom-Header", value = "custom header") //aplicado a todos os métodos
public interface MyApi {

  @Path("/customers/{id}") @Get
  @Header(name = "Accept", value = "application/json")
  Customer findCustomerById(String id);

}
```

As anotações `@Header` do topo da interface e do método são unificadas no momento da construção da requisição. No exemplo acima, ao invocar o método `findCustomerBy`, a requisição HTTP terá os cabeçalhos `X-Custom-Header`(que será enviado em todos os métodos da interface) e `Accept`.

A anotação `@Header` é **repetível**, e pode ser utilizada para informar vários cabeçalhos (no topo da interface ou por método):

```java
@Header(name = "X-Custom-Header", value = "custom header")
@Header(name = "X-Other-Custom-Header", value = "custom header")
public interface MyApi {

  @Path("/customers/{id}") @Get
  @Header(name = "Accept", value = "application/json")
  @Header(name = "X-Custom-Customer-Header", value = "specific custom method header")
  Customer findCustomerById(String id);
  
}
```

### @HeaderParameter

Para cabeçalhos dinâmicos, existe a anotação `@HeaderParameter`:

```java
public interface MyApi {
  
  @Path("/customers/{id}") @Get
  Customer findCustomerById(String id, @HeaderParameter("X-Custom-Customer-Header") String customHeader);

}
```

### Shortcuts

Outras anotações úteis para inclusão de cabeçalhos são:

```java
public interface MyApi {

  @Path("/customers/{id}") @Get
  @AcceptAll
  Customer findCustomerById(String id);

  /* @AcceptJson adiciona o cabeçalho "Accept=application/json" */
  @Path("/customers/{id}") @Get
  @AcceptJson
  Customer findCustomerByIdAsJson(String id);

  /* @AcceptXml adiciona o cabeçalho "Accept=application/xml" */
  @Path("/customers/{id}") @Get
  @AcceptXml
  Customer findCustomerByIdAsXml(String id);

  /* @FormURLEncoded adiciona o cabeçalho "Content-Type=application/x-www-form-urlencoded" */
  @Path("/customers") @Post
  @FormURLEncoded
  Customer createCustomerAsForm();

  /* @MultipartFormData adiciona o cabeçalho "Content-Type=multipart/form-data" */
  @Path("/customers") @Post
  @FormURLEncoded
  Customer createCustomerAsMulipart();

  /* @JsonContent adiciona o cabeçalho "Content-Type=application/json" */
  @Path("/customers") @Post
  @JsonContent
  Customer createCustomerAsJson();

  /* @XmlContent adiciona o cabeçalho "Content-Type=application/xml" */
  @Path("/customers") @Post
  @XmlContent
  Customer createCustomerAsXml();

  /* @SerializableContent adiciona o cabeçalho "Content-Type=application/octet-stream" */
  @Path("/customers") @Post
  @SerializableContent
  Customer createCustomerAsSerializable();
}
```

### Cookies

#### @Cookie

A anotação `@Cookie` é utilizada para definição de `cookies` da requisição, que são enviados através do cabeçalho **Cookie**. 

De maneira análoga à anotação `@Header`, `cookies` podem ser definidos no topo da interface ou ao nível do método; `cookies` dinâmicos também podem ser enviados através de argumentos do método anotados com `@CookieParameter`.

```java
@Cookie(name = "my-cookie", value = "cookie-value") //aplicado a todos os métodos
public interface MyApi {

  @Path("/customers/{id}") @Get
  @Cookie(name = "other-cookie", value = "other-cookie-value")
  Customer findCustomerById(String id);

  @Path("/customers/{id}") @Get
  Customer findCustomerById(String id, @CookieParameter("other-cookie") String cookie);
}
```

## Query parameters

### @QueryParameter

Para o envio de `query parameters`, utilize a anotação `@QueryParameter`:

```java
public interface MyApi {

  // "/customers?name=..."
  @Path("/customers") @Get
  Customer findCustomerByName(@QueryParameter String name);

  // "/customers?customer_name=..."
  @Path("/customers") @Get
  Customer findCustomerByName(@QueryParameter("customer_name") String name);

}
```

### @QueryParameters

Para enviar múltiplos parâmetros, naturalmente, pode-se criar um método com vários argumentos anotados com `@QueryParameter`, mas existem outras alternativas, utilizando a anotação `@QueryParameters`.

É possível utilizar um único parâmetro do tipo ``Map<String, ?>``:

```java  
public interface MyApi {

  @Path("/customers") @Get
  Customer findCustomerByParameters(@QueryParameters Map<String, String> mapParameters);
}

public static void main(String[] args) {

  MyApi myApi = new RestifyProxyBuilder()
    .target(MyApi.class)
        .build();

  // adiciona dois query parameters
  Map<String, String> mapParameters = new LinkedHashMap<>;
  mapParameters.put("name", "Tiago de Freitas Lima");
  mapParameters.put("age", "33");

  // /customers?name=Tiago+de+Freitas+Lima&age=33
  Customer customer = myApi.findCustomerByParameters(mapParameters);
}
```

A chave do mapa **deve** ser do tipo `String`, e os valores podem ser de qualquer tipo.

Outra opção é utilizar um parâmetro do tipo `Parameters`, um objeto *Map-like* que permite adicionar múltiplos valores por parâmetro:

```java  
public interface MyApi {

  @Path("/customers") @Get
  Customer findCustomerByParameters(@QueryParameters Parameters parameters);

}

public static void main(String[] args) {

  MyApi myApi = new RestifyProxyBuilder()
    .target(MyApi.class)
        .build();

  // Parameters é um objeto imutável
  Parameters parameters = new Parameters()
    .put("name", "Tiago de Freitas Lima")
    .put("age", "31")
    .put("socialPreferences", "facebook")
    .put("socialPreferences", "twitter");

  // /customers?name=Tiago+de+Freitas+Lima&age=31&socialPreferences=facebook&socialPreferences=twitter
  Customer customer = myApi.findCustomerByParameters(parameters);
}
```

## Request body

### @BodyParameter

Para enviar um objeto no **corpo** da requisição, utilize a anotação `@BodyParameter`:

```java
public interface MyApi {

  @Path("/customers") @Post
  @JsonContent
  Customer createCustomer(@BodyParameter Customer customer);

}
```

Para que o objeto seja adequadamente serializado, o cabeçalho *Content-Type* **deve**, obrigatoriamente, estar definido. No exemplo acima, usando a anotação `@JsonContent`, o *Content-Type* da requisição será `application/json`, e o objeto será serializado nesse formato se houver um **converter** registrado (consulte a [documentação detalhada do mecanismo de serialização/deserialização](../content-type/default.md) e dos tipos de conteúdo suportados).

Apenas **um** argumento do método deve ser anotado com `@BodyParameter`.

## Versionamento

### @Version

A anotação `@Version` pode ser utilizada para indicar explicitamente a versão do `endpoint` que está sendo consumido. A versão será incluída sempre **antes** do `path` especificado para o método.

```java
public interface MyApi {

  // /v1/customers
  @Version("v1")
  @Path("/customers") @Post
  Customer createCustomer(@BodyParameter Customer customer);

}

@Path("http://my.api.com")
public interface MyApi {

  // http://my.api.com/v1/customers
  @Version("v1")
  @Path("/customers") @Post
  Customer createCustomer(@BodyParameter Customer customer);

}
```

A anotação também pode ser utilizada no topo da interface, sendo aplicada para todos os métodos:

```java
@Version("v1")
public interface MyApi {

  // /v1/customers
  @Path("/customers") @Post
  Customer createCustomer(@BodyParameter Customer customer);

}
```

Eventualmente, a estratégia de versionamento da API não será explícita no `path`, e sim através de algum cabeçalho da requisição. Nesse caso, é possível utiizar a anotação apenas para informar explicitamente a versão, sem incluí-la no `path`:

```java
public interface MyApi {

  // o path construído será "/customers"
  @Version(value = "v1", uri = false)
  @Path("/customers") @Post
  Customer createCustomer(@BodyParameter Customer customer);

}
```

A informação da versão ainda estará disponível, e poderá ser acessada em um **interceptor** para inclusão de algum cabeçalho customizado, ou qualquer outra maneira de enviar essa informação na requisição (consulte a [documentação detalhada da API de interceptors](../interceptors/default.md)).

## Serialização de argumentos

Para obter o valor do argumento que deve ser concatenado ao `path`, ao `header` ou ao `query string`, é utilizado o método `toString()` (argumentos com valores nulos são desconsiderados). Mas esse comportamento pode ser customizado através do atributo `serializer`, presente nas anotações `@PathParameter`, `@HeaderParameter`, `@CookieParameter`, `@QueryParameter` e `@QueryParameters`. Esse atributo recebe uma referência para uma implementação da interface `ParameterSerializer`.

Por exemplo, suponhamos que o argumento do método é do tipo `Date`, mas queremos concatenar ao `path` o **timestamp**. A estratégia padrão, usando o método `toString()`, não atenderia esse caso de uso.

```java
public interface MyApi {

  @Path("/resource/{timestamp}")
  String getResourceByTimestamp(@PathParameter(serializer = TimestampParameterSerializer.class) Date timestamp);

}

public class TimestampParameterSerializer implements ParameterSerializer {

  @Override
  public String serialize(String name, Type type, Object source) {
    Date date = (Date) source;
    
    return Long.toString(date.getTime());
  }
}
```