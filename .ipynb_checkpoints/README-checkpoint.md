# Sistema de Avaliação de Imóveis (RBC)
[![Python](https://img.shields.io/badge/Python-3.10%2B-blue?logo=python&logoColor=yellow)](https://www.python.org/)
[![Pandas](https://img.shields.io/badge/Pandas-1.5%2B-blue?logo=pandas)](https://pandas.pydata.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Lab-orange?logo=jupyter)](https://jupyter.org/)

Este repositório contém a implementação de um sistema de **Raciocínio Baseado em Casos (RBC)** para o domínio de avaliação de preços de imóveis.

O projeto foi desenvolvido em um Jupyter Notebook (`ciclo_RBC.ipynb`) e utiliza a biblioteca Pandas para manipulação da base de casos.

## Contexto do Projeto

Este trabalho foi desenvolvido para a disciplina de Raciocínio Baseado em Casos, ministrada pelo Prof. Dr. André Pinz Borges, da Universidade Tecnológica Federal do Paraná (UTFPR).

O objetivo principal é implementar o **ciclo completo de RBC** (Recuperar, Reusar, Revisar, Reter) em um domínio de livre escolha, cumprindo todos os requisitos listados na definição do trabalho.

## O Ciclo RBC no Projeto

O sistema implementa o ciclo completo de RBC conforme descrito nas aulas.

### 1. A Base de Casos

A "Base de Casos" (`Case Library`) é armazenada no arquivo `base_de_casos.csv` e gerenciada com Pandas. Cada caso (imóvel vendido) é definido por:

* **Problema:** Um conjunto de 6 atributos que descrevem o imóvel:
    1.  `metros_quadrados` (Numérico)
    2.  `num_quartos` (Numérico)
    3.  `num_banheiros` (Numérico)
    4.  `vagas_garagem` (Numérico)
    5.  `bairro` (Simbólico Não-Ordenado)
    6.  `mobiliado` (Simbólico Binário)
* **Solução:**
    1.  `preco_vendido` (Numérico)

### 2. Etapa 1: RECUPERAÇÃO (Retrieve)

O objetivo desta etapa é encontrar o caso mais similar (o "vizinho mais próximo" ou k-NN com k=1) ao novo problema inserido pelo usuário.

* **Similaridade Global:** É calculada como uma **média ponderada** das similaridades locais, com pesos definidos no notebook.
* **Similaridade Local (Numérica):** Para atributos como `metros_quadrados`, o sistema usa **Normalização Min-Max** (`(valor - min) / (max - min)`) para colocar todos os valores na escala [0, 1]. A similaridade é então `1 - abs(diferença_normalizada)`.
* **Similaridade Local (Simbólica):**
    * Para `bairro`, é usada uma **Tabela de Similaridade** (`TABELA_SIMILARIDADE_BAIRRO`) que define o quão parecidos os bairros são entre si.
    * Para `mobiliado`, a similaridade é `1.0` se os valores forem iguais e `0.0` se forem diferentes.
* **Implementação:** A função `recuperar_caso_mais_similar` usa `pandas.apply()` para executar a `calcular_similaridade_global` em todos os casos e `idxmax()` para encontrar o melhor.

### 3. Etapa 2: REUSO (Reuse)

Esta etapa adapta a solução do caso recuperado. Foi implementada uma **Adaptação Transformacional Substitucional** baseada em regras:

1.  **Cálculo Base:** O sistema calcula o `preço/m²` do caso antigo.
2.  **Adaptação Principal:** Esse `preço/m²` é aplicado aos `metros_quadrados` do novo problema para gerar um `preco_base_adaptado`.
3.  **Regra de Ajuste:** O sistema então verifica se há diferença no atributo `mobiliado`. Se o novo caso for mobiliado e o antigo não, ele aplica um bônus (ex: +10%). Se for o inverso, aplica uma penalidade (ex: -10%).

### 4. Etapa 3: REVISÃO (Revise)

Esta etapa avalia a solução gerada pelo Reuso. Conforme sugerido nas aulas, a revisão é feita por **"Interação com usuário"**.

1.  O sistema apresenta o `Preço Sugerido (Adaptado)`.
2.  O usuário informa se a sugestão foi precisa (`s/n`).
3.  Se `n` (incorreta), o sistema solicita ao usuário o "valor reparado". Este valor será usado na etapa de Retenção.

### 5. Etapa 4: RETENÇÃO (Retain)

É o "processo de incorporação" do novo conhecimento. O sistema aprende com a nova experiência para melhorar seu desempenho futuro.

1.  A função `reter_novo_caso` pega o novo problema e a solução (confirmada ou reparada da Etapa 3).
2.  Ela adiciona essa nova entrada como uma nova linha ao DataFrame do Pandas.
3.  O arquivo `base_de_casos.csv` é **sobrescrito** com a nova base de casos (agora com 16+ casos).
4.  O sistema **recalcula** os `ranges_normalizacao` (Min-Max) para incluir o novo caso nos cálculos futuros.

---

## Como Executar o Projeto

Este projeto é um Jupyter Notebook e deve ser executado em um ambiente que o suporte (como Jupyter Lab, VS Code, etc.).

### Pré-requisitos

Certifique-se de ter Python e as seguintes bibliotecas instaladas:
* `pandas`
* `jupyter`
* `notebook`

```bash
pip install pandas jupyter notebook
```

### Instruções

1.  Clone este repositório:
    ```bash
    git clone https://github.com/artrosisca/avaliacao-imoveis-CBR.git
    cd avaliacao-imoveis-CBR
    ```
2.  Inicie o Jupyter Notebook:
    ```bash
    jupyter notebook
    ```
3.  Abra o arquivo `ciclo_RBC.ipynb` (o nome do seu notebook).
4.  Se você ainda não o fez, **execute a célula de "CONVERSÃO"** (Célula 2) **apenas uma vez**. Isso irá criar o `base_de_casos.csv` a partir do `.json`. Você pode apagar esta célula depois.
5.  **Execute todas as outras células** em ordem (Menu "Kernel" -> "Restart & Run All") para carregar as funções e a base de dados.
6.  **Execute a última célula ("CICLO RBC COMPLETO")** para iniciar o processo. O sistema fará as perguntas no próprio notebook.

---