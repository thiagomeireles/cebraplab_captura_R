## Abrir csv

Salvamos em um arquivo _.csv_ os dados raspados no site do Datafolha com instabilidade. Abaixo apresentamos como abri-los sem necessidade de raspar mais uma vez.

```{r}
library(readr)
dados_noticias <- read_csv("https://raw.githubusercontent.com/thiagomeireles/cebraplab_captura_R/master/dados_noticias.csv",
                  col_types = cols(X1 = col_skip()),
                  locale = locale(encoding = "ISO-8859-1"))
```
