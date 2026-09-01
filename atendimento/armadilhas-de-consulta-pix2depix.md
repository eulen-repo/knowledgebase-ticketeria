# Armadilhas de consulta na base do pix2depix

As consultas desta base falham de um jeito específico: em vez de dar erro, elas devolvem menos
linha. Um filtro de prefixo que passou a excluir metade da semana, uma data que parece ser a do
pagamento e é outra coisa, uma coluna de atualização usada como se fosse de criação. Nenhuma
dessas dá mensagem nenhuma, e o resultado tem cara de resposta.

Este documento reúne o que já enganou, com o teste que separa a leitura certa da errada.

## O prefixo do identificador é um relógio

Os identificadores são UUID versão 7, cujos primeiros bits são o instante de criação em
milissegundos. O prefixo, portanto, muda com o tempo sozinho. Ele virou de `019` para `01a` em
14/08/2026, e volta a virar, para `01b`, em 17/10/2028.

Isso quebra qualquer filtro escrito como `LIKE '019%'`, e quebra em silêncio. Depois da virada,
um recorte por `LIKE '019%'` passou a esconder todas as linhas de prefixo `01a` — quem tivesse
esse recorte numa consulta salva viu o volume cair sem que nada tivesse caído.

A leitura correta usa o carimbo dentro do próprio identificador em vez do texto dele. Os doze
primeiros dígitos hexadecimais são os milissegundos desde a época Unix:

```sql
to_timestamp(
  (('x' || substring(replace(id::text, '-', ''), 1, 12))::bit(48)::bigint) / 1000.0
) AS criado_em
```

## `last_update` não é data de criação

A coluna é reescrita por eventos posteriores. O caminho de estorno grava `last_update = NOW()`
ao marcar o QR como devolvido, e o carimbador de expiração faz o mesmo ao vencer um QR.

O tamanho do desvio surpreende: parte das linhas com `last_update` num mês tinha sido criada mais
de uma hora antes, e a maior defasagem entre criação e última atualização passava de 70 dias. Uma
série temporal montada sobre `last_update` conta esses QR no mês errado. Para "quando isto foi
criado", o campo certo é o identificador, pela expressão da seção anterior.

## `tx_date` é declarada por quem entregou o webhook

O daemon preenche `tx_date` com a data de transferência que a **processadora** mandou no payload
e, quando esse campo chega vazio, cai para o instante em que a linha foi processada, deixando um
aviso no log. Nos dois casos a data vem de fora, e não de uma medição nossa.

A consequência é que `tx_date` não responde "quando o pagador pagou". Ela responde "que data veio
no webhook", e quando uma processadora libera de uma vez um lote que estava retido, as linhas
entram carimbadas com o que ela declarar naquele momento.

## O carimbo dentro do E2E está em UTC

O identificador ponta a ponta do Pix, o E2E, carrega a instituição e o instante do pagamento em
posições fixas: a partir do segundo caractere vêm os oito dígitos do ISPB, o identificador da
instituição no Sistema de Pagamentos Brasileiro, e a partir do décimo vêm doze dígitos de data e
hora no formato `AAAAMMDDHHMM`.

Esse carimbo está em UTC, e essa é a parte que engana, porque a base renderiza `tx_date` em BRT.
A comparação dia a dia em UTC casa com praticamente todos os depósitos; comparar com `tx_date` em
BRT quase não casa. Para investigar um caso pelo horário real, o E2E é a fonte, e `tx_date` é a
data da linha.

```sql
substring(central_bank_id, 2, 8)   AS ispb_do_pagador,
substring(central_bank_id, 10, 12) AS instante_utc
```

## Identificadores compostos usam dois separadores

O `bank_tx_identifier` traz a rail no começo, mas nem toda rail usa o mesmo separador: as linhas
de estorno automático usam hífen onde as demais usam sublinhado. Quebrar só por `_` devolve uma
rail distinta por linha de estorno, o que infla a contagem de categorias e some com o
agrupamento. As duas passagens resolvem:

```sql
split_part(split_part(bank_tx_identifier, '_', 1), '-', 1) AS rail
```

## O que liga um depósito ao QR

A junção é `generated_pix_qr_code.id = bank_tx.qr_id`. A tabela de QR não tem coluna `qr_id`: o
único campo dela com `qr` no nome é `qr_code`, que guarda o payload. Juntar por `q.qr_id` é o raro
erro desta lista que o banco reporta em vez de engolir.

Uma linha de `bank_tx` com `qr_id` nulo é um pagamento que não passou por QR nosso, isto é, uma
chave estática. Quem for daí ao dono do dinheiro precisa da regra descrita em
[Depósito em chave estática](deposito-em-chave-estatica.md), porque nesse caso o dono não vem do
QR.

## O schema não é `public`

Na réplica de leitura, as tabelas vivem no schema `pix2depix`. `bank_tx`, `generated_pix_qr_code`
e `sending_depix` aparecem nesse schema e em nenhum outro. Consulta sem prefixo depende do
`search_path` da sessão, que no Metabase não é o que se espera.

## Relacionado

- [Depósito em chave estática](deposito-em-chave-estatica.md) — o que fazer quando `qr_id` é nulo.
- [Armadilhas do Metabase](../observabilidade/metabase-armadilhas.md) — as falhas do outro lado,
  quando a consulta está certa e o alerta é que parou de avisar.
