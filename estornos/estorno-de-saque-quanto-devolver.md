# Estorno de saque: quanto devolver, e quando não devolver

Duas situações chegam com a mesma frase no grupo do parceiro, "o saque não foi", e pedem números
diferentes. Numa delas o PIX nunca saiu da conta e o parceiro recebe de volta o DePix cheio que
mandou. Na outra o PIX saiu, o recebedor devolveu, e o que volta é o valor em reais do saque, com a
taxa retida. Confundir as duas devolve dinheiro a mais ou a menos, e nenhum dos dois erros aparece
sozinho depois.

Este documento separa os dois casos, mostra por que só um deles tem comando, e registra a
conferência que o portão de estado do daemon não faz por você. O caminho inverso, quando é o
parceiro que devolve à Eulen num MED, fica em
[proc-devolucao-med.md §7.4](../compliance/brazil-pt/processos/proc-devolucao-med.md#74-quando-o-parceiro-devolve-o-depix).

Lido em `origin/main` do `pix2depixd` e medido no db2 em 29/08/2026.

## Quando o PIX nunca saiu: o DePix bruto que chegou

Esse é o caso que o `/refundwithdraw` resolve, e o valor não é escolhido por ninguém. O comando
devolve o DePix que efetivamente entrou para aquele saque, lido de
`withdraw.exact_amount_received_in_smallest_unit` (`src/pkg/banktx/withdraw_refund.go:461-500`).
Não houve prestação de serviço, então não há taxa a reter.

O comentário dessa função guarda a armadilha que justifica a leitura direta. O campo equivalente
na estrutura em memória, `WithdrawInfo.AmountReceivedInSmallestUnit`, cai para a coluna de valor
esperado quando a recebida é nula, de modo que um saque que não recebeu nada ainda assim reporta
um valor diferente de zero. Estornar por ele mandaria DePix que nunca chegou. A leitura é feita
uma vez, dentro da função que move o dinheiro, para que nenhum caminho novo possa errar de outro
jeito.

O comando aceita o endereço como argumento opcional. Omitido, a devolução vai para o
`refund_address` que o parceiro declarou no próprio pedido de saque, que é o caso comum
(`src/pkg/botcmd/cmdprivate/refundwithdraw.go:29-38`).

## O portão de estado, e o que ele recusa mesmo com `force`

O estado do saque decide a elegibilidade antes de qualquer envio
(`src/pkg/banktx/withdraw_refund.go:430-457`).

| Estado | Valor | O que acontece |
|---|---|---|
| `Unsent` | 0 | elegível só com `force` ou por gatilho de retenção |
| `Sending` | 1 | recusado sempre |
| `Sent` | 2 | recusado sempre |
| `Error` | 3 | elegível |
| `Canceled` | 4 | elegível |
| `Replaced` | 5 | recusado sempre |
| `Refunded` | 6 | recusado, já devolvido |

As três recusas incondicionais têm o mesmo motivo, e nenhum argumento as contorna. Em `Sending` e
`Sent` os reais saíram, ou estão saindo, e devolver o DePix por cima é perda direta. Em `Replaced`
existe um saque mais novo que substituiu aquele, criado por `/retrywithdraw`, e ele pode ter sido
pago: devolver o depósito do original perde o dinheiro duas vezes.

## Quando o PIX saiu e voltou: o valor em reais, taxa retida

Essa situação não tem estado próprio. O saque continua em `Sent`, com o dinheiro de volta na nossa
conta e o DePix do parceiro já queimado. Como `Sent` é recusa incondicional, o `/refundwithdraw`
não se aplica, e a devolução é decidida e executada à mão.

O valor devido é o valor em reais do saque, não o DePix que entrou. A regra foi decidida em
18/08/2026 e o fundamento é que houve prestação de serviço dos dois lados, da Eulen e do parceiro,
então a taxa fica retida. Num caso daquela semana, um saque com pouco mais de 506 DePix recebidos
e R$ 501,47 pagos e devolvidos pelo recebedor no mesmo dia gerou devolução de 501,47, e os pouco
mais de 5 de taxa permaneceram.

Diferente do fluxo de MED, aqui a mensagem ao parceiro pode carregar uma frase de valor, porque
ela explica o número em vez de negociar a causa. Uma frase basta, colada ao valor, dizendo que
houve prestação de serviço dos dois lados e que a devolução sai com a taxa descontada.

## A conferência que o portão não faz

O portão de estado protege contra devolver um saque que saiu. Ele não protege contra devolver um
DePix que já saiu por outro saque.

Vários saques podem compartilhar o mesmo `receiving_txid`, e nesse arranjo o portão libera os
irmãos cancelados enquanto o pago fica de fora. Devolver qualquer um dos cancelados manda de volta
um DePix que já virou reais. Medido em 29/08/2026 sobre toda a história da tabela `withdraw`, isso
aconteceu uma única vez: um grupo de quatro saques, todos em 08/05/2026, um deles em `Sent` e três
em `Canceled`. É raro, e é barato conferir antes de um lote:

```sql
SELECT receiving_txid, count(*) AS saques
  FROM pix2depix.withdraw
 WHERE receiving_txid IS NOT NULL AND receiving_txid <> ''
 GROUP BY 1
HAVING count(*) > 1
```

Qualquer linha devolvida por essa consulta exige olhar os estados do grupo antes de estornar
qualquer um deles.

## Relacionado

- [Estorno de depósito](./estorno-de-deposito.md) — o outro lado do fluxo, com o modelo de dados e
  como provar que uma devolução aconteceu.
- [proc-devolucao-med.md](../compliance/brazil-pt/processos/proc-devolucao-med.md) — o sentido
  inverso, em que o parceiro devolve à Eulen, incluindo por que a resposta não carrega frase de
  valor nenhuma.
- [Armadilhas de consulta](./armadilhas-de-consulta-pix2depix.md) — as demais leituras desta base
  que devolvem menos linha em vez de erro.
- [Manual operacional do bot Pix](./manual-operacional-bot-pix.md) — os comandos em volta, do
  cancelamento à consulta de estado.
