---
title: "Aula 1 — Introdução prática ao Web Scraping no contexto de RI (Python via reticulate)"
author: "Prof. MSc. Weslley Rodrigues"
date: "2025-07-28"
output:
  prettydoc::html_pretty:
    theme: cayman
    toc: true
    toc_depth: 3
    highlight: github
---

```{r message=FALSE, warning=FALSE, include=FALSE}
# Configuração do ambiente Python via reticulate (padrão do professor)
library(reticulate)
python_path <- "/Users/mr.weslley/.virtualenvs/r-reticulate/bin/python3"
use_python(python_path, required = TRUE)
py_config()

# Opções gerais de knitr
knitr::opts_chunk$set(
  echo = TRUE,
  message = FALSE,
  warning = FALSE,
  fig.align = "center",
  fig.width = 8,
  fig.height = 4
)
```

## Nota inicial

Turma, este documento replica o notebook Python no **RStudio** usando `reticulate`, mantendo **todos os códigos originais** em blocos `{python}`.

Disponibilizei na sala online o ".ipynb" para que poosam utilizar na IDE que preferir.

------------------------------------------------------------------------

# Aula 1 — Introdução prática ao Web Scraping no contexto de RDI

### **Objetivo**

1)  Apresentar um primeiro pipeline de **coleta de dados web** com `requests` + `BeautifulSoup`, conectando cada etapa ao ciclo de **Recuperação da Informação (RI)**: 1) **Coleta** (HTTP request e parsing do HTML);

\
2) **Representação/Indexação** (limpeza e estruturação em tabela);

\
3) **Busca** (seleção e filtragem dos elementos relevantes);\
4) **Avaliação** (verificação de qualidade, cobertura e precisão dos dados coletados).

**Resultados esperados:** ao final, a turma compreende a anatomia de uma página, seleciona elementos com segurança e produz um dataset mínimo para exploração posterior.

# Atividade Prática: Web Scraping com Python

RDI Aula 01 - Introdução - 1/2025 -10022025

# Passo 1: Importar as bibliotecas necessárias

```{python}
# Bibliotecas necessárias
import requests           # Realiza requisições HTTP para acessar páginas web
from bs4 import BeautifulSoup  # Faz a análise e manipulação do conteúdo HTML/XML
```

## Anatomia da requisição e do HTML (DOM)

### Diagrama de referência (Request/Response + DOM)

```{r, echo=FALSE, out.width='90%', fig.align='center'}
knitr::include_graphics("A_two-part_educational_digital_illustration_explai.png")
```

### Exemplo mínimo de HTML (para leitura da árvore DOM)

``` html
<!doctype html>
<html>
  <head><title>Exemplo</title></head>
  <body>
    <div class="card">
      <h2 class="title">Título da Vaga</h2>
      <p class="location">Brasília, DF</p>
      <a href="/detalhes" id="detalhe">ver detalhes</a>
    </div>
  </body>
</html>
```

**O que observar**

Tag raiz: `<html>`;

filhos diretos: `<head>` e `<body>`.

Estrutura hierárquica (árvore DOM)

Cada tag contém nós/atributos/texto.

Atributos: `class="card"`, `class="title"`, `class="location"`, `id="detalhe"`.

Texto visível: "Título da Vaga", "Brasília, DF", "ver detalhes".

### Demonstração rápida

```{python}
from bs4 import BeautifulSoup

html_example = """
<!doctype html>
<html>
  <head><title>Exemplo</title></head>
  <body>
    <div class="card">
      <h2 class="title">Título da Vaga</h2>
      <p class="location">Brasília, DF</p>
      <a href="/detalhes" id="detalhe">ver detalhes</a>
    </div>
  </body>
</html>
"""
soup_demo = BeautifulSoup(html_example, "html.parser")

titulo = soup_demo.select_one("h2.title").get_text(strip=True)
local  = soup_demo.select_one("p.location").get_text(strip=True)
link   = soup_demo.select_one("a#detalhe")["href"]

print("Título:", titulo)
print("Local:", local)
print("Link:", link)
```

-   **HTTP Request/Response:** enviamos um pedido (URL, headers) e recebemos um **status code** (200 = OK, 403/404 = bloqueios/ausência).\
-   **HTML/DOM:** a página é uma árvore (tags, atributos, texto). O BeautifulSoup transforma esse HTML em uma estrutura navegável.\
-   **Encoding:** páginas podem ter encodes distintos; atenção a acentos e `response.encoding`.

# Passo 2: Realizando uma requisição HTTP

```{python}
# URL do site a ser raspado
url = "https://realpython.github.io/fake-jobs/"

# Realiza a requisição para obter o conteúdo HTML da página
response = requests.get(url)

# Cria um objeto BeautifulSoup para análise do HTML
soup = BeautifulSoup(response.content, "html.parser")

# Exibe uma versão formatada dos primeiros 1000 caracteres do HTML
print(soup.prettify()[:1000])
```

# Passo 3: Extraindo informações da página

## Seletores (CSS) e estratégia de extração

-   **Seletores CSS** são diretos e legíveis (`div.card > a.title`, `.classe`, `#id`).\
-   **Estratégia robusta:** prefira seletores **estáveis** (classes/ids que não mudam a cada deploy).\
-   **Validação rápida:** antes de popular um DataFrame, teste `len(soup.select('SELETOR'))` para confirmar que o seletor encontra os elementos esperados.\
-   **Fallback:** se possível, garanta um plano B (ex.: `find()` com condições mais genéricas).

```{python}
# Localiza todos os blocos que contêm informações das vagas
job_elements = soup.find_all('div', class_='card-content')

# Itera sobre cada bloco e imprime os títulos das vagas
for job in job_elements:
    title = job.find('h2', class_='title').get_text().strip()
    print("Título da Vaga:", title)
```

------------------------------------------------------------------------

# Parte 4: Cabeçalho User-Agent e Organização dos Dados com pandas

Nesta parte, é implementado um cabeçalho (User-Agent) requisição HTTP para reduzir o risco de bloqueios pelo servidor.

-   Além disso, os dados extraídos são organizados em um DataFrame utilizando a biblioteca pandas e, por fim, exportados para um arquivo CSV.

### *Vamos estudar isto com **calma e elegância** no momento apropriado. merece uma sessão exclusiva !*

------------------------------------------------------------------------

## Ligação com o ciclo de Recuperação da Informação (RI)

-   **Coleta (scraping):** adquirir o conteúdo bruto (HTML).\
-   **Representação/Indexação:** padronizar campos, normalizar texto (minúsculas, remoção de espaços) e estruturar em **tabela**.\
-   **Busca/Consulta:** filtrar/ranquear os itens coletados por campos (título, data, link).\
-   **Avaliação:** checar **cobertura** (quantos itens esperados foram coletados?), **precisão** (quantos itens são realmente relevantes?) e **atualidade** (dados atualizados?).

## Saída, versionamento e registro

-   **CSV com timestamp**: salvar como `dados_scraping_YYYYMMDD.csv` para rastreabilidade.\
-   **README curto (GitHub Classroom):** descreva alvo, seletor usado, data/hora, contagem de itens e limitações conhecidas.\
-   **Reprodutibilidade:** fixe versões de pacotes no `requirements.txt` (ou registre `pip freeze`).

```{python}
# bibliotecas necessárias
import requests           # Realiza requisições HTTP com cabeçalho customizado
from bs4 import BeautifulSoup  # Faz a análise do conteúdo HTML/XML
import pandas as pd       # Manipulação e organização dos dados em DataFrame

# Definição de um cabeçalho com User-Agent para simular um navegador real
headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) " \
                  "AppleWebKit/537.36 (KHTML, like Gecko) " \
                  "Chrome/98.0.4758.102 Safari/537.36"
}

# Realiza a requisição HTTP com o cabeçalho customizado
url = "https://realpython.github.io/fake-jobs/"
response = requests.get(url, headers=headers)
soup = BeautifulSoup(response.content, "html.parser")

# Encontrando os elementos que contêm as informações das vagas
job_elements = soup.find_all('div', class_='card-content')

# Inicializando listas para armazenar os dados extraídos
titles = []
companies = []
locations = []

# Iterando sobre os elementos para extrair os dados
for job in job_elements:
    title = job.find('h2', class_='title').get_text().strip()
    company = job.find('h3', class_='company').get_text().strip() if job.find('h3', class_='company') else "N/A"
    location = job.find('p', class_='location').get_text().strip() if job.find('p', class_='location') else "N/A"

    titles.append(title)
    companies.append(company)
    locations.append(location)

# Organização dos dados em um DataFrame
df_jobs = pd.DataFrame({
    "Título": titles,
    "Empresa": companies,
    "Localização": locations
})

# Exportação do DataFrame para um arquivo CSV (sem o índice)
df_jobs.to_csv("fake_jobs.csv", index=False)

print("Dados extraídos e salvos no arquivo 'fake_jobs.csv'.")
```

## Robustez mínima (extensões futuras sem alterar o fluxo de hoje)

-   **Tratamento de erro:** `try/except` para requisição e parsing.\
-   **Controle de ritmo:** intervalo aleatório (`time.sleep`) para evitar sobrecarga.\
-   **Retry/backoff exponencial:** em caso de falha, tentar novamente com espera crescente.\
-   **Monitoramento de mudanças:** se o seletor quebrar, registrar a data e o seletor vigente para ajuste rápido.

# Aquela olhadinha

```{python eval=FALSE, include=FALSE}
import pandas as pd
from tabulate import tabulate

df_jobs = pd.read_csv("fake_jobs.csv")
print("Tabela de Vagas Extraídas")
print(tabulate(df_jobs.head(), headers='keys', tablefmt='grid'))

## Tabela formatada (kable)
```

```{r}
# Converte o DataFrame do Python para R (via reticulate) e exibe com kable
library(knitr)
suppressWarnings(suppressMessages({
  if (!requireNamespace("kableExtra", quietly = TRUE)) {
    install.packages("kableExtra")
  }
  library(kableExtra)
}))

# df_jobs foi criado em Python; acessamos como py$df_jobs
df_kable <- try(py$df_jobs, silent = TRUE)

# Caso a sessão tenha sido reiniciada, recarregar do CSV gerado
if (inherits(df_kable, "try-error") || is.null(df_kable)) {
  if (file.exists("fake_jobs.csv")) {
    df_kable <- read.csv("fake_jobs.csv", stringsAsFactors = FALSE)
  } else {
    df_kable <- data.frame(Mensagem = "Tabela não disponível: execute os chunks Python de coleta antes.")
  }
}

# Limitar a 15 linhas para uma visualização mais limpa no HTML
n_show <- min(15, nrow(df_kable))
df_view <- head(df_kable, n_show)

kable(df_view, format = "html", booktabs = TRUE, align = "l",
      caption = paste0("Vagas extraídas (", n_show, " primeiras linhas)")) |>
  kable_styling(full_width = FALSE, bootstrap_options = c("striped", "condensed")) |>
  row_spec(0, bold = TRUE)
```

\`\`\`

## Perguntas para reflexão/avaliação

1.  O seletor escolhido captura **apenas** os itens relevantes? O que faria para reduzir ruído?\
2.  Como verificar **cobertura** (quantos itens esperados vs. coletados)?\
3.  Se o site mudar a classe dos elementos amanhã, qual seria seu **plano de contingência**?\
4.  Há uma **API** disponível? O scraping continua justificável?\
5.  Quais campos coletados são suficientes para análises simples (frequências, séries, correlações)?
