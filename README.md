# Desafio_Star_Schema_Financials
# Desafio de Projeto: Modelagem e Transformação de Dados com DAX no Power BI

Este repositório contém a solução do desafio prático de modelagem dimensional **Star Schema (Esquema em Estrela)** utilizando o conjunto de dados *Financial Sample* no Power BI.

## 📌 Objetivo
Transformar uma tabela única e plana (`financials`) numa estrutura relacional em estrela, separando os dados em tabelas dimensão e uma tabela fato para otimizar a performance e a organização das análises.

## 🛠️ Etapas de Desenvolvimento

### 1. Preparação da Tabela Origem (`Power Query`)
- Renomeação da tabela principal para `financials_origem` e desabilitação da opção **Habilitar Carga** (backup oculto).
- Criação do índice condicional `ID_Produto` para mapeamento numérico dos produtos.
- Criação da chave primária `SK_ID` (coluna de índice de 0 a N).

### 2. Construção do Star Schema
A partir da tabela origem, foram criadas as seguintes consultas por duplicado/referência:
- **`D_Produtos`**: Tabela agregada por produto com métricas calculadas (médias de vendas e manufatura, valores mínimo, máximo e mediana).
- **`D_Produtos_Detalhes`**: Detalhes de preços de venda, preços de fabricação e unidades vendidas.
- **`D_Descontos`**: Mapeamento de faixas e valores de desconto por produto.
- **`D_Detalhes`**: Informações financeiras complementares (COGS, Gross Sales, Sales).
- **`F_Vendas`**: Tabela fato central contendo as métricas principais e as chaves estrangeiras (`ID_Produto`, `SK_ID`, `Date`).

### 3. Tabela Temporal com DAX
Criação da tabela dimensão de calendário `D_Calendário` através da seguinte fórmula DAX:

```dax
D_Calendário = 
VAR DataMinima = MIN(F_Vendas[Date])
VAR DataMaxima = MAX(F_Vendas[Date])
RETURN
ADDCOLUMNS (
    CALENDAR(DataMinima, DataMaxima),
    "Ano", YEAR([Date]),
    "Mês Num", MONTH([Date]),
    "Nome do Mês", FORMAT([Date], "mmmm"),
    "Trimestre", "T" & FORMAT([Date], "q"),
    "Semestre", IF(MONTH([Date]) <= 6, "S1", "S2"),
    "Dia da Semana", FORMAT([Date], "dddd")
)
