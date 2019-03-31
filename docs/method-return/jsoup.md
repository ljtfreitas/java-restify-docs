# Retornos de método - Jsoup

[jsoup](https://jsoup.org/) é uma biblioteca Java para *parse* e manipulação de HTML, bastante poderosa para `webcrawling` e `webscraping`. 

## Instalação

O suporte ao jsoup está na dependência `java-restify-jsoup`. Os `handlers` serão registrados automaticamente.

### Maven

```xml
<dependency>
  <groupId>com.github.ljtfreitas</groupId>
  <artifactId>java-restify-jsoup</artifactId>
  <version>{version}</version>
</dependency>
```

### Gradle

```groovy
dependencies {
  compile("com.github.ljtfreitas:java-restify-jsoup:{version}")
}
```

## Utilização

O principal objeto do jsoup é o [Document](https://jsoup.org/apidocs/org/jsoup/nodes/Document.html), que representa um documento HTML.

```java
import org.jsoup.nodes.Document;

@Path("http://www.google.com")
public interface Google {

    @Path("/") @Get
    Document home();
}
```
