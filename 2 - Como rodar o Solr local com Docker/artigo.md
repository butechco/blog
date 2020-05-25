# Como rodar o Solr local com Docker

No [post anterior](https://medium.com/butech-co/um-gole-de-m%C3%A1quinas-de-busca-fa41b7d577c6) discutimos o que é um motor de busca.
Hoje tentaremos utilizar uma das mais famosas plataformas de buscas utilizadas no mercado: o [Solr](https://lucene.apache.org/solr/).
Ele é uma plataforma de busca construída em cima do [Lucene](https://lucene.apache.org/) que é o motor de busca.

O objetivo aqui é explicar como rodar o Solr local, mas com uma pequena introdução de todo o potencial que a plataforma pode proporcionar.

## Lucene e Solr

O Lucene é uma biblioteca Java responsável por fazer a escrita no índice (index) e prover as features de busca (search) para a leitura.
Podem haver transformações nos campos de um documento antes de ser inserido e no processo busca (pipeline).
O Lucece é responsável por implementar os recursos (features) do motor de busca.

Já o Solr é uma cada que normalmente fica a frente do Lucene.
É literalmente um servidor que usa o Lucene por baixo dos panos.
É a camada que facilita a iteração da busca com uma aplicação.

### Pipeline de index e search

No nosso contexto, "pipeline" é como chamamos o fluxo para entrada ou saída de alguma informação.

#### Index

Na escrita o documento pode ter vários tratamentos para cada um de seus campos de maneira personalizada:
- podem ser removidos acentos
- podemos deixar apenas o radical da palavra;
- [tokenizer](https://lucene.apache.org/solr/guide/8_5/about-tokenizers.html) - podemos dizer como as palavras de uma sentença podem ser separadas (por espaço ou outro delimitador)
- etc


#### Search

Já na leitura é aplicado pipeline do termo buscado (query) e a lógica pode ficar muito parecida:
- remover acentos;
- extrair o radical;
- tokenizer
- adicionar sinônimos
- etc

Dá pra perceber que pipeline de escrita pode ser diferente da leitura, pois eles são indepentes.
A [documentação do próprio Solr](https://lucene.apache.org/solr/guide/8_5/field-type-definitions-and-properties.html#field-type-definitions-in-schema-xml) ajuda bastante a entender tudo que é possível ser feito.

Uma coisa importante para entender é que o fica gravado no índice precisa ser equivalente ao termo que será buscado.
Se eu remover os acentos do índice, eu preciso remover os acentos do termo de busca, senão o termo nunca poderá ser encontrado.
Mas algumas coisas podem diferir, como foi o caso dos sinônimos, pois é possível adicioná-los apenas ao termo buscado (no exemplo do pipeline ali de cima).

## E afinal, como rodar?

Por padrão o Solr sobe na porta `8983`, por isso, ao subir o container Docker precisamos deixar essa porta exposta.
Para que o Solr tenha resilência ele utiliza mais de uma instância.
Não vamos entrar em detalhes disso agora, mas saiba que para gerenciar essas instâncias ele precisa do Zookeeper, por isso ele vai subir junto.


Vamos utilizar o [Docker Compose](https://docs.docker.com/compose/) para facilitar essa parte e simplificar um pouco as coisas.

- No terminal você deve baixar o arquivo de descrição da sua infra:
```shell
curl --output docker-compose.yaml -L https://gist.github.com/andreformento/c0657e53ed534946e0a9f39b7103e1f7/raw/solr-example.yaml
```

- Depois, basta subir o Solr com suas dependências:
```shell
docker-compose up -d
```

Agora basta acessar através do browser no endereço: http://localhost:8983/

## Um pouco sobre a sua estrutura

O Solr utiliza o conceito de coleção (collection) para representar a forma como os documentos serão gerenciados - através do seu schema.
Um paralelo ao schema seria a tabela de um banco de dados relacional que possui campos e seus respectivos tipos.

- Para criar a coleção do nosso exemplo (`meus_produtos`) vamos utilizar a sua [API de gerenciamento](https://lucene.apache.org/solr/guide/8_5/collection-management.html#collection-management):
```shell
curl -X POST 'http://localhost:8983/solr/admin/collections?action=CREATE&name=meus_produtos&numShards=1'
```

Ao acessar os arquivos de configuração da sua coleção, você vai ver que lá são definidos os campos e tipos com seus pipelines.
Veja em http://localhost:8983/solr/#/meus_produtos/files?file=managed-schema

O arquivo mostra que podemos ter os tipos, os campos e várias outras configurações relacionadas.
Vejamos, por exemplo o _field\_type_ `text_general`:
```xml
<fieldType name="text_general" class="solr.TextField" positionIncrementGap="100" multiValued="true">
    <analyzer type="index">
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt" />
        <!-- in this example, we will only use synonyms at query time
        <filter class="solr.SynonymGraphFilterFactory" synonyms="index_synonyms.txt" ignoreCase="true" expand="false"/>
        <filter class="solr.FlattenGraphFilterFactory"/>
        -->
        <filter class="solr.LowerCaseFilterFactory"/>
    </analyzer>
    <analyzer type="query">
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt" />
        <filter class="solr.SynonymGraphFilterFactory" synonyms="synonyms.txt" ignoreCase="true" expand="true"/>
        <filter class="solr.LowerCaseFilterFactory"/>
    </analyzer>
</fieldType>
```

Aqui fica claro o que já dissemos referente ao foi dito sobre os pipelines de `index` e `query`, aqui descritos pelos seus respectivos _analyzers_.

### Alterando o schema

Para alterar a collection você pode utilizar a API do Solr:
```shell
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "add-field":{
     "name":"descricao_do_produto",
     "type":"text_pt",
     "stored":true }
}' http://localhost:8983/api/collections/meus_produtos/schema
```

Conforme a [documentação](https://lucene.apache.org/solr/guide/8_5/schema-api.html#add-a-new-field), estamos adicionando o campo `descricao_do_produto` com o tipo `text_pt` e `stored: true`.
Ou seja, o campo não servirá apenas para ser encontrado durante o pipeline de query, mas também poderá ser encontrado na resposta da busca.
No caso de um campo `stored: false` não é possível ter acesso à ele quando o documento for encontrado, só pro Solr utilizar na sua busca.
Na prática, isso quer dizer que o valor no seu formato original não é guardado com o objetivo de otimização (diminuição) do tamanho do índice.

Existem muitos outros parâmetros possíveis para esta ação de criar um campo, assim como muitas outras ações possíveis.
O objetivo maior aqui é mostrar que as possibilidades de personalização da busca são enormes e a documentação poderá ajudar a entender tudo que a plataforma oferece.

### Adicionando um documento

A API permite adicionar um documento da seguinte forma:
```shell
curl -X POST -H 'Content-Type: application/json' 'http://localhost:8983/solr/meus_produtos/update?commit=true' --data-binary '[
    {"id": "flight_666","descricao_do_produto": "Canecas do Iron Maiden"},
    {"id": "m0","descricao_do_produto": "Caneca do Metallica"},
    {"id": "m1","descricao_do_produto": "Camiseta do Metallica"},
    {"id": "m2","descricao_do_produto": "Caneta metálica"},
    {"id": "b2","descricao_do_produto": "Caneta azul"}
]'
```

### E finalmente a busca!

- Para buscar todos os documentos sem nenhum critério basta dizer que a query (`q`) deve buscar em todos os campos (`*`) em todos os valores (`*`).
```shell
curl -H 'Content-Type: application/json' 'http://localhost:8983/solr/meus_produtos/select?q=*:*'
```
A sintaxe então seria: `nome_do_campo:termo_de_busca`

- Para buscar dentro do campo que criamos seria assim:
```shell
curl -H 'Content-Type: application/json' 'http://localhost:8983/solr/meus_produtos/select?q=descricao_do_produto:caneca'
```

No resultado vimos os documentos `Caneca do Metallica` e `Canecas do Iron Maiden`, mostrando então resultado no plural e no singular.
Isso porque o tipo que usamos no pipeline de query e search foi o `text_pt`.
Dentro dele a classe [PortugueseLightStemFilterFactory](https://lucene.apache.org/core/8_5_1/analyzers-common/org/apache/lucene/analysis/pt/PortugueseLightStemFilterFactory.html) é responsável por extrair o radical, que neste caso é `canec`, conforme podemos ver no (schema)[http://localhost:8983/solr/#/meus_produtos/files?file=managed-schema]:
```xml
<fieldType name="text_pt" class="solr.TextField" positionIncrementGap="100">
    <analyzer>
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.StopFilterFactory" format="snowball" words="lang/stopwords_pt.txt" ignoreCase="true"/>
        <filter class="solr.PortugueseLightStemFilterFactory"/>
    </analyzer>
</fieldType>
```

- Se a busca for executada com o radical teremos o mesmo resultado:
```shell
curl -H 'Content-Type: application/json' 'http://localhost:8983/solr/meus_produtos/select?q=descricao_do_produto:canec'
```

## E agora?

- Para finalizar e remover o que construímos, basta parar os serviços:
```shell
docker-compose down -v -t 0
```

Muitos assuntos podem surgir dos vários pontos discutidos.
Como fazer a escrita em tempo real para que os novos produtos (ou alterados) fiquem disponíveis o mais rápido possível? Podemos discutir ferramentas para fazer [NRT](https://lucene.apache.org/solr/guide/8_5/near-real-time-searching.html) ou ferramentas como [Spark](https://github.com/lucidworks/spark-solr)

Também ficam temas como colocar em produção, manutenções, escalabilidade, os vários tipos de campos e estratégias de como montar o pipeline, dentre várias outras coisas.
Mas isso fica pro futuro. :)