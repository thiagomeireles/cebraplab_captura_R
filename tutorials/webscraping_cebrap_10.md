# Tutorial 10 - Análise de texto no R - pacote _tm_

## Corpus e o pacote _tm_

O pacote mais popular para trabalharmos com texto no R se chama _tm_ ("Text Mining"). Vamos carregá-lo e passar por algumas funções do pacote para, então, trabalharmos com uma nova classe de objeto: Corpus.

Carregue o pacote:

```{r}
library(tm)
```

Uma boa prática ao trabalharmos com texto é transformarmos todas as palavras em minúsculas (exceto, obviamente, quando a diferenciação importar). _tolower_, função da biblioteca básica do R, cumpre a tarefa e vamos criar um objeto "noticias2", que será nossa versão modificada dos noticias.

```{r}
noticias2 <- tolower(noticias)
noticias2[1]
```

Pontuação também costuma ser um problema ao trabalharmos com texto. A não ser que nos interesse recortar o texto usando os pontos como marcas, convém aplicarmos a função _removePunctuation_ do pacote _tm_ para retirar a pontuação:

```{r}
noticias2 <- removePunctuation(noticias2)
noticias2[1]
```

O mesmo ocorre com números. Se não forem de interesse específico, melhor extraí-los. A função _removeNumbers_ resolve o problema:

```{r}
noticias2 <- removeNumbers(noticias2)
noticias2[1]
```

Vamos olhar novamente para a nuvem de palavras, usando agora o nosso objeto de texto transformado:

```{r}
wordcloud(noticias2, max.words = 50)
```

Note que as palavras com mais frequência são aquelas de maior ocorrência na língua portuguesa. Qual é a utilidade de incluí-las na análise se sabemos que são frequentes?

O pacote _tm_ oferece a função _stopwords_. Essa função gera um vetor com as palavras mais frequentes da língua indicada:

```{r}
stopwords("pt")
```

Com a função _removeWords_ podemos excluir as "stopwords" da língua portuguesa de nosso conjunto de textos:

```{r}
noticias2 <- removeWords(noticias2, stopwords("pt"))
noticias2[1]
```

Vamos aproveitar que já fizemos inúmeras remoções -- pontuação, números e stopwords -- e retirar os espaços excedentes que sobraram no texto:

```{r}
noticias2 <- stripWhitespace(noticias2)
noticias2[1]
```

E vamos repetir nossa nuvem de palavras:

```{r}
wordcloud(noticias2, max.words = 50)
```

Note, porém, o destaque a "datafolha". As matérias fazem muitas referências ao próprio instituto. Outros termos relacionados ao instituto são "levantamento", "entrevistados", "pesquisa" e "levantamento". 

Podemos, então, incrementar a lista de stopwords com padrões que conhecemos:

```{r}
stopwords_pt <- c(stopwords("pt"), "datafolha", "entrevistados", "pesquisa", "levantamento")
```

E gerar um novo objeto removendo as novas stopwords:

```{r}
noticias3 <- removeWords(noticias2, stopwords_pt)
wordcloud(noticias3, max.words = 50)
```

Com uma imagem, podemos ter alguma ideia dos temas e termos recorrentes nas notícias sobre eleições no DataFolha.

Uma funcionalidade do pacote _tm_ não muito bem implementada em português é a "stemização de palavras". "Word Stem", em linguística, significa extrair de um conjunto de palavras apenas a raiz da palavra ou o denominador comum de várias palavras. Por exemplo, "discurso", "discursivo", "discursar" e "discussão", "stemizadas", deveriam se tornar "discus", e poderíamos agrupá-las para fins analíticos. Vamos ver um exemplo em inglês:

```{r}
stemDocument(c("politics", "political", "politically"), language = "english")
```

Vamos ver o resultado da função _stemDocument_ no primeiro discurso:

```{r}
noticias4 <- stemDocument(noticias2, language = "portuguese")
noticias4[1]
```

Hummmm... meio estranho, não? Mas você pegou o espírito. Vamos seguir em frente

### Tokenização

Tokenização de um texto significa a separação em pequenos "tokens", que podem ser palavras ou n-grams, que são pequenos conjuntos de palavras. Bigrams, por exemplo, são pares de palavras. Voltaremos a esse tópico adiante e com mais cuidado. Mas vamos aproveitar o objeto tal como está para apresentarmos uma função do pacote _stringr_ que deixamos propositalmente para trás: _str\_split_. Como as palavras estão separadas por espaço, no resultado final será uma lista contendo um vetor de tokens para cada discurso:

```{r}
tokens <- str_split(noticias2, " ")
```

_unlist_ transforma a lista em um vetor único:

```{r}
unlist(tokens)
```

### Corpus

Corpus, em linguística, é um conjunto de textos, normalmente grande e em formato digital. Um corpus é composto pelo conteúdo dos textos e pelos metadados de cada texto. Na linguagem R, Corpus é também uma classe de objetos do pacote _tm_ e à qual podemos aplicar uma série de funções e transformações.

Vamos ver como criar um Corpus.

Em primeiro lugar, é preciso uma fonte. A fonte pode ser um vetor, um data frame ou um diretório. Vejamos os dois primeiros, começando com o vetor com o qual já estamos trabalhando:

```{r}
noticias_source <- VectorSource(noticias)
```

"noticias_source" é um objeto que apenas indica uma fonte de textos para funções do pacote _tm_. Para criar um Corpus, utilizados a função _VCorpus_ (volatile corpus, com o qual vamos trabalhar, e que armazena os dados na memória) ou _PCorpus_ (permanent corpus, usada para quando os dados estão em uma base externa ao R).

```{r}
noticias_corpus <- VCorpus(noticias_source)
```

Vamos observar o objeto "noticias_corpus" e sua classe:

```{r}
noticias_corpus
class(noticias_corpus)
```

Veja que um VCorpus contém "Metadata" e "Content". Neste caso, não temos nenhum metadado sobre os noticias, mas poderíamos criar. Vamos observar o que há na primeira posição de um VCorpus. (hey, note que um VCorpus é uma lista!)

```{r}
str(noticias_corpus[[1]])
```

Em metadata temos diversas variáveis: author, description, id, language, etc. Veja que id está preenchido com a ordem dos noticias e a língua está em inglês, por "default". Neste exercícios temos mais controle sobre os metadados, pois capturamos os textos de uma fonte específica, mas seria legal armazenar os metadados de um Corpus para compartilhá-lo ou trabalhar com Corpora (plural de Corpus) mais complexos.

Aliás, metadados são a única boa razão para trabalharmos com Corpus e não com vetores. Guardar informações sobre os textos é fundamental para selecionarmos subconjuntos e produzirmos análise.

Vamos reabrir os dados usando um data frame como fonte. Vamos criar um:

```{r}
noticias_df <- data.frame(doc_id = as.character(1:length(noticias)), 
                          text = noticias,
                          stringsAsFactors = F)
str(noticias_df)
```

E repetir o processo, com a diferença que utilizamos _DataframeSource_ para indicar a fonte dos dados:

```{r}
noticias_df_source <- DataframeSource(noticias_df)
noticias_df_corpus <- VCorpus(noticias_df_source)
```

Mesma coisa, não?

Ao trabalharmos com Corpus, não aplicamos diretamente as funções do pacote _tm_. Em vez disso, utilizamos a função _tm\_map_, que aplica uma outra função a todos os elementos do Corpus. Esse uso lembra as funções do pacote _purrr_ e da família _apply_, caso você tenha lido sobre elas alhures. Observe a remoção de pontuação com _removePunctuation_:

```{r}
noticias_corpus <- tm_map(noticias_corpus, removePunctuation)
```

A aplicação de qualquer função do pacote _tm_ segue este procedimento. Quando a função não pertence ao pacote _tm_, porém, precisamos "passá-la" dentro da função _content\_transformer_:

```{r}
noticias_corpus <- tm_map(noticias_corpus, content_transformer(tolower))
```

Se você criar uma função para alteração de um texto, você deve utilizar _content\_transformer_ também.

Mais dois exemplos, com _removeNumbers_ e _removeWords_:

```{r}
noticias_corpus <- tm_map(noticias_corpus, removeNumbers)
noticias_corpus <- tm_map(noticias_corpus, removeWords, stopwords("pt"))
```

Porque tanto trabalho? Para trabalharmos com Corpus, que tem a vantagem de armazenar os metadados, em vez de um vetor.

Para poupar seu trabalho, você pode "envelopar" todas as transformações que quiser produzir em um Corpus em uma função:

```{r}
limpa_corpus <- function(corpus){
  
  corpus <- tm_map(corpus, removePunctuation)
  corpus <- tm_map(corpus, content_transformer(tolower))
  corpus <- tm_map(corpus, removeNumbers)
  corpus <- tm_map(corpus, removeWords, stopwords("pt"))

  corpus
}
```

E aplicar a função aos Corpora com os quais estiver trabalhando:

```{r}
noticias_corpus <- limpa_corpus(noticias_corpus)
```

### Matriz Documento-Termo

O principal uso do pacote _tm_ é gerar uma matriz de documentos-termos ("dtm"). Basicamente, essa matriz tem cada documento na linha e cada termo na coluna. O conteúdo da célula é a frequência do termo em cada documento.

```{r}
dtm_noticias <- DocumentTermMatrix(noticias_corpus)
```

Veja um fragmento da "dtm" que criamos (documentos 101 a 105 e termos 996 a 1000):

```{r}
as.matrix(dtm_noticias[101:105, 996:1000])
```

Se quisermos rapidamente olhar os termos com frequência maior do que, digamos, 500:

```{r}
findFreqTerms(dtm_noticias, 500)
```

Há uma série de usos para a classe Corpus do pacote _tm_ para mineração de texto. Não vamos explorá-los e você pode buscar sozinh@. Vamos adotar agora uma abordagem que não envolve a criação de um Corpus.
