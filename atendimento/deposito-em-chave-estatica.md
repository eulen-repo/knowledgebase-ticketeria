# Depósito em chave estática: quem é o dono do dinheiro

Um depósito em QR dinâmico traz o dono embutido, porque o QR foi gerado por nós e a linha do banco
aponta para ele. Um pagamento em chave estática não traz nada disso. O dono é decidido por uma
única string que vem de fora, dentro do webhook da **processadora**, e vários incidentes se apoiam
inteiramente nesse fato: um pagamento que ninguém recebeu, um creditado ao parceiro errado e uma
leva que falhou na ingestão.

Este documento diz como a atribuição funciona, quais bordas ela não cobre e como conferir um caso
sem se enganar.

## A atribuição existe em uma rail só

A chave estática é recurso de **uma processadora específica** por construção. A resolução do nó
exige uma conta nessa processadora com chave Pix configurada, seja pela preferência do parceiro,
seja pelo padrão do sistema, e falha com `ErrNoStaticPixBankingNode` quando nenhuma das duas
resolve.

Do lado do recebimento, o webhook da processadora só reconhece um pagamento como depósito em chave
estática quando três coisas coincidem: o método é `BANK_TRANSFER`, o tipo de objeto é
`BankTransferPayment`, e `data.metadata["code"]` traz o identificador do parceiro. Sem o `code` não
há como resolver quem deve receber o crédito, e o payload deixa de ser um depósito estático válido.
Um payload que não passa nesse teste é registrado como inválido e ignorado.

O valor do campo é aparado nas bordas antes de virar identificador, e um `code` ausente, não
textual ou só com espaços equivale a não ter `code` nenhum. Quando o `code` existe mas aponta para
um parceiro desconhecido, a resolução devolve vazio sem erro, e a transação vai para modo manual
com o cliente marcado como `unknown`.

A consequência prática para medição é que um pagamento estático que chegue por outra rail não tem
por onde ser atribuído. O parceiro não perde volume nessa situação: ele desaparece da série, e um
gráfico por parceiro lê o desaparecimento como se a chave dele tivesse parado.

## Pagar na chave não é passar o QR estático

Um QR estático carrega o identificador do parceiro no campo `62/05` do padrão EMV, e a processadora
devolve esse valor no `metadata["code"]` do webhook. Um pagamento feito digitando a chave direto,
sem ler o QR e sem usar o copia e cola, não carrega referência nenhuma.

Foi assim que um pagamento de valor baixo ficou **sem dono**: o dinheiro liquidou na conta de
recebimento e o daemon estacionou a transação em modo manual, recebida e vinculada a ninguém, e o
caso só apareceu quando o parceiro reclamou dias depois. O tratamento certo era `/setpartner`
seguido de `setauto`, que entregaria o DePix. O caso foi lido como falha do QR estático e devolvido,
e a devolução também não liquidou.

A lição operacional cabe numa pergunta: antes de tratar um estático sem dono como falha, conferir
se o pagador **leu o QR** ou **digitou a chave**. As duas situações têm a mesma aparência na base e
tratamentos opostos.

## O campo que parece chave e não é

Na linha da processadora, `bank_tx.pix_key` guarda o `customerId` do pagador (a equivalência nunca
foi validada). O campo é por pagador, então o mesmo valor aparece com parceiros diferentes toda vez
que uma pessoa comprou de dois deles. Agrupar por `pix_key` para achar chave compartilhada devolve
milhares de linhas sem que exista incidente algum por trás.

Quem decide o dono no nosso lado continua sendo o `metadata.code`.

## O `code` pode chegar errado, e o teste que mostra isso

O daemon copia `metadata.code` sem interpretar. Quando o valor chega errado da processadora, o
crédito sai errado, e nenhuma checagem interna acusa.

Num caso real, um pagador enviou o pagamento com um identificador impresso no comprovante dele, e o
webhook do mesmo pagamento chegou dias depois, numa leva de recuperação, declarando outro parceiro.
A plataforma creditou o que o webhook dizia. A conferência que separou a corrupção externa de um
erro nosso foi barata: a mesma linha trazia um segundo campo furado, com o banco de origem declarado
no payload divergindo do ISPB (o identificador da instituição que vem dentro do próprio E2E do Pix).
Esse cruzamento devolve pouquíssimas linhas, o que o torna um teste com pouco falso positivo.

A regra que sai daí é que um comprovante contestado **não se fecha pelo `metadata.code` sozinho**.
Ler o identificador impresso na imagem e cruzar o ISPB é o que decide.

## Como conferir cobertura sem fixar um número

A cobertura do `code` por rail muda quando o tráfego migra, então vale medir na hora em vez de
citar um percentual antigo. A quebra útil separa quantas linhas estáticas existem por rail e
quantas delas têm `code`, o que distingue queda de volume de queda de atribuição:

```sql
SELECT split_part(split_part(bank_tx_identifier, '_', 1), '-', 1) AS rail,
       count(*) AS linhas,
       count(tx_extra_info_json::jsonb #>> '{data,metadata,code}') AS com_code
  FROM pix2depix.bank_tx
 WHERE qr_id IS NULL
   AND tx_date >= DATE '2026-06-01'
 GROUP BY 1
 ORDER BY 2 DESC
```

O separador precisa das duas passagens porque as linhas de estorno automático usam hífen no lugar
do sublinhado. Uma linha com `qr_id` nulo é um pagamento sem QR nosso, que é o filtro que isola a
estática. Ao agrupar por data, lembrar que `tx_date` é declarada por quem entregou o webhook (ver
[Armadilhas de consulta](armadilhas-de-consulta-pix2depix.md)).

## Relacionado

- [FAQ de atendimento](faq-atendimento.md) — o que responder ao parceiro quando o assunto é
  depósito não creditado.
- [Estorno de depósito](../estornos/estorno-de-deposito.md) — o modelo de dados por trás de uma
  devolução, para quando a decisão é devolver em vez de atribuir.
- [Armadilhas de consulta](armadilhas-de-consulta-pix2depix.md) — as demais leituras desta base que
  devolvem menos linha em vez de erro.
