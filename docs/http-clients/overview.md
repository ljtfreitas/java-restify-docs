# Clientes HTTP

## Visão geral

Os objetos que realizam uma requisição HTTP são representados por duas interfaces: `EndpointRequestExecutor`, que recebe uma requisição construída a partir dos anotações do método e retorna uma resposta completa (com `status code`, cabeçalhos e corpo); e `HttpClientRequestFactory`, uma fábrica cuja implementação constrói efetivamente a requisição HTTP propriamente dita, utilizando algum framework, bibiloteca ou conjunto de objetos especializados.

A implementação padrão do `EndpointRequestExecutor` é responsável pelo processo de serialização/deserialização, e delega a construção da requisição para alguma implementação de `HttpClientRequestFactory`. Essa composição é bastante flexível pois permite variar entre as várias bibliotecas HTTP suportadas (que são implementações de `HttpClientRequestFactory`).

## Client HTTP padrão

Por padrão, o `java-restify` utiliza apenas recursos disponíveis na API padrão do Java, e isso também é válido para o *client* HTTP. A implementação padrão do `HttpClientRequestFactory` utiliza o [HttpUrlConnection](https://docs.oracle.com/javase/8/docs/api/java/net/HttpURLConnection.html). 

Esse objeto não possui nenhum suporte especial para requisições assíncronas, portanto, conforme detalhado na [documentação sobre requisições assíncronas](../async/overview.md), a requisição apenas será realizada em outra `thread`.

O `RestifyProxyBuilder` possui algumas opções de configurações que se aplicam somente à implementação padrão:

```java
MyApi myApi = new RestifyProxyBuilder()
    .client()
        .charset() //charset padrão para requisições e respostas (padrão é UTF-8)
        .connectionTimeout(Duration.ofMillis(1000)) // timeout da conexão (padrão é 0 = infinito)
        .readTimeout(Duration.ofMillis(1000)) // timeout de leitura da resposta (padrão é 0 = infinito)
        .followRedirects() // configuração de follow redirects
            .disabled() // padrão é enabled
        .useCaches() // configuração de uso de caches
            .disabled() // padrão é enabled
        .proxy(Proxy.NO_PROXY) // configuração de proxy (padrão é nenhum)
        .ssl() // configurações de ssl (utilizadas apenas se forem configuradas)
            .hostnameVerifier(...)
            .sslSocketFactory(...)
        .and()
    .target(MyApi.class)
        .build();
```

### Buffer request body e streaming mode

Duas configurações que merecem uma explicação em particular são:

```java
MyApi myApi = new RestifyProxyBuilder()
    .client()
        .bufferRequestBody()
            .enabled()
        .outputStreaming()
            .enabled()
            .chunkSize(1024)
            .and()
        .and()
    .target(MyApi.class)
        .build();
```

Por padrão as duas configurações acima (`bufferRequestBody` e `outputStreaming`) são habilitadas. Quando `bufferRequestBody` é habilitado, o conteúdo do corpo da requisição é **bufferizado em memória** (antes de ser enviado); e se `outputStreaming` também estiver habilitado (sim, por padrão) o tamanho do corpo é explicitamente definido usando o método [setFixedLengthStreamingMode](https://docs.oracle.com/javase/8/docs/api/java/net/HttpURLConnection.html#setFixedLengthStreamingMode-long-). Isso permite que o `HttpUrlConnection` escreva na conexão HTTP sem bufferizar novamente o conteúdo. 

Essa estratégia é mais simples e eficiente para requisições com conteúdo relativamente menores, mas pode ser problemática para envio de grandes arquivos, ou `payloads` de tamanho maior. Nesse cenário, escrever o conteúdo em `chunks` (ou definindo o tamanho do conteúdo, caso seja conhecido) pode ser mais eficiente.

Se `bufferRequestBody` for desabilitado, nenhum `buffer` intermediário do conteúdo será realizado e os dados serão escritos diretamente na conexão HTTP. Para melhorar o desempenho, o ideal é que outros parâmetros relativos ao tamanho do conteúdo estejam adequadamente configurados. Assumindo que `bufferRequestBody` está desabilitado e `outputStreaming` está habilitado:

- se o cabeçalho *Content-Length* estiver preenchido, assume-se que o tamanho do conteúdo é **pré-conhecido** e é definido usando o método [setFixedLengthStreamingMode](https://docs.oracle.com/javase/8/docs/api/java/net/HttpURLConnection.html#setFixedLengthStreamingMode-long-)

- se o cabeçalho *Content-Length* não estiver preenchido, assume-se que o tamanho do conteúdo não é conhecido, e o envio de dados será feito usando [chunk transfer encoding](https://en.wikipedia.org/wiki/Chunked_transfer_encoding). O tamanho do `chunk` será o tamanho em *bytes* definido na configuração `outputStreaming().chunkSize()` (o tamanho padrão é 4kb).
