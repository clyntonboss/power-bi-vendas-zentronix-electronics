# Documentação das Medidas: Projeto Faturamento — Zentronix Electronics

Este documento lista todas as medidas criadas no modelo Power BI, suas regras de negócio, dependências e retornos esperados.

---

## Medidas de Faturamento

### faturamento

```DAX
faturamento = 

-- Medida:
--      faturamento
--
-- Descrição:
--      Calcula o valor total de faturamento com base nas vendas registradas.
--
-- Tabela origem:
--      fPedidos
--
-- Regra de negócio:
--      Soma todos os valores da coluna valor_venda respeitando o contexto de filtros aplicado no relatório (datas, produtos, clientes etc.).
--
-- Dependência:
--      Coluna fPedidos[valor_venda]
--
-- Retorno:
--      Valor numérico representando o faturamento total no contexto filtrado.

VAR _Resultado =
    SUM(
        fVendas[faturamento]
    )

RETURN
    _Resultado
```

### maior_menor_faturamento

```DAX
maior_menor_faturamento = 

-- Medida:
--      maior_menor_faturamento
--
-- Descrição:
--      Retorna o valor de faturamento apenas para os meses com maior ou menor faturamento dentro do período selecionado.
--
-- Tabela de apoio:
--      dCalendario
--
-- Regra de negócio:
--      Identifica o maior e o menor faturamento considerando o contexto de seleção do relatório (ALLSELECTED).
--      Caso o mês atual corresponda ao maior ou ao menor faturamento, o valor de faturamento é retornado.
--      Caso contrário, retorna BLANK().
--
-- Dependência:
--      [faturamento]
--
-- Uso típico:
--      Destaque em gráficos de colunas ou linhas para evidenciar os meses de melhor e pior desempenho.

VAR _MenorFaturamento =
    MINX(
        ALLSELECTED(
            dCalendario[mes_abreviado],
            dCalendario[mes_numero]
        ),
        [faturamento]
    )

VAR _MaiorFaturamento =
    MAXX(
        ALLSELECTED(
            dCalendario[mes_abreviado],
            dCalendario[mes_numero]
        ),
        [faturamento]
    )

VAR _FaturamentoAtual =
    [faturamento]

RETURN
    IF(
        _FaturamentoAtual = _MaiorFaturamento ||
        _FaturamentoAtual = _MenorFaturamento,
        _FaturamentoAtual
    )
```

## Medidas de Produtos Vendidos

### produtos_vendidos

```DAX
produtos_vendidos = 

-- Medida:
--      produtos_vendidos
--
-- Descrição:
--      Calcula a quantidade total de produtos vendidos considerando o contexto de filtros aplicado no relatório.
--
-- Tabela origem:
--     fPedidos
--
-- Regra de negócio:
--      Soma os valores da coluna quantidade da tabela de pedidos, representando o total de itens vendidos no contexto filtrado.
--
-- Dependência:
--      fPedidos[quantidade]
--
-- Retorno:
--      Número inteiro representando a quantidade total de produtos vendidos.
--
-- Observação:
--      COALESCE é utilizado para evitar retorno BLANK().

VAR _Resultado =
    SUM(
        fVendas[quantidade]
    )

RETURN
    _Resultado
```

### maior_menor_produtos_vendidos

```DAX
maior_menor_produtos_vendidos = 

-- Medida:
--      maior_menor_produtos_vendidos
--
-- Descrição:
--      Retorna a quantidade de produtos vendidos apenas para os meses com maior ou menor quantidade dentro do período selecionado.
--
-- Tabela de apoio:
--      dCalendario
--
-- Regra de negócio:
--      Identifica a maior e a menor quantidade de produtos vendidos considerando o contexto de seleção do relatório (ALLSELECTED).
--      Caso o mês atual corresponda à maior ou à menor quantidade vendida, o valor da quantidade é retornado.
--      Caso contrário, retorna BLANK().
--
-- Dependência:
--      [produtos_vendidos]
--
-- Uso típico:
--      Destaque em gráficos de colunas ou linhas para evidenciar os meses com maior e menor volume de produtos vendidos.

VAR _MenorQuantidade =
    MINX(
        ALLSELECTED(
            dCalendario[mes_abreviado],
            dCalendario[mes_numero]
        ),
        [produtos_vendidos]
    )

VAR _MaiorQuantidade =
    MAXX(
        ALLSELECTED(
            dCalendario[mes_abreviado],
            dCalendario[mes_numero]
        ),
        [produtos_vendidos]
    )

VAR _QuantidadeAtual =
    [produtos_vendidos]

RETURN
    IF(
        _QuantidadeAtual = _MaiorQuantidade ||
        _QuantidadeAtual = _MenorQuantidade,
        _QuantidadeAtual
    )
```

### percentual_produtos_vendidos_loja

```DAX
percentual_produtos_vendidos_loja = 

-- Medida:
--      percentual_produtos_vendidos_loja
--
-- Descrição:
--      Retorna o percentual de produtos vendidos por loja no período analisado.
--
-- Medida de apoio:
--      quantidade_produtos
--
-- Regra de negócio:
--      Divide a quantidade de produtos vendidos pela quantidade de produtos vendidos removendo filtros.
--
-- Dependência:
--      [quantidade_pedidos]
--
-- Uso típico:
--      Indica o percentual de produtos vendidos por loja.

VAR _Resultado =
    DIVIDE(
        [produtos_vendidos],
        CALCULATE(
            [produtos_vendidos],
            REMOVEFILTERS(
                dLojas
            )
        )
    )

RETURN
    _Resultado
```

## Medidas de Ticket Médio

```DAX
ticket_medio = 

-- Medida:
--      ticket_medio
--
-- Descrição:
--      Calcula o valor médio por pedido (ticket médio).
--
-- Regra de negócio:
--      Divide o faturamento total pela quantidade de pedidos considerando o contexto de filtros do relatório.
--
-- Dependências:
--      [faturamento]
--      [quantidade_pedidos]
--
-- Retorno:
--      Valor monetário representando o ticket médio.
--
-- Observação:
--      COALESCE é utilizado para evitar retorno BLANK().

VAR Resultado =
    DIVIDE(
        [faturamento],
        [transacoes]
    )

RETURN
    COALESCE(
        Resultado,
        0
    )
```

## Medidas de Transações

```DAX
transacoes = 

-- Medida:
--      transacoes
--
-- Descrição:
--      Calcula a quantidade total de transações registradas considerando o contexto de filtros aplicado no relatório
--
-- Tabela origem:
--      fPedidos
--
-- Regra de negócio:
--      Conta o número de registros na tabela de pedidos.
--      Cada linha representa uma transação.
--
-- Dependência:
--      Tabela fPedidos
--
-- Retorno:
--      Número inteiro representando a quantidade de transações no contexto filtrado.
--
-- Observação:
--      COALESCE é utilizado para evitar retorno BLANK().

VAR _Resultado =
    COUNTROWS(
        fVendas
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```

```DAX
maior_menor_transacao = 

-- Medida:
--      maior_menor_vendas
--
-- Descrição:
--      Retorna a quantidade de transações apenas para os meses com maior ou menor volume dentro do período selecionado.
--
-- Tabela de apoio:
--      dCalendario
--
-- Regra de negócio:
--      Identifica a maior e a menor quantidade de transações considerando o contexto de seleção do relatório (ALLSELECTED).
--      Caso o mês atual corresponda à maior ou à menor quantidade de pedidos, o valor é retornado.
--      Caso contrário, retorna BLANK().
--
-- Dependência:
--      [transacoes]
--
-- Uso típico:
--      Destaque em gráficos de colunas ou linhas para evidenciar os meses com maior e menor volume de vendas.

VAR _MenorTransacao =
    MINX(
        ALLSELECTED(
            dCalendario[mes_abreviado],
            dCalendario[mes_numero]
        ),
        [transacoes]
    )

VAR _MaiorTransacao =
    MAXX(
        ALLSELECTED(
            dCalendario[mes_abreviado],
            dCalendario[mes_numero]
        ),
        [transacoes]
    )

VAR _TransacaoAtual =
    [transacoes]

RETURN
    IF(
        _TransacaoAtual = _MaiorTransacao ||
        _TransacaoAtual = _MenorTransacao,
        _TransacaoAtual
    )
```
<br>

```DAX
cores_maior_menor_transacao = 

-- Medida:
--      cores_maior_menor_transacao
--
-- Descrição:
--      Define cores para destacar o maior e o menor volume de transações no período selecionado.
--
-- Tabela de apoio:
--      dCalendario
--
-- Regra de negócio:
--      Identifica, dentro do contexto filtrado (ALLSELECTED), o mês com maior e o mês com menor quantidade de transações.
--      Maior valor → Verde escuro (#33CC33)
--      Menor valor → Vermelho escuro (#FF0000)
--      Demais valores → Cinza (#808080)
--
-- Dependência:
--      [transacoes]
--
-- Uso:
--      Aplicada em formatação condicional de gráficos ou tabelas no Power BI para destacar desempenho mensal.

VAR _MenorTransacao =
    MINX(
        ALLSELECTED(
            dCalendario[mes_abreviado],
            dCalendario[mes_numero]
        ),
        [transacoes]
    )

VAR _MaiorTransacao =
    MAXX(
        ALLSELECTED(
            dCalendario[mes_abreviado],
            dCalendario[mes_numero]
        ),
        [transacoes]
    )

RETURN
    SWITCH(
        TRUE(),
        [transacoes] = _MaiorTransacao, "#33CC33",
        [transacoes] = _MenorTransacao, "#FF0000",
        "#808080"
    )
```
