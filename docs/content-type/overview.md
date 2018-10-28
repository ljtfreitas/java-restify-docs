# Tipos de conteúdo - Serialização e deserialização

## Definição

No `java-restify`, **serialização** se refere ao mecanismo utilizado para escrever o **corpo da requisição** ; e **deserialização** se refere ao mecanismo utilizado para ler o **corpo da resposta**. 

Ambos são feitos de maneira transparente; a serialização se aplica ao parâmetro do método que representa o **corpo** (anotado com `@RequestBody`, no caso das anotações padrão), e a deserialização se aplica ao **tipo de retorno** do método.

## Formatos

O formato utilizado na serialização e deserialização é determinado pelo cabeçalho **Content-Type**. 

O `java-restify` fornece várias implementações para lidar com tipos específicos de conteúdo, e essas implementações também influenciam argumentos e retornos de método que podem ser utilizados.

Os serializadores, responsáveis por converter um objeto para um determinado formato e escrever no **corpo da requisição**, são implementações da interface `HttpMessageWriter`.

Os deserializadores, responsáveis por converter o **corpo da resposta** (em um formato qualquer) para um objeto, são implementações da interface `HttpMessageReader`.

Essas duas interfaces extendem `HttpMessageConverter`, que fornece o tipo de conteúdo que aquele objeto é capaz de lidar. Caso você precise serializar/deserializar formatos não suportados pelo `java-restify`, basta implementar ``HttpMessageWriter`` (para serialização) ou ``HttpMessageReader`` (para deserialização) ou ambos, e registrar seu `converter` customizado.

```java
import com.github.ljtfreitas.restify.http.client.message.ContentType;
import com.github.ljtfreitas.restify.http.client.message.converter.HttpMessageReadException;
import com.github.ljtfreitas.restify.http.client.message.converter.HttpMessageReader;
import com.github.ljtfreitas.restify.http.client.message.converter.HttpMessageWriteException;
import com.github.ljtfreitas.restify.http.client.message.converter.HttpMessageWriter;
import com.github.ljtfreitas.restify.http.client.message.request.HttpRequestMessage;
import com.github.ljtfreitas.restify.http.client.message.response.HttpResponseMessage;

public class MyCustomConverter implements HttpMessageReader<Object>, HttpMessageWriter<Object> {

  @Override
  public ContentType contentType() {
    return ContentType.of("text/whatever");
  }

  @Override
  public boolean canWrite(Class<?> type) {
    // verifica se essa implementação é capaz de serializar o tipo de objeto
    return false;
  }

  @Override
  public void write(Object body, HttpRequestMessage httpRequestMessage) throws HttpMessageWriteException {
    // escreve o objeto no corpo da requisição
  }

  @Override
  public boolean canRead(Type type) {
    // verifica se essa implementação é capaz de deserializar a resposta para esse tipo de objeto
    return false;
  }

  @Override
  public Object read(HttpResponseMessage httpResponseMessage, Type expectedType) throws HttpMessageReadException {
    // lê a resposta e converte para o objeto esperado
    return null;
  }
}

MyApi myApi = new RestifyProxyBuilder()
  .converters()
    .add(new MyCustomConverter())
    .and()
  .target(MyApi.class)
    .build();
```

Os únicos `converters` registrados por padrão são capazes de deserializar [qualquer tipo de conteúdo](all/all.md) (`Content-Type=*/*`), e podem apenas **ler** respostas.

As demais implementações fornecidas pelo `java-restify` estão em artefatos separados, e serão registradas automaticamente caso estejam disponíveis no `classpath`; basta incluir a dependência adequada para o tipo de contúdo que deseja utilizar. O `java-restify` já fornece objetos capazes de lidar com os formatos [application/json](json/json.md), [application/xml](xml/xml.md), [text/plain](text/text-plain.md), [text/html](text/text-html.md), [application/x-www-form-urlencoded](url-encoded/url-encoded.md), [multipart/form-data](multipart/multipart.md) e [application/octet-stream](octet/octet.md).

```java
// todos os formatos listados acima, se disponíveis no classpath, serão registrados
MyApi myApi = new RestifyProxyBuilder()
  .target(MyApi.class)
    .build();
```

Também é possível implementar um controle fino dos `converters` utilizados, selecionando apenas os que fizerem sentido para o seu caso de uso.

```java
MyApi myApi = new RestifyProxyBuilder()
  .converters()
    .json() //apenas json será automaticamente registrado, se estiver disponível no classpath
    .and()
  .target(MyApi.class)
    .build();

MyApi myApi = new RestifyProxyBuilder()
  .converters()
    .xml() //apenas xml será automaticamente registrado, se estiver disponível no classpath
    .and()
  .target(MyApi.class)
    .build();
```

Eventualmente também pode ser necessário desligar o registro automático dos `converters` disponíveis no `classpath`.

```java
MyApi myApi = new RestifyProxyBuilder()
  .converters()
    .discovery()
        .disabled() // desabilita a descoberta automática de converters
    .json()
    .and()
  .target(MyApi.class)
    .build();
```

Caso você registre algum `converter` customizado, as configurações padrão do `java-restify` serão sobrescritas e apenas os `converters` registrados manualmente serão utilizados.

```java
MyApi myApi = new RestifyProxyBuilder()
  .converters()
    .add(new MyCustomConverter()) //apenas esse converter será utilizado
    .and()
  .target(MyApi.class)
    .build();

// demais converters, se forem necessários, também devem ser incluídos explicitamente
MyApi myApi = new RestifyProxyBuilder()
  .converters()
    .add(new MyCustomConverter())
    .json() // além do converter acima, json também será suportado, se estiver disponível no classpath
    .and()
  .target(MyApi.class)
    .build();
```

### Formatos suportados

O `java-restify` fornece implementações para alguns tipos de conteúdo:

* [wildcard (qualquer tipo de conteúdo)](all/all.md)
* [application/json](json/json.md)
* [application/xml](xml/xml.md)
* [application/x-www-form-urlencoded](url-encoded/url-encoded.md)
* [multipart/form-data](multipart/multipart.md)
* [text/plain](text/text-plain.md)
* [text/html](text/text-html.md)
* [application/octet-stream](octet/octet.md)