![image](https://github.com/AlbertoFAraujo/R_Analise_RFM/assets/105552990/48658809-bff4-4275-8048-2601f8390643)

### Tecnologias utilizadas: 
| [<img align="center" alt="R_studio" height="60" width="60" src="https://github.com/AlbertoFAraujo/R_Petrobras/assets/105552990/02dff6df-07be-43dc-8b35-21d06eabf9e1">](https://posit.co/download/rstudio-desktop/) | [<img align="center" alt="ggplot" height="60" width="60" src="https://github.com/AlbertoFAraujo/R_Petrobras/assets/105552990/db55b001-0d4c-42eb-beb2-5131151c7114">](https://plotly.com/r/) | [<img align="center" alt="plotly" height="60" width="60" src="https://github.com/AlbertoFAraujo/R_Petrobras/assets/105552990/5f681062-c399-44af-a658-23e94b8b656f">](https://plotly.com/r/) | [<img align="center" alt="quantmod" height="60" width="60" src="https://github.com/AlbertoFAraujo/R_Analise_RFM/assets/105552990/a81c295f-2bd2-4aa8-8b7a-213d178ac3c7">](https://www.rdocumentation.org/packages/tidyverse/versions/2.0.0) | [<img align="center" alt="caret" height="60" width="60" src="https://github.com/AlbertoFAraujo/R_Analise_RFM/assets/105552990/feb90a81-2b4b-476d-a384-58923d4b5913">](https://www.rdocumentation.org/packages/caret/versions/6.0-94) | [<img align="center" alt="readxl" height="60" width="60" src="https://github.com/AlbertoFAraujo/R_Analise_RFM/assets/105552990/72528ffe-203d-4e10-9347-b04e8be4cb6a">](https://www.rdocumentation.org/packages/readxl/versions/1.4.3) | 
|:---:|:---:|:---:|:---:|:---:|:---:|
| R Studio | Ggplot2 | Plotly | Tidyverse | Caret | Readxl |

- **RStudio:** Ambiente integrado para desenvolvimento em R, oferecendo ferramentas para escrita, execução e depuração de código.
- **Ggplot2:** Pacote para criação de visualizações de dados elegantes e flexíveis em R.
- **Plotly:** Biblioteca interativa para criação de gráficos e visualizações em diversas linguagens.
- **Tidyverse:** Conjunto de pacotes para manipulação de dados, visualização e modelagem em R, seguindo o paradigma "tidy data".
- **Caret:** Pacote para treinamento e avaliação de modelos de machine learning em R.
- **Readxl:** Pacote para leitura de dados em formato Excel (XLSX/XLS) em R, facilitando a importação de planilhas para análise de dados.
<hr>

### Objetivo: 

Realizar a análise de cliente pelo método RFM :
- **Recência:** quão recente um cliente fez a compra;
- **Frequência:** com que frequência um cliente faz a compra;
- **Valor Monetário:** quanto dinheiro um cliente gasta em compras.

De acordo com essas métricas, é possível segmentar os clientes em grupos para entender quais deles compram muitas coisas com frequência, que compram poucas coisas, mas frequentemente, e que não compram nada há muito tempo.

Base de dados: https://archive.ics.uci.edu/dataset/502/online+retail+ii
<hr>

### Script R: 

```r
# Ajustar as casas decimais
options(scipen = 999, digits = 4)

# Definir um espelho de CRAN
options(repos = "http://cran.rstudio.com/")
```
### 1. Importação da base de dados e bibliotecas
Esta seção apresenta a importando das bibliotecas necessárias para manipulações e tratamento dos dados e também a importação do arquivo excel (.xls) da base de dados.
```r
library(dplyr)

# Lista dos pacotes necessários
pacotes <- c('tidyverse', 'ggplot2', 'caret', 'plotly',
             'readxl', 'rfm', 'stats','factoextra','plyr')

# Verifica se NÃO está instalado, se for verdadeiro
# Instala e carrega o pacote
pacotes %>% 
  lapply(function(pacote){
    if (!require(pacote, character.only = TRUE)) {
      install.packages(pacote)
      library(pacote, character.only = TRUE)
    }
  })
```
```r
# Verificar os pacotes instalados
list(search())
```
```r
# Carregar os dados da planilha
plan1 <- read_excel('online+retail+ii/online_retail_II.xlsx', sheet = 'Year 2009-2010')
plan2 <- read_excel('online+retail+ii/online_retail_II.xlsx', sheet = 'Year 2010-2011')
df <- rbind(plan1, plan2)
```
### Definição das variáveis

- **InvoiceNo:** Número da fatura. Nominal. Um número integral de 6 dígitos atribuído exclusivamente a cada transação. Se esse código começar com a letra 'c', ele indica um cancelamento;
- **StockCode:** Código do produto (item). Nominal. Um número integral de 5 dígitos atribuído exclusivamente a cada produto distinto;
- **Description:** Nome do produto (item);
- **Quantity:** As quantidades de cada produto (item) por transação. Numérico;
- **InvoiceDate:** Data e hora invictas. Numérico. O dia e a hora em que uma transação foi gerada;
- **UnitPrice:** Preço unitário. Numérico. Preço do produto por unidade em libras esterlinas (Â£);
- **CustomerID:** Número do cliente. Nominal. Um número integral de 5 dígitos atribuído exclusivamente a cada cliente;
- **Country:** Nome do país. Nominal. O nome do país onde o cliente reside.
------------------------------------------------------------------------
### 2. Análise Exploratória das variáveis
Nesta seção será explorado as variáveis para identificar os possíveis padrões e comportamentos, valores nulos, outliers e engenharia de atributos, se necessário.
```r
# Carregar as 10 primeiras entradas do dataset
head(df)
sample(df)
```
| Invoice | StockCode | Description                        | Quantity | InvoiceDate         | Price | Customer ID | Country        |
|---------|-----------|------------------------------------|----------|---------------------|-------|-------------|----------------|
| 489434  | 85048     | 15CM CHRISTMAS GLASS BALL 20 LIGHTS | 12       | 2009-12-01 07:45:00 | 6.95  | 13085       | United Kingdom |
| 489434  | 79323P    | PINK CHERRY LIGHTS                  | 12       | 2009-12-01 07:45:00 | 6.75  | 13085       | United Kingdom |
| 489434  | 79323W    | WHITE CHERRY LIGHTS                 | 12       | 2009-12-01 07:45:00 | 6.75  | 13085       | United Kingdom |
| 489434  | 22041     | RECORD FRAME 7" SINGLE SIZE        | 48       | 2009-12-01 07:45:00 | 2.10  | 13085       | United Kingdom |
| 489434  | 21232     | STRAWBERRY CERAMIC TRINKET BOX     | 24       | 2009-12-01 07:45:00 | 1.25  | 13085       | United Kingdom |
| 489434  | 22064     | PINK DOUGHNUT TRINKET POT          | 24       | 2009-12-01 07:45:00 | 1.65  | 13085       | United Kingdom |

```r
# Analisar as dimensões do dataset (linhas x colunas)
dim(df)
```
[1] 1067371       8

```r
# Verificar o nome das variáveis(colunas)
names(df)
```
[1] "Invoice"     "StockCode"   "Description" "Quantity"   
[5] "InvoiceDate" "Price"       "Customer ID" "Country" 

```r
# Verificar o tipo das variáveis
str(df)
```
tibble [1,067,371 × 8] (S3: tbl_df/tbl/data.frame)
- $ Invoice    : chr [1:1067371] "489434" "489434" "489434" "489434" ...
- $ StockCode  : chr [1:1067371] "85048" "79323P" "79323W" "22041" ...
- $ Description: chr [1:1067371] "15CM CHRISTMAS GLASS BALL 20 LIGHTS" "PINK CHERRY LIGHTS" "WHITE CHERRY LIGHTS" "RECORD FRAME 7\" SINGLE SIZE" ...
- $ Quantity   : num [1:1067371] 12 12 12 48 24 24 24 10 12 12 ...
- $ InvoiceDate: POSIXct[1:1067371], format: "2009-12-01 07:45:00" ...
- $ Price      : num [1:1067371] 6.95 6.75 6.75 2.1 1.25 1.65 1.25 5.95 2.55 3.75 ...
- $ Customer ID: num [1:1067371] 13085 13085 13085 13085 13085 ...
- $ Country    : chr [1:1067371] "United Kingdom" "United Kingdom" "United Kingdom" "United Kingdom" ...

```r
# Transformando a variável "country" em factor, devido ser categórica nominal
table(df$Country)
df$Country <- as.factor(df$Country)
```
```r
# Verificando os valores nulos das variáveis
colSums(is.na(df))
```
| Invoice | StockCode | Description | Quantity |     InvoiceDate     | Price | Customer ID |  Country  |
|---------|-----------|-------------|----------|----------------------|-------|-------------|-----------|
|    0    |     0     |     4382    |     0    |          0           |   0   |   243007    |     0     |

Como trabalharemos com operações matemáticas, será necessário que as variáveis não apresentem valores nulos, bem como a dimensão da base possui 1067371 linhas, então removeremos os valores ausentes das variáveis "Description" e "Customer ID", pois não podemos inputar nenhuma informação para suprir essas ausências, visto que irá influenciar nas estatísticas de compra do cliente e portanto na análise RFM.

```r
# Criando uma coluna de Preço Total
df$Total_Price <- df$Quantity * df$Price 
head(df$Total_Price)
```
[1]  83.4  81.0  81.0 100.8  30.0  39.6

```r
# Criando uma cópia do dataset original
df_copy <- df
```
```r
# Removendo os valores ausentes do dataset

df_copy <- na.omit(df)

# Seleciona os valores que NÃO são do tipo C na coluna "Invoice"
df_copy <- df_copy[!grepl("C",df_copy$Invoice),]
dim(df_copy)

# Resultado final do dataset após a limpeza
print(paste0("Foram removidas: ", 
             dim(df)[1] - dim(df_copy)[1],
             "(",
             round(((dim(df)[1] - dim(df_copy)[1])/dim(df)[1]) * 100, digits = 2),
             "%) linhas após a limpeza"))
```
[1] 805620      9

[1] "Foram removidas: 261751(24.52%) linhas após a limpeza"

```r
# Verificando a disposição da variável "Total_Price"

ggplot(
  df_copy, 
  mapping = aes(x = Total_Price)) + 
  geom_density(fill = "#ff79ae", color = "#282a36", alpha = 3.5) + 
  labs(title = "Distribuição da variável Total_Price")
  
```
![image](https://github.com/AlbertoFAraujo/R_Analise_RFM/assets/105552990/cdcb7c38-2e04-4137-a791-2e1fd6ff04ff)

```r
# Quantidade de transações
length(df_copy$`Customer ID`)
```
[1] 805620

```r
# Quantidade de clientes
length(unique(df_copy$`Customer ID`))
```
[1] 5881

```r
# Transformando em data table
df_copy <- data.table::data.table(df_copy)

# Valor Monetário por cliente
valor_monetario <- df_copy[,.(Total = sum(Total_Price)), by = `Customer ID`][order(desc(Total))]
print(valor_monetario)
```
| Customer ID |  Total  |
|-------------|---------|
|    18102    |  608822 |
|    14646    |  528603 |
|    14156    |  313946 |
|    14911    |  295973 |
|    17450    |  246973 |
|    13694    |  196483 |
|    17511    |  175604 |
|    16446    |  168473 |
|    16684    |  147143 |
|    12415    |  144458 |

```r
print(summary(valor_monetario$Total))
```
|   Min.   | 1st Qu. | Median |  Mean  | 3rd Qu. |   Max.   |
|----------|---------|--------|--------|---------|----------|
|     0    |    348  |  898   |  3017  |   2304  |  608822  |

```r
# Cálculo da Recência
max(df_copy$InvoiceDate)
data_ref <- as.Date.character("25/12/2011", "%d/%m/%Y")

converter_data <- function(x){
  options(digits.secs = 3)
  return(as.Date(as.POSIXct(x$InvoiceDate,'GMT')))
}

# Alterando a coluna InvoiceDate para ocultar o horário
df_copy$InvoiceDate <- converter_data(df_copy)
head(df_copy)
```
```r
# Calculando o RFM
calculo_rfm <- df_copy[,
        .(Recencia = as.numeric(data_ref - max(InvoiceDate)),
          Frequencia = .N,
          Monetario = sum(Total_Price),
          Primeira_compra = min(InvoiceDate)), 
        by = `Customer ID`]

calculo_rfm
```
| Customer ID | Recencia | Frequencia | Monetario | Primeira_compra |
|-------------|----------|------------|-----------|-----------------|
|    13085    |   173    |     84     |  2433.28  |    2009-12-01   |
|    13078    |    19    |    801     | 29532.45  |    2009-12-01   |
|    15362    |   464    |     40     |   613.08  |    2009-12-01   |
|    18102    |    16    |    1058    | 608821.65 |    2009-12-01   |
|    12682    |    19    |    1039    | 24033.91  |    2009-12-01   |
|    18087    |   114    |     88     | 14761.52  |    2009-12-01   |
|    13635    |    83    |    162     |  2999.16  |    2009-12-01   |
|    14110    |    19    |    400     | 12987.95  |    2009-12-01   |
|    12636    |   754    |     1      |   141.00  |    2009-12-01   |
|    17519    |    33    |    222     |  5109.47  |    2009-12-01   |

```r
# Removendo os Outliers (Valores muito distantes da média)
Q1 <- quantile(calculo_rfm$Monetario, .25)
Q3 <- quantile(calculo_rfm$Monetario, .75)
IQR <- IQR(calculo_rfm$Monetario)

calculo_rfm <- subset(
                calculo_rfm,
                calculo_rfm$Monetario >= (Q1 - 1.5*IQR) &
                calculo_rfm$Monetario <= (Q3 + 1.5*IQR) 
                                    )
head(calculo_rfm)
```
| Customer ID | Recencia | Frequencia | Monetario | Primeira_compra |
|-------------|----------|------------|-----------|-----------------|
|    13085    |   173    |     84     |   2433.3  |    2009-12-01   |
|    15362    |   464    |     40     |    613.1  |    2009-12-01   |
|    13635    |    83    |    162     |   2999.2  |    2009-12-01   |
|    12636    |   754    |     1      |    141.0  |    2009-12-01   |
|    17519    |    33    |    222     |   5109.5  |    2009-12-01   |
|    16321    |    88    |     23     |    604.6  |    2009-12-01   |

### 3. Machine Learning - Clusterização Kmeans
Nesta seção será criado um modelo de algoritmo não supervisionado do tipo Cluster Kmeans, na qual agrupa os dados em clusters (grupos) com base em suas características, sem a necessidade de rótulos prévios. O algoritmo tenta encontrar estruturas nos dados agrupando-os em um número pré-definido de clusters, minimizando a variação dentro de cada cluster e maximizando a variação entre os clusters.

```r
# Set seed
set.seed(42)
```
```r
# Cria uma lista
resultados <- list()

# Filtrando somente as colunas necessárias
dados_rfm <- calculo_rfm %>% 
  select(Recencia, Frequencia, Monetario)

dados_rfm
```
| Recencia | Frequencia | Monetario |
|----------|------------|-----------|
|   173    |     84     |  2433.28  |
|   464    |     40     |   613.08  |
|    83    |    162     |  2999.16  |
|   754    |      1     |   141.00  |
|    33    |    222     |  5109.47  |
|    88    |     23     |   604.55  |
|   430    |     63     |  1386.12  |
|   754    |     13     |   148.30  |
|    25    |    166     |  3472.41  |
|   443    |    115     |  1703.07  |

```r
# Cria o modelo e divide em 5 grupos
# com base na distância entre os pontos
modelo_kmeans <- kmeans(dados_rfm,
                        centers = 5,
                        iter.max = 50)
```
```r
# Verificando o modelo criado
modelo_kmeans
```
```r
# Plot do modelo
resultados$plot <- fviz_cluster(modelo_kmeans,
                                data = dados_rfm,
                                geom = c('point'),
                                ellipse.type = 'euclid')
```
```r
# Organiza os dados
# Retornando os dados do ID para identificar os clientes
dados_rfm$Customer_ID <- calculo_rfm$`Customer ID`
dados_rfm$clusters <- modelo_kmeans$cluster


resultados$data <- dados_rfm

grafico <- resultados[1]
grafico
```
![image](https://github.com/AlbertoFAraujo/R_Analise_RFM/assets/105552990/b1e48993-9543-4c5c-9eb1-2579c58519a5)

```r
tab <- as.data.frame(resultados[2])
head(tab)
```
| Recencia | Frequencia | Monetario | Customer_ID | Clusters |
|----------|------------|-----------|-------------|----------|
|   173    |     84     |  2433.3   |    13085    |    1     |
|   464    |     40     |   613.1   |    15362    |    2     |
|    83    |    162     |  2999.2   |    13635    |    1     |
|   754    |      1     |   141.0   |    12636    |    2     |
|    33    |    222     |  5109.5   |    17519    |    5     |
|    88    |     23     |   604.6   |    16321    |    4     |

```r
# Analisando os resultados da tabela dos cliente
clusters_clientes <- data.table::data.table(tab)
```
```r
# Frequência dos clientes por Clusters
clusters_clientes[,
                  .(Total = .N),
                  by = data.clusters
                  ][order(desc(Total))]
```
| data.clusters | Total |
|---------------|-------|
|       2       |  2376 |
|       4       |  1352 |
|       3       |   734 |
|       1       |   473 |
|       5       |   318 |

```r
# Média de Recencias por clusters
clusters_clientes[,
                  .(Media_recencia = median(data.Recencia),
                    Media_frequencia = median(data.Frequencia),
                    Media_monetario = median(data.Monetario)),
                  by = data.clusters
                  ]
```
| data.clusters | Media_recencia | Media_frequencia | Media_monetario |
|---------------|----------------|------------------|-----------------|
|       1       |      55.0      |       159        |      2951.1     |
|       2       |     349.5      |        19        |       293.6     |
|       5       |      47.0      |       228        |      4370.4     |
|       4       |      96.0      |        56        |       949.5     |
|       3       |      83.0      |       103        |      1823.7     |

```r

# Plotagem dos gráficos de RFM
plotagem <- function(coluna){
  
  titulo_coluna <- substr(coluna, 6, nchar(coluna))
  
# Criar o boxplot com Plotly
boxplot_plotly <- plot_ly(data = clusters_clientes, 
                          x = ~data.clusters,
                          y = ~get(coluna),
                          type = "box"
                          ) %>%
  layout(title = paste("Boxplot de ", titulo_coluna, " por cluster"),
         xaxis = list(title = "Cluster"),
         yaxis = list(title = titulo_coluna))

# Exibir o boxplot
return(boxplot_plotly)
}
```
```r
plotagem("data.Recencia")
```
![1](https://github.com/AlbertoFAraujo/R_Analise_RFM/assets/105552990/ff70559e-f7b4-4ab3-9f83-f068bdc41de6)

```r
plotagem("data.Frequencia")
```
![2](https://github.com/AlbertoFAraujo/R_Analise_RFM/assets/105552990/d42cc634-7970-495f-95c7-694d23458ae0)

```r
plotagem("data.Monetario")
```
![3](https://github.com/AlbertoFAraujo/R_Analise_RFM/assets/105552990/087e89a3-0930-4254-864d-84ef1febdba6)

### Análise dos Clusters:

- **Recência Média:** A recência média representa o quão recentemente os clientes de um cluster realizaram uma transação. Dentro deste contexto, o cluster 5 apresenta a melhor média (menor), indicando que são possíveis usuários mais ativos e/ou engajados com a marca. Em contrapartida, o cluster 2 apresenta uma média 7 vezes maior que o cluster 5, possivelmente este grupo não está envolvido com a marca ou estão se afastando;

- **Frequência Média:** A frequência média indica com que frequência os clientes de um cluster realizam transações. O cluster 5 apresenta uma frequência média de 228, indicando clientes possivelmente fiéis e que retornam regularmente para realizarem novas compras. O cluster 2 apresenta um frequência de 19, indicando clientes menos engajados e que realizam compras de forma esporádicas;

- **Valor Monetário Médio:** O valor monetário representa o valor médio das transações realizadas pelos clientes de um cluster. O cluster 5 apresenta uma média de 4370.4, demonstrando que os clientes estão gastando mais em cada transação, o que indica maior lealdade ou interesse em serviços e nos produtos da marca.

