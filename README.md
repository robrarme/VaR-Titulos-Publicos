# 📊 Cálculo de Value at Risk (V@R) de Títulos Públicos | Power BI

Este projeto consiste em 3 painéis no Power BI focados na análise de risco de mercado (V@R) para Títulos Públicos, aplicando 2 abordagens distintas para mensuração de risco: V@R Paramétrico e V@R Histórico, com parametrização dinâmica do Intervalo de Confiança e do Período de Análise.

## 📌 Sumário

1. [Visão Geral e Telas](#-visão-geral-e-telas)
2. [Fonte dos Dados](#-fonte-dos-dados)
3. [Fundamentação Financeira](#-fundamentação-financeira)
   * [1. Gráfico de Preço Unitário (PU)](#1-preço-unitário-pu-e-retornos-logarítmicos)
   * [2. V@R Paramétrico](#2-vr-paramétrico-variância-covariância)
   * [3. V@R Histórico](#3-vr-histórico)
4. [Arquitetura de Dados & Modelagem](#-arquitetura-de-dados--modelagem)
5. [Implementação em DAX](#-implementação-em-dax)
   * [Parametrização do Fator-Z](#-factor-z-dinâmico)
   * [Cálculo do V@R Paramétrico](#-vr-paramétrico)
   * [Cálculo do V@R Histórico](#-vr-histórico)
6. [Como Executar o Projeto](#-como-executar-o-projeto)

## 🖼 Visão Geral e Telas

O relatório foi estruturado em três telas interativas, desenhadas para priorizar a clareza e a facilidade de navegação:

1. **Evolução do Preço Unitário (PU):** Acompanhamento temporal da curva de preços de um título público.
   [Tela 1 - Gráfico PU](./assets/pu_chart.png)

2. **V@R Paramétrico:** Cálculo da perda máxima esperada considerando a volatilidade e a média da série histórica para o intervalo de confiança selecionado.
   [Tela 2 - V@R Paramétrico](./assets/var_parametrico.png)

3. **V@R Histórico:** Análise não paramétrica baseada no percentil real da distribuição dos retornos observados num determinado período.
   [Tela 3 - V@R Histórico](./assets/var_historico.png)

## 🌐 Fonte dos Dados

Os dados históricos de preços e taxas dos títulos públicos utilizados neste projeto são públicos e foram obtidos diretamente da plataforma de Dados Abertos da Secretaria do Tesouro Nacional:

* **Fonte:** [Tesouro Transparente](https://www.tesourotransparente.gov.br/ckan/dataset/taxas-dos-titulos-ofertados-pelo-tesouro-direto)
* **Ativos Analisados:** Títulos públicos federais (Tesouro IPCA+, Tesouro Prefixado e Tesouro Selic).

## 📐 Fundamentação Financeira

### 1. Preço Unitário (PU) e Retornos Logarítmicos

A variação diária dos títulos públicos foi mensurada através do retorno contínuo (log-retorno):

$$
R_t = \ln\left(\frac{PU_t}{PU_{t-1}}\right)
$$

### 2. V@R Paramétrico

A abordagem paramétrica assume que os retornos dos ativos seguem uma distribuição Normal Padrão. O ponto de corte do V@R é mensurado ajustando o desvio padrão pela média da distribuição:

$$
\text{V@R}_{\alpha} = \text{Média} - (Z_{\alpha} \times \sigma)
$$

Onde:
* $\text{Média}$: Retorno médio diário do título no período filtrado.
* $\sigma$: Desvio padrão (volatilidade diária) dos retornos no período filtrado.
* $Z_{\alpha}$: Z-score correspondente ao nível de confiança selecionado $\alpha$ (ex: $Z_{0.95} \approx 1.645$, $Z_{0.99} \approx 2.326$).

### 3. V@R Histórico

A abordagem histórica ordena os retornos históricos reais do período e extrai o percentil correspondente ao nível de probabilidade de perda ($1 - \alpha$):

$$
\text{V@R}_{\text{Histórico}, \alpha} = \text{Percentil}\left(R, 1 - \alpha\right)
$$

## 🛠 Arquitetura de Dados & Modelagem

* **Origem dos Dados:** Extração e ETL automatizado via Power Query a partir de planilha disponível no Tesouro Transparente.
* **Modelo de Dados:** Originalmente uma só tabela, porém normalizada e transformada para o Esquema Estrela (*Star Schema*).
* **Tabelas de Parâmetros:** Criação de parâmetro *What-If* para a seleção do Intervalo de Confiança ($90\%$, $95\%$, $99\%$).

## 💻 Implementação em DAX

Abaixo estão os blocos de código DAX principais que sustentam a inteligência quantitativa do painel:

### 📈 V@R Paramétrico

Calcula o percentual do V@R diário subtraindo o produto do Z-score pela volatilidade ($\sigma$) a partir da média dos retornos:

```dax
Linha_VaR_95 = 
[Media_Histograma] - (NORM.S.INV('IntervaloConfiança'[ValorIntervaloConfiança]) * [Desvio_Padrao])
```

### 📉 V@R Histórico

Obtém o limite de perda na cauda esquerda da distribuição dos retornos observados com base no percentil ($1 - \text{Confiança}$):

```dax
Linha_VaR_95_Historico = 
PERCENTILE.INC(fPrecoTaxa[VariaçãoDiária], 1 - 'IntervaloConfiança'[ValorIntervaloConfiança])
```

## 🚀 Como Executar o Projeto

1. Clone este repositório:
   ```bash
   git clone https://github.com/seu-usuario/var-titulos-publicos-powerbi.git
   ```
2. Abra o arquivo `.pbix` localizado na pasta *assets* utilizando o **Power BI Desktop**.
3. Use os filtros laterais para navegar entre os Títulos Públicos (ex: Tesouro IPCA+, Prefixado), alterar o Período de Análise e simular o Nível de Confiança desejado.

---

### 👤 Autor

* **Ronnie B. A. Meneses** - [LinkedIn](https://www.linkedin.com/in/ronnie-meneses/) | [GitHub](https://github.com/robrarme)
