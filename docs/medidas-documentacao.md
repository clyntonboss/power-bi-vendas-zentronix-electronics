# Documentação das Medidas: Projeto Faturamento — Zentronix Electronics

Este documento descreve as medidas criadas no modelo semântico do Power
BI relacionadas à tabela **fPedidos**.

Todas as medidas seguem o padrão de desenvolvimento adotado no projeto:

-   Uso de `VAR` para armazenar resultados intermediários
-   Uso de `COALESCE()` para evitar retorno `BLANK()`
-   Uso de `DIVIDE()` para evitar erro de divisão por zero
-   Documentação clara da regra de negócio
-   Estrutura padronizada para facilitar manutenção e escalabilidade

------------------------------------------------------------------------

# Medidas da Fato Pedidos

## faturamento

**Descrição**

Faturamento total gerado pelos pedidos registrados no modelo.

**Tabela origem**

`fPedidos`

**Regra de negócio**

Soma os valores da coluna `valor_venda` da tabela `fPedidos`,
representando a receita total das vendas.

**Retorno**

Valor numérico representando o faturamento total.

``` dax
faturamento =
VAR Resultado =
    SUM(
        fPedidos[valor_venda]
    )
RETURN
    COALESCE(Resultado, 0)
```

------------------------------------------------------------------------

## quantidade_pedidos

**Descrição**

Quantidade total de pedidos registrados no modelo.

**Tabela origem**

`fPedidos`

**Regra de negócio**

Conta todas as linhas da tabela `fPedidos`, onde cada linha representa
um pedido registrado no sistema.

**Retorno**

Número inteiro representando a quantidade total de pedidos.

``` dax
quantidade_pedidos =
VAR Resultado =
    COUNTROWS(
        fPedidos
    )

RETURN
    COALESCE(
        Resultado,
        0
    )
```

------------------------------------------------------------------------

## quantidade_produtos

**Descrição**

Quantidade total de produtos vendidos.

**Tabela origem**

`fPedidos`

**Regra de negócio**

Soma os valores da coluna `quantidade_produto` da tabela `fPedidos`,
representando o total de itens vendidos.

**Retorno**

Número inteiro representando a quantidade total de produtos vendidos.

``` dax
quantidade_produtos =
VAR Resultado =
    SUM(
        fPedidos[quantidade_produto]
    )

RETURN
    COALESCE(
        Resultado,
        0
    )
```

------------------------------------------------------------------------

## ticket_medio

**Descrição**

Valor médio gasto por pedido.

**Regra de negócio**

Divide o faturamento total pela quantidade total de pedidos,
representando o valor médio de cada pedido.

**Dependências**

-   `[faturamento]`
-   `[quantidade_pedidos]`

**Retorno**

Valor monetário representando o ticket médio.

``` dax
ticket_medio =
VAR Resultado =
    DIVIDE(
        [faturamento],
        [quantidade_pedidos]
    )

RETURN
    COALESCE(
        Resultado,
        0
    )
```

------------------------------------------------------------------------

# Medidas de Destaque de Desempenho

Estas medidas são utilizadas para **identificar extremos de desempenho
(maior e menor valor)** dentro do período selecionado no relatório.

Elas normalmente são utilizadas em **gráficos de linha ou coluna** para
destacar visualmente os meses de melhor e pior resultado.

A lógica considera sempre o contexto filtrado utilizando
**`ALLSELECTED()`**, respeitando seleções feitas pelo usuário no
relatório.

------------------------------------------------------------------------

## maior_menor_faturamento

``` dax
maior_menor_faturamento =

VAR MenorFaturamento =
    MINX(
        ALLSELECTED(
            dCalendario[mes_abreviado],
            dCalendario[mes_numero]
        ),
        [faturamento]
    )

VAR MaiorFaturamento =
    MAXX(
        ALLSELECTED(
            dCalendario[mes_abreviado],
            dCalendario[mes_numero]
        ),
        [faturamento]
    )

VAR FaturamentoAtual =
    [faturamento]

RETURN
    IF(
        FaturamentoAtual = MaiorFaturamento ||
        FaturamentoAtual = MenorFaturamento,
        FaturamentoAtual
    )
```

------------------------------------------------------------------------

## maior_menor_vendas

``` dax
maior_menor_vendas =

VAR MenorVenda =
    MINX(
        ALLSELECTED(
            dCalendario[mes_abreviado],
            dCalendario[mes_numero]
        ),
        [quantidade_pedidos]
    )

VAR MaiorVenda =
    MAXX(
        ALLSELECTED(
            dCalendario[mes_abreviado],
            dCalendario[mes_numero]
        ),
        [quantidade_pedidos]
    )

VAR VendaAtual =
    [quantidade_pedidos]

RETURN
    IF(
        VendaAtual = MaiorVenda ||
        VendaAtual = MenorVenda,
        VendaAtual
    )
```

------------------------------------------------------------------------

## maior_menor_quantidade_produtos

``` dax
maior_menor_quantidade_produtos =

VAR MenorQuantidade =
    MINX(
        ALLSELECTED(
            dCalendario[mes_abreviado],
            dCalendario[mes_numero]
        ),
        [quantidade_produtos]
    )

VAR MaiorQuantidade =
    MAXX(
        ALLSELECTED(
            dCalendario[mes_abreviado],
            dCalendario[mes_numero]
        ),
        [quantidade_produtos]
    )

VAR QuantidadeAtual =
    [quantidade_produtos]

RETURN
    IF(
        QuantidadeAtual = MaiorQuantidade ||
        QuantidadeAtual = MenorQuantidade,
        QuantidadeAtual
    )
```

------------------------------------------------------------------------

## cores_maior_menor_vendas

``` dax
cores_maior_menor_vendas = 

VAR MenorVenda =
    MINX(
        ALLSELECTED(
            dCalendario[mes_abreviado],
            dCalendario[mes_numero]
        ),
        [quantidade_pedidos]
    )

VAR MaiorVenda =
    MAXX(
        ALLSELECTED(
            dCalendario[mes_abreviado],
            dCalendario[mes_numero]
        ),
        [quantidade_pedidos]
    )

RETURN
    SWITCH(
        TRUE(),
        [quantidade_pedidos] = MaiorVenda, "#33CC33",
        [quantidade_pedidos] = MenorVenda, "#FF0000",
        "#808080"
    )
```

------------------------------------------------------------------------

# Padrão adotado no projeto

-   Clareza na regra de negócio
-   Padronização de código
-   Uso de variáveis (`VAR`) para melhorar legibilidade
-   Tratamento de valores nulos com `COALESCE`
-   Uso de funções seguras como `DIVIDE`
-   Facilidade de manutenção e evolução do modelo

Esse padrão garante maior **qualidade técnica**, **legibilidade do
modelo semântico** e **facilidade de colaboração em ambientes
versionados**, como projetos mantidos no GitHub.
