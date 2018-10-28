# Artefatos

## Core

* `java-restify`: O principal artefato, já contendo todas as principais dependências e pronto para uso.

* `java-restify-call-handler`: Interfaces para criação de `handlers` de retornos de método (já incluida no `java-restify`).

* `java-restify-contract`: Anotações e principais objetos para definição de contratos (já incluida no `java-restify`).

* `java-restify-http-client`: Principais objetos e interfaces para execução de requisições HTTP (já incluida no `java-restify`).

* `java-restify-http-message`: Principais objetos e interfaces para representação e manipulação de requisições e respostas (já incluida no `java-restify`).

* `java-restify-spi`: Implementação do `Service Loader` dos componentes do `java-restify`, utilizado para auto-descoberta de componentes no `classpath` (já incluida no `java-restify`).

## Retry

* `java-restify-retry`: Implementação de `retry` (já incluida no `java-restify`).

## Clientes HTTP

* `java-restify-http-client-apache-httpclient`: Implementações de cliente HTTP utilizando o Apache HTTP Client e Apache HTTP Async Client.

* `java-restify-http-client-jersey`: Implementação de cliente HTTP utilizando o Jersey.

* `java-restify-http-client-netty`: Implementação de cliente HTTP utilizando o Netty.

* `java-restify-http-client-okhttp`: Implementação de cliente HTTP utilizando o OkHttp.

## Plugins

* `java-restify-hateoas`: Implementação do suporte a HATEOAS.

* `java-restify-circuit-breaker`: Anotações e interfaces do suporte a `circuit breaker`.

### CDI

* `java-restify-cdi`: Plugin para o CDI do `java-restify`.

### Contratos

* `java-restify-jaxrs-contract`: Suporte para o uso das anotações do JAX-RS para definição de contratos de API.

### Converters

#### Wildcard

* `java-restify-wildcard-converter`: Deserializadores para qualquer tipo de conteúdo (já incluida no `java-restify`).

#### Json

* `java-restify-json-jackson-converter`: Suporte para o Jackson.

* `java-restify-json-gson-converter`: Suporte para o Gson.

* `java-restify-json-jsonb-converter`: Suporte para o JSON-B.

* `java-restify-json-jsonp-converter`: Suporte para o JSON-P.

#### XML

* `java-restify-xml-jaxb-converter`: Suporte para o JAX-B.

#### Formulários

* `java-restify-form-encoded-multipart-converter`: Suporte para os formatos `application/x-www-form-urlencoded` e `multipart/form-data`.

#### Texto

* `java-restify-text-converter`: Suporte para os formatos `text/plain` e `text/html`.

#### Tipos serializáveis

* `java-restify-octet-converter`: Suporte para o formato `application/octet-stream`.

### Retornos de método

* `java-restify-guava`: Suporte para o Guava

* `java-restify-jsoup`: Suporte para o Jsoup

* `java-restify-rxjava`: Suporte para o RxJava (1.x)

* `java-restify-rxjava-2`: Suporte para o RxJava (2.x)

* `java-restify-reactor`: Suporte para o Reactor

* `java-restify-vavr`: Suporte para o Vavr

### Frameworks Netflix OSS

* `java-restify-netflix-hystrix`: Implementação do suporte a `circuit-breaker` utilizando o Hystrix.

* `java-restify-netflix-ribbon`: Implementação de cliente HTTP utilizando o Ribbon, com suporte a `service discovery`.

* `java-restify-netflix-service-discovery`: Principais objetos e interfaces para implementações de `service discovery`, para serem utilizadas com o Ribbon.

* `java-restify-netflix-kubernets-service-discovery`: Implementação de `service discovery` usando o Kubernetes, para ser usada com o Ribbon.

* `java-restify-netflix-zookeeper-service-discovery`: Implementação de `service discovery` usando o Zookeeper, para ser usada com o Ribbon.

* `java-restify-reactor-netflix`: Implementações para utilizar os objetos do Reactor em conjunto com o Hystrix.

### Spring Framework

* `java-restify-spring`: Suporte ao uso de anotações do Spring MVC, objetos do Spring como retorno de método e implementação de cliente HTTP utilizando o RestTemplate.

* `java-restify-spring-reactive`: Implementação de cliente HTTP utilizando o WebClient do Spring WebFlux.

* `java-restify-spring-autoconfigure`: Auto-configuração do Spring Boot para o `java-restify` (incluído no `starter`).

* `java-restify-spring-starter`: Starter do Spring Boot para o `java-restify`.

* `java-restify-netflix-spring-autoconfigure`: Auto-configuração do Spring Boot para os componentes do `java-restify` que utilizam os frameworks do Netflix OSS.

### Autenticação OAUTH 2

* `java-restify-oauth2-authentication`: Implementação da autenticação utilizando OAUTH 2.

* `java-restify-oauth2-access-token-cache-caffeine`: Implementação do cache de `access tokens` usando o Caffeine, para ser utilizado com a autenticação OAUTH 2.

* `java-restify-oauth2-access-token-cache-jcache`: Implementação do cache de `access tokens` usando o JCache, para ser utilizado com a autenticação OAUTH 2.

## Utilitários

* `java-restify-reflection`: Classes para manipulação de `reflection`, `scanning` de anotações e verificações de tipos.

* `java-restify-util`: Classes utilitárias para uso interno. Não são classes de propósito geral.

* `java-restify-util-async`: Classes utilitárias de lógica assíncrona, para uso interno. Não são classes de propósito geral.
