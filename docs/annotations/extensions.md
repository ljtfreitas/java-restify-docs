# Anotações - Extensões

O processamento de anotações é realizado por implementações da interface `ContractReader`. A implementação padrão utiliza as anotações fornecidas pelo `java-restify`, mas existem extensões para outros conjuntos de anotações.

Caso deseje implementar algum suporte customizado para qualquer conjunto de anotações, basta criar sua própria implementação de `ContractReader` e utilizá-la na construção do `proxy`:

```java
import com.github.ljtfreitas.restify.http.contract.metadata.ContractReader;
import com.github.ljtfreitas.restify.http.contract.metadata.EndpointMethods;
import com.github.ljtfreitas.restify.http.contract.metadata.EndpointTarget;

class MyContractReader implements ContractReader {

    @Override
    public EndpointMethods read(EndpointTarget target) {
        /* 
        EndpointTarget fornece o tipo da interface e a url base utilizada na construção do proxy.
        
        EndpointMethods representa a coleção de métodos que serão utilizados para requisições.
        */
    }
}

MyApi myApi = new RestifyProxyBuilder()
  .contract(new MyContractReader())
  .target(MyApi.class)
    .build();
```
