# Armadilhas do Metabase ao rodar consulta

O [mapa do Metabase](./metabase-mapa.md) diz onde cada número mora e os
[alertas](./metabase-alertas.md) cobrem a camada de notificação, incluindo a condição de meta que
dispara na travessia e depois cala. Falta a camada do meio: o que acontece entre escrever o SQL e
ler o resultado. As armadilhas abaixo têm em comum devolverem uma resposta plausível em vez
de um erro, o que faz o número errado atravessar a revisão.

Tudo abaixo foi medido contra a instância de produção em 29/08/2026, pelo `/api/dataset` no
banco de leitura do `pix2depixd`.

## O teto de 2.000 linhas

Uma consulta nativa disparada pela API volta cortada em 2.000 linhas, e o corte independe de a
consulta ser agregada ou não:

| Consulta | Linhas pedidas | Linhas devolvidas |
|---|---|---|
| `SELECT g FROM generate_series(1, 5000) g` | 5.000 | 2.000 |
| `SELECT g FROM generate_series(1, 20000) g` | 20.000 | 2.000 |
| a mesma com `GROUP BY 1` | 5.000 | 2.000 |
| a mesma com `GROUP BY 1` | 20.000 | 2.000 |

O corte vem do middleware `add-default-userland-constraints?`, que a própria resposta declara em
`json_query.middleware`. O Metabase avisa: o bloco `data` traz `rows_truncated: 2000` junto com as
linhas. Quem consome a resposta por um cliente que só imprime `data.rows` nunca vê esse aviso, e é
aí que o teto passa despercebido.

A consequência prática é que qualquer contagem feita do lado de fora, somando as linhas devolvidas,
mente para cima de 2.000. Agregar no SQL resolve, porque uma contagem em uma linha nunca é cortada;
paginar por faixa de data resolve quando o detalhe é necessário. Conferir `rows_truncated` antes de
usar o resultado é o que separa as duas situações.

## `to_timestamp` já devolve o horário de Brasília

A instância roda com fuso de sessão `America/Sao_Paulo`, e `to_timestamp` devolve
`timestamp with time zone`. Conferido na mesma execução: `current_setting('TimeZone')` responde
`America/Sao_Paulo`, e `to_timestamp(0)` renderiza `1969-12-31T21:00:00-03:00`, que é o epoch já
convertido.

Isso importa porque a extração de data de um identificador UUIDv7 passa por `to_timestamp`, como
descreve o [documento de armadilhas de consulta](./armadilhas-de-consulta-pix2depix.md). O reflexo
de "o banco está em UTC, subtraio três horas" produz um carimbo três horas atrás do real, e o erro
é constante, então nada na saída parece estranho. A conferência barata é comparar o resultado com
`now()` na mesma consulta.

## A resposta de sucesso é 202, não 200

`POST /api/dataset` responde `202 Accepted` mesmo quando a consulta terminou e as linhas vieram no
corpo. Cliente que trata sucesso como `status == 200` descarta um resultado bom e reporta falha de
conexão. O tratamento correto aceita as duas faixas, que é o que o `metabase_query.py` do
`eulen-operations` faz.

Erro de SQL também volta com corpo, e não com status de erro: a mensagem chega em `error` dentro
de `data`, ou na raiz da resposta. Um cliente que só olha o status conclui que a consulta passou e
segue com zero linha.

## Relacionado

- [Mapa do Metabase](./metabase-mapa.md) — onde cada dashboard e cada card vivem.
- [Alertas do Metabase](./metabase-alertas.md) — a camada de notificação, com a condição de meta
  que dispara uma vez e depois fica muda.
- [Armadilhas de consulta na base do pix2depix](./armadilhas-de-consulta-pix2depix.md) — as
  armadilhas do banco, do outro lado da mesma consulta.
