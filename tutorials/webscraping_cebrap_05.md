# Tutorial 8 - Formulários na web

Até agora trabalhamos com exemplos nos quais retiramos informação de páginas em html, mas não enviamos nenhuma informação ao servidor com as quais estamos nos comunicando (exceto manualmente). Contudo, é muito comum nos depararmos com formulários em páginas que queremos raspar. Formulários, na maioria das vezes, aparecerão como caixas de consulta acompanhada de um botão.

Mecanismos de busca (Google, DuckDuckGo, etc) têm formulários nas suas páginas iniciais. Portais de notícia ou de Legislativos têm formulários de busca (como o que usamos manualmente no caso da Folha de São Paulo ou nas tabelas da ALESP e AL-AP). Por vezes, mesmo para "passar" de página nos deparamos com um formulário.

Neste tutorial vamos aprender a preencher um formulário, enviá-lo ao servidor da página e capturar sua resposta.

Faremos um exemplo simples, o buscador da ALESP.

## _rvest_, formulários e ALESP

Vamos começar carregando os pacotes _rvest_, _dplyr_ e _tidyr_:

```{r}
library(rvest)
library(dplyr)
library(tidyr)
```

Nosso primeiro passo ao lidar com formulários será estabelecer uma conexão com o servidor, antes mesmo de capturar a página na qual o formulário está. Damos a essa conexão o nome de "session", e utilizamos a função _html\_session_ para criá-la observamos a página em que preenchemos o formulário para a busca observável [aqui]. Vamos ver como funciona?

```{r}
alesp_url <- "https://www.al.sp.gov.br/alesp/pesquisa-proposicoes/"

alesp_session <- html_session(alesp_url)
```

Estabelecida a conexão, precisamos conhecer o formulário. Começamos, obviamente, obtendo o código HTML da página do formulário. Na sequência, vamos extrair do código da página os formulários. Como tabelas em HTML, fomrulários tem suas tags próprias e contamos no pacote _rvest_ com uma função que extrai uma lista contendo todos os formulários da página:

```{r}
alesp_pagina <- read_html(alesp_session)

alesp_form_list <- html_form(alesp_pagina)

alesp_form_list
```

No caso do buscador da Google, há apenas um formulário na página. Com dois colchetes, extraímos o formulário que está na primeira posição da lista de formulários. Você verá que no caso do STF, obteremos mais de um formulário e precisaremos identificar qual queremos.

```{r}
alesp_form <- alesp_form_list[[3]]

class(alesp_form)

alesp_form
```

Examine o objeto que contém o formulário. Ele é um objeto da classe "form" e podemos observar todos os parâmetros que o compõe, ou seja, tudo aquilo que pode ser preenchido para envio ao servidor, ademais dos botões de submissão.

Vá para o navegador e inspecione a caixa de busca. Você observará que cada "campo" do formulário é uma tag "input". O atributo "type", define se será oculto ("hidden"), texto ("text") ou botão de submissão ("submit"). Por sua vez, o atributo "name" dá nome ao campo.

Alguns "inputs" já contêm valores (no atributo "values"). No nosso exemplo, os botões são destacados. Estes últimos jáidentificaram o idioma ("hl") e o enconding ("ie") com o qual trabalhamos.

O que nos interesse preencher, obviamente, é o "input" chamado "text". Para capturar mais dados, podemos indicar quantos resultados queremos por página com "rowsPerPage". Vamos manter a busca por "previdencia" e limitar a 400 resultados.

Vamos, então, preencher os campos com a função _set\_values_:

```{r}
alesp_form <- set_values(alesp_form,
                          'text' = "previdencia",
                          'rowsPerPage' = 400)
```

Simples, não? Colocamos o objeto do formulário no primeiro parâmetro da função e os campos a serem preenchidos na sequência, tal como no exemplo.

Reexamine agora o formulário. Você verá que "text" está preenchido e "rowsPerPage" teve o número de resultados alterado:

```{r}
alesp_form
```

Legal! Agora vamos fazer a submissão do formulário. Na _submit\_form_, precismos informar a sessão que criamos (conexão com o servidor), o formulário que vamos submeter e o nome do botão de submissão. No nosso exemplo, no entando, o botão de submissão é identificado como "NULL". Veja o exemplo: 

```{r}
alesp_submission <- submit_form(session = alesp_session, 
                                 form = alesp_form)
```

Pronto! Agora basta raspar o resultado como já haviámos feito antes. A página que queremos raspar é o objeto que resulta da função _submit\_form_. Abra o resultado de uma busca na ALESP e tente entender o código abaixo.

```{r}
alesp_resultado <- read_html(alesp_submission) 
  
table_resultado <- html_table(alesp_resultado)

dados <- table_resultado[[1]] 
seq_impar <- seq(1,800,2)
dados <- dados[seq_impar,]
  
dados <- dados %>%
  separate("Documento", into = c("Documento", "Ementa"), sep = "\r\n\t\t\t\t\t\t\r\n\t\t\t\t\t\t\r\n\t\t\t\t\t") 
dados$Documento <- gsub("\r\n\t\t\t\t\t\t\t\t", " ", dados$Documento)

nodes_links <- html_nodes(alesp_resultado, xpath = "//*[@id='lista_resultado']/table/tbody/tr/td[2]/a[@style ='color: #000000']")

links <- html_attr(nodes_links, name = "href")
links <- paste0("https://www.al.sp.gov.br/", links)
links <- as.vector(links)

dados <- bind_cols(dados, links)
```

