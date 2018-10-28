# Instalação

## Maven

```xml
<dependency>
  <groupId>com.github.ljtfreitas</groupId>
  <artifactId>java-restify</artifactId>
  <version>{version}</version>
</dependency>
```

## Gradle

```groovy
dependencies {
  compile("com.github.ljtfreitas:java-restify:{version}")
}
```

**Nenhuma** dependência adicional será incluída no seu `classpath`; um princípio de implementação do `java-restify` é utilizar por padrão apenas as classes disponíveis no JDK.

Algumas dependências adicionais deverão ser incluídas caso você necessite usar funcionalidades específicas. Por exemplo, o suporte para JSON utilizando o [Gson](https://github.com/google/gson) está disponível em outro artefato. Consulte a [documentação sobre artefatos adicionais](../artifacts/list.md) para mais detalhes.
