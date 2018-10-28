# Tipos de retorno de método

Como detalhado na [documentação sobre serialização e deserialização](../content-type/overview.md), o `java-restify` tentará deserializar o corpo da resposta HTTP para o **tipo de retorno do método**, mas nem sempre esse será o comportamento desejado. 

Eventualmente, podemos querer encapsular a resposta em algum outro objeto; por exemplo, poderíamos projetar nossa interface da seguinte maneira:

```java
public interface MyApi {

  //utiliza Optional como retorno de método, para representar uma resposta potencialmente vazia
  @Path("/customers/{id}") @Get
  Optional<Customer> findCustomer(@PathParameter String id);
}
```

No exemplo acima, embora o retorno do método seja do tipo `Optional`, a resposta (se houver) deve ser deserializada para um objeto `Customer`. Ou seja, o tipo de retorno do método é diferente do tipo que representa a resposta.

## Como funciona?

A propósito, o exemplo acima (`Optional` encapsulando o objeto de resposta) é perfeitamente possível e válido. O que permite esse tipo de composição é a interface `EndpointCallHandler`, um objeto que permite diferenciar e adaptar o **objeto de resposta da requisição** para o **tipo de retorno do método**.

O `java-restify` registra um conjunto de `handlers` para vários tipos de objetos diferentes. Os `handlers` são utilizados em um formato de `chain of responsability`, permitindo múltiplos níveis de composição e adaptação. 

Por exemplo, outra possibilidade para o método acima seria uma execução assíncrona através de um `Future`:

```java
public interface MyApi {

  //métodos que retornam Future serão executados assincronamente
  @Path("/customers/{id}") @Get
  Future<Optional<Customer>> findCustomer(@PathParameter String id);
}
```

No exemplo acima, o `handler` responsável por métodos que retornam `Future` irá adaptar a resposta do `handler` seguinte, que é o responsável por retornos do tipo `Optional`, que por sua vez utiliza um `handler` que não faz nenhuma adaptação, apenas retornando o objeto de resposta deserializado.

Instâncias de `EndpointCallHandler` são fornecidas por implementações de `EndpointCallHandlerProvider`, uma fábrica com duas especializações: `EndpointCallHandlerAdapter`, que gera `handlers` que adaptam outros `handlers`, e `EndpointCallHandlerFactory`, que fabrica `handlers` que convertem a resposta da requisição para outros tipos.

Cada implementação de `EndpointCallHandler` é responsável por um formato ou assinatura de método em particular, como um tipo de retorno específico ou a presença de alguma anotação. O `java-restify` fornece várias implementações, suportando muitos casos de uso diferentes. Consulte os [tipos de retorno suportados por padrão](default-types.md).

Requisições assíncronas também podem depender do tipo de retorno do método, como o exemplo anterior utilizando um `Future`. Consulte a [documentação detalhada sobre requisições assíncronas](../async/overview.md), e os [tipos de retorno assíncronos suportados por padrão](default-types.md) 

E se não houver algum `handler` específico para o tipo de retorno do método? Nos exemplos acima, utilizamos um objeto `Customer` do nosso próprio modelo para representar a resposta:

```java
public interface MyApi {

  Customer findCustomer(@PathParameter String id);
}
```

O método acima retorna um `Customer` mas não existe um `handler` específico para esse tipo; nesse cenário, será utilizado uma implementação de `EndpointCallHandler` que não realiza nenhuma adaptação, assumindo que o tipo de resposta da requisição é mesmo tipo de retorno do método.

### Utilizando handlers

Os `handlers` fornecidos pelo `java-restify` são registrados automaticamente, caso estejam disponíveis no `classpath`.

É possível registrar novos `handlers` facilmente:

```java
MyApi myApi = new RestifyProxyBuilder()
  .handlers()
    .add(new MyEndpointCallHandlerFactory())
    .and()
  .target(MyApi.class)
    .build();
```

Eventualmente pode ser necessário desligar a descoberta automática:

```java
MyApi myApi = new RestifyProxyBuilder()
  .handlers()
    .discovery()
      .disabled() // desabilita a descoberta automática (habilitada por padrão)
    .and()
  .target(MyApi.class)
    .build();
```

## Outras implementações

O `java-restify` também oferece suporte para objetos específicos de alguns frameworks:

* [Guava](guava.md)
* [Jsoup](jsoup.md)
* [Frameworks reativos](../reactive/overview.md)
* [Vavr](vavr.md)

O [suporte ao Spring](../spring-framework/overview.md) também fornece mais opções para retorno de método.

