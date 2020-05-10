# Um gole de Máquinas de Busca

Hoje em dia é comum utilizar máquina de busca para pesquisar conteúdo na internet como Google, DuckDuckGo, Bing, Yahoo, dentre outras. Costumamos usá-las sem questionar ou interessar-se em: Como o conteúdo foi parar lá? Como é feita a ordenação do resultado? Como pesquisar de forma simples e eficaz? Como dar a resposta rapidamente ao usuário?

Pensando nessas perguntas, discutiremos nesse artigo sobre como as máquinas de busca funcionam.

## O que uma máquina de busca se propõe a resolver?

Para contextualizar, inicialmente falaremos sobre a importância da máquina de busca na internet. Quando a internet se popularizou, já existia uma quantidade significativa de sites e de uma grande variedade de temas, portanto, era difícil descobri-los sem recomendações de pessoas: seja através do bom e velho boca a boca, ou através de hyperlinks que por ventura chegassem até elas.

O primeiro buscador na internet foi o Wandex desenvolvido por Matthew Gray no MIT, seguido por vários outros até chegarmos nos gigantes da atualidade: Yahoo, Bing e Google.

Atualmente qualquer site possui uma máquina de busca nos bastidores. Em todo e-commerce há uma caixa de busca que utiliza os mesmos conceitos dos gigantes da internet unificando todo o resultado através de uma simples consulta. Pode-se notar que é uma tecnologia amplamente difundida em empresas de todos os portes.

Um pulo do gato importante das máquinas de busca é atacar o problema em fases, ou melhor, tempos distintos. São eles:
1. Consumo de conteúdo
2. Consulta do usuário

<img src="imagens/imagem1.png">


#### Consumo de Conteúdo
	
Nesta etapa consome-se o conteúdo da fonte dos dados que desejamos buscar como por exemplo: sites (blogs, notícias, artigos), arquivos de documentos (pdfs, docs, txts), bancos de dados, etc. Ao final armazena-se este conteúdo na base de dados da máquina de busca (também conhecida como índice).

Detalhando um pouco mais esta fase, imaginemos que precisamos consumir (ou indexar) um determinado site.
Para tanto necessitaremos de um consumidor deste tipo de fonte. Chamamos este componente de Crawler ou WebCrawler que nada mais é que um robô que varre todo o site alvo, navegando por cada link encontrado, armazenando todo o conteúdo no índice.

*Nota: O consumo de conteúdo também pode ocorrer de forma mais avançada através de integrações de sistemas.*
	
#### Consulta do Usuário

Etapa responsável por auxiliar o usuário a encontrar o conteúdo mais relevante para sua consulta (ou query).

Estes auxílios podem se dar de diversas maneiras, seja através de sugestão de termos ou transformação da query original através de correção ortográfica ou expansão de sinônimos por exemplo.

Portanto, esta fase envolve componentes e técnicas para habilitar funcionalidades conhecidas como “Vocês quis dizer?”, “Filtros” e “Auto completar” que poderão ser abordadas em posts futuros.

Dado esse contexto, podemos detalhar os componentes de uma máquina de busca da seguinte forma:

##### Conectores
Auxiliam no trabalho de conectar e consumir diversas fontes de dados, como varrer um site, fazer integrações com outros sistemas (API) e rastrear um sistema de arquivos.
Sua responsabilidade portanto, é injetar dados na máquina de busca.
		
##### Processamento de Conteúdo
Antes mesmo do conteúdo ser persistido no índice, estes dados crus passam por um conjunto de processos de tratamento de normalização linguística e enriquecimento de dados, de forma que o conteúdo se torne mais amplamente *buscável* pelo usuário. Vale ressaltar que essa etapa é uma das mais importantes de uma máquina de busca.

##### Índice 
Responsável por armazenar os dados normalizados do conteúdo. Por ora vamos abstrair o conceito como uma tabela de um banco dados. Vale notar no entanto, que a magia da velocidade dos motores de busca vem justamente da diferença entre como os dados são persistidos num banco de dados tradicional e no índice de busca.

Segue abaixo uma representação de dois documentos gravados em um banco de dados:  

<img src="imagens/imagem2.png">

Como podemos observar acima, a chave tem um valor único que possibilita fazer uma busca simples. Se tivermos a necessidade de procurar pelo "valor" do documento, a busca se torna inviável, pois teremos que processar o texto do documento. 

Na estrutura de índice invertido, que é utilizado por máquina de busca, esse conceito é diferente:

<img src="imagens/imagem3.png">

As chaves são palavras do texto e os valores são os ids dos documentos (documento 1 e documento 2). Os artigos e preposições foram removidos para economizar espaço e deixar somente palavras relevantes.

Ao contrário de um banco de dados relacional, a busca nessa estrutura de dados é mais eficiente, porque todas as palavras são chaves. Entretanto,  é necessário normalizar o processamento da consulta do usuário. 


##### Processamento de Consulta
Da mesma forma que o Processamento de Conteúdo trata o conteúdo, esta fase se responsabiliza por tratar linguisticamente o termo de busca do usuário de forma que este se compatibilize com a natureza do conteúdo do índice. Em outras palavras, esta fase garante que ambos conteúdo e termo de busca estão igualmente normalizados.
Nesta fase também é possível, como dito anteriormente, aplicar tratamentos como enriquecer a consulta com sinônimos, tratar erros ortográficos, entre outros.

## Para finalizar...

Podemos notar o quão importante é uma máquina de busca em nosso cotidiano quando nos damos conta de que é quase impossível começar sua jornada na internet sem realizar uma busca.

Dia após dia, empresas investem em pesquisas com a proposta de modernizar e torná-las mais eficientes.

Nos próximos artigos aprofundaremos como os dados são normalizados no índice de busca.

Amigão, passa a régua pra gente?

### Referências
- [https://pt.wikipedia.org/wiki/Motor_de_busca](https://pt.wikipedia.org/wiki/Motor_de_busca)
