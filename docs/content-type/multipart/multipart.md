# Serialização e deserialização - Formatos - multipart/form-data

O *Content-Type* **multipart/form-data** representa um formulário de parâmetros, em que os campos são enviados em diferentes "partes" isoladas. Esse formato permite o envio de arquivos.

O `java-restify` fornece várias opções para essa funcionalidade: instâncias do tipo `Map`, objetos do tipo `MultipartFile` (para envio de arquivos somente), objetos do tipo `MultipartParameters` e objetos anotados com `@MultipartForm`. 

* [Map](map.md)
* [MultipartFile](multipart-file.md)
* [MultipartParameters](multipart-parameters.md)
* [@MultipartForm](form.md)

## Instalação

O suporte para todas as opções acima estão na dependência `java-restify-form-encoded-multipart-converter`. Uma vez presente no `classpath`, os `converters` adequados serão automaticamente registrados.

### Maven

```xml
<dependency>
  <groupId>com.github.ljtfreitas</groupId>
  <artifactId>java-restify-form-encoded-multipart-converter</artifactId>
  <version>{version}</version>
</dependency>
```

### Gradle

```groovy
dependencies {
  compile("com.github.ljtfreitas:java-restify-form-encoded-multipart-converter:{version}")
}
```
