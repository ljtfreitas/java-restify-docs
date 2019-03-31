# Anotações - Extensões - JAX-RS

**JAX-RS** é uma específicação Java para criação de serviços REST, e oferece um rico conjunto de anotações. O `java-restify` fornece suporte para utilizar as mesmas anotações no *client-side*.

## Instalação

Para utilizar as anotações do JAX-RS, adicione a dependência `java-restify-jaxrs-contract`, que fornece a implementação `JaxRsContractReader`.

### Maven

```xml
<dependency>
  <groupId>com.github.ljtfreitas</groupId>
  <artifactId>java-restify-jaxrs-contract</artifactId>
  <version>{version}</version>
</dependency>
```

### Gradle

```groovy
dependencies {
  compile("com.github.ljtfreitas:java-restify-jaxrs-contract:{version}")
}
```

## Utilização

```java
import javax.ws.rs.ApplicationPath;
import javax.ws.rs.Consumes;
import javax.ws.rs.DELETE;
import javax.ws.rs.GET;
import javax.ws.rs.HeaderParam;
import javax.ws.rs.CookieParam;
import javax.ws.rs.POST;
import javax.ws.rs.PUT;
import javax.ws.rs.Path;
import javax.ws.rs.PathParam;
import javax.ws.rs.Produces;
import javax.ws.rs.QueryParam;

@ApplicationPath("http://myapi.com") //@Path também é suportado no topo da interface
interface MyApi {

    @Path("/resource") //@Path para métodos (@ApplicationPath é suportado apenas no topo da interface)
    @GET
    String getResource();

    @Path("/resource/{id}")
    @GET
    String getResourceById(@PathParam("id") String id);

    @Path("/resource")
    @GET
    String getResourceWithHeader(@HeaderParam("X-Custom-Header") String id);

    @Path("/resource")
    @GET
    String getResourceWithCookie(@CookieParam("whatever") String content);

    @Path("/resource")
    @GET
    String getResourceByName(@QueryParam("name") String name);

    @Path("/resource/{id}")
    @DELETE
    void deleteResourceById(@PathParam("id") String id);

    @Path("/resource/{id}")
    @PUT
    @Consumes("application/json") //adiciona o cabeçalho Content-Type:application/json
    void updateResourceById(@PathParam("id") String id, Resource body); //o argumento sem anotação será considerado como request body

    @Path("/resource")
    @POST
    @Consumes("application/json") //adiciona o cabeçalho Content-Type:application/json
    void createResource(Resource body); //o argumento sem anotação será considerado como request body

    @Path("/resource/{id}")
    @GET
    @Produces("application/json") //adiciona o cabeçalho Accept:application/json
    String getResourceByIdAsJson(@PathParam("id") String id);
}

MyApi myApi = new RestifyProxyBuilder()
  .contract(new JaxRsContractReader())
  .target(MyApi.class)
    .build();

```

As anotações [MatrixParam](https://javaee.github.io/javaee-spec/javadocs/javax/ws/rs/MatrixParam.html) e [BeanParam](https://javaee.github.io/javaee-spec/javadocs/javax/ws/rs/BeanParam.html) não são suportadas.
