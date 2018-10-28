# Quickstart

O funcionamento do `java-restify` é baseado em **proxies de interfaces**.

Primeiro, precisamos de uma interface Java para abstrair a API que desejamos consumir. Com o uso de algumas anotações, podemos representar os detalhes das requisições HTTP.

```java
import com.github.ljtfreitas.restify.http.contract.Path;
import com.github.ljtfreitas.restify.http.contract.Get;
import com.github.ljtfreitas.restify.http.contract.Post;
import com.github.ljtfreitas.restify.http.contract.BodyParameter;
import com.github.ljtfreitas.restify.http.contract.PathParameter;

@Path("http://whatever.api.com")
public interface WhateverApi {
  
  @Path("/resource") @Get
  String getResource();

  @Path("/resource/{id}") @Get
  String getResourceById(@PathParameter("id") String id);

  @Path("/resource") @Post
  String createResource(@BodyParameter String content);
}
```

Os métodos da interface acima irão se comportar da seguinte maneira:

* o método *getResource* irá realizar uma chamada HTTP ``GET`` para a URL **http://whatever.api.com/resource**

* o método *getResourceById* irá realizar uma chamada HTTP ``GET`` para a URL **http://whatever.api.com/resource/{id}**, sendo que o *placeholder* "{id}" será substituído pelo valor do parâmetro anotado com ``@PathParameter``.

* o método *createResource* irá realizar uma chamada HTTP ``POST`` para a URL **http://whatever.api.com/resource**, e o valor do parâmetro anotado com ``@BodyParameter`` será enviado no corpo da requisição.

Agora, precisamos criar uma instância dessa interface. Para criá-la, usamos o objeto `RestifyProxyBuilder`.

```java
import com.github.ljtfreitas.restify.http.RestifyProxyBuilder;

WhateverApi whateverApi = new RestifyProxyBuilder()
  .target(WhateverApi.class)
    .build();
```

Também é possível definir a URL base da API no `builder`, ao invés da anotação ``@Path`` no topo da interface.

```java
import com.github.ljtfreitas.restify.http.RestifyProxyBuilder;

WhateverApi whateverApi = new RestifyProxyBuilder()
  .target(WhateverApi.class, "http://whatever.api.com")
    .build();
```

Por padrão, você pode utilizar apenas ``String``, ``byte[]`` ou ``InputStream`` como retornos de método, para obter a resposta da requisição. Mas existem diversos *plugins* que extendem esse comportamento, permitindo utilizar vários objetos diferentes.

No exemplo abaixo, vamos consumir a API do GitHub (que utiliza JSON), deserializando a resposta para um objeto. Podemos utilizar o [Jackson](https://github.com/FasterXML/jackson) para lidar com o JSON, e para usá-lo com o `java-restify`, precisamos adicionar a dependência `java-restify-json-jackson-converter`. Com essa dependência no seu `classpath`, o `java-restify` irá registrar automaticamente um componente que utiliza o Jackson para serializar/deserializar requisições e respostas no formato JSON.

Usando o Maven:

```xml
<dependency>
  <groupId>com.github.ljtfreitas</groupId>
  <artifactId>java-restify-json-jackson-converter</artifactId>
  <version>{version}</version>
</dependency>
```

Ou o Gradle:

```groovy
dependencies {
  compile("com.github.ljtfreitas:java-restify-json-jackson-converter:{version}")
}
```

Primeiro, criamos os objetos representando a API do Github.

```java
@Path("https://api.github.com")
interface GitHub {

  @Path("/repos/{owner}/{repo}/contributors")
  @Get
  public List<Contributor> contributors(
    @PathParameter("owner") String owner,
    @PathParameter("repo") String repo);
}

@JsonIgnoreProperties(ignoreUnknown = true)
class Contributor {

  @JsonProperty
  private String login;

  @JsonProperty
  private int contributions;

  @Override
  public String toString() {
  return "Contributor: [" + login + "] - " + contributions + " contributions.";
  }
}
```

Agora podemos utilizar o `RestifyProxyBuilder` para obter uma instância da interface.

```java
GitHub gitHub = new RestifyProxyBuilder()
  .target(GitHub.class)
    .build();

/*
  A chamada do método "contributors" vai realizar um GET para https://api.github.com/repos/ljtfreitas/java-restify/contributors. 
  O "bind" dos argumentos do método com o path é realizado utilizando o nome dos parâmetros.
  A resposta da API do GitHub está no formato application/json; o java-restify irá automaticamente deserializar o JSON de resposta para o tipo de retorno do método
*/

gitHub.contributors("ljtfreitas", "java-restify")
  .forEach(System.out::println);

/*
output:

Contributor: [ljtfreitas] - 127 contributions.
*/
```
