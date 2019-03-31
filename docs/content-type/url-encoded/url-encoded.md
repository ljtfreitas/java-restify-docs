# Serialização e deserialização - Formatos - application/x-www-form-urlencoded

O *Content-Type* **application/x-www-form-urlencoded** representa um "formulário" de parâmetros, e é basicamente uma sequência de campos no formato `nome=valor` separados pelo caracter `&`.

O `java-restfy` permite representar essa estrutura de diferentes maneiras: instâncias do tipo `Map`, objetos do tipo `Parameters` e objetos anotados com `@Form`.

* [Map](map.md)
* [Parameters](parameters.md)
* [@Form](form.md)

## Instalação

O suporte para todas as opções acima estão no artefato `java-restify-form-encoded-multipart-converter`. Uma vez presente no `classpath`, os `converters` adequados serão automaticamente registrados.

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
