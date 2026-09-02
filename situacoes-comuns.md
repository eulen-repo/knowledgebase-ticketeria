# Situações Comuns

## "O DePix não chegou na carteira"

Na maioria dos casos o DePix **chegou**, mas não está aparecendo:

- **Primeira compra do cliente:** muitas carteiras **escondem o ativo por padrão**. O DePix está lá,
  só não está visível — basta **ativar a visualização** do ativo na carteira.
- **O DePix é um ativo da rede Liquid.** Ele só cai em uma carteira compatível com Liquid, em um
  endereço que começa com `lq1` ou `VJL`. Nunca cai em um endereço Bitcoin comum e nunca em uma chave
  Pix.
- Para confirmar que o envio aconteceu, é possível verificar o **TXID** (identificador da transação)
  em um explorador da rede Liquid.

Se, a partir da segunda compra, o TXID não confirmar o envio, aí sim é caso de investigar com o
suporte.

## A transação está "em análise" ou parada

Uma transação pode ficar retida por **revisão** ou por um **atraso programado** (o QR Delay, uma
janela de segurança). Os motivos mais comuns de uma retenção por revisão:

- **O CPF que pagou é diferente do informado na cobrança** — a transação é recusada/devolvida.
- **O limite diário do pagador foi atingido.**
- **Muitas transações em pouco tempo** vindas do mesmo pagador.
- **O pagador (ou o banco de origem) está bloqueado.**

Uma transação em revisão **não é recusada automaticamente** — ela aguarda análise.

## Diferença de valor no depósito

- **Diferença de 1 centavo:** quase sempre é arredondamento.
- **Diferença grande** (o valor pago é muito diferente do cobrado): o pagamento **não é processado
  automaticamente**. O caminho é criar uma **nova cobrança** com o valor que efetivamente foi pago.

## "Recebi menos do que esperava" (saque)

Quase sempre é a **taxa** do saque (1% do valor, com piso de R$ 1,00). Ver [Limites e Taxas](limites-e-taxas.md#taxas).

## Quero uma chave Pix fixa

Uma chave Pix fixa própria por parceiro **não existe** (limite do Banco Central de 20 chaves por
conta). A solução é o **QR estático**, que é reutilizável. Ver [Integração / API](integracao-api.md#qr-estático-e-chave-pix-fixa).

## Recebi um MED (uma contestação)

O **MED (Mecanismo Especial de Devolução)** é um mecanismo do Pix pelo qual o banco de quem pagou
pede a devolução de uma transação, alegando fraude ou golpe. Pontos importantes:

- Enquanto o caso é analisado, o **banco do recebedor pode bloquear o saldo** correspondente.
- **Não há um prazo regulatório fixo de bloqueio.** Os prazos que um banco informa (ex.: "48 horas",
  "N dias") são daquele banco, não uma regra do Banco Central. O desbloqueio acontece por evento
  (quando o banco encerra a análise, conclui a devolução, ou o caso é cancelado).
- A Eulen **nunca inicia** um MED. Quando solicitado, pode fornecer uma **declaração de que não
  solicitou o MED**, que o parceiro encaminha ao seu banco.
- Ao contestar, é útil reunir **a evidência do que foi entregue**: o que o cliente recebeu e quando, em
  qual plataforma, o identificador da entrega no seu sistema, e o histórico do cliente. Havendo saldo
  remanescente daquele cliente, recomenda-se bloqueá-lo.
- Sempre que possível, oriente o **cliente final a pedir o cancelamento do MED** à instituição em que
  ele fez o Pix — o cancelamento retira a marcação de fraude.

## Boas práticas contra fraude (MED)

- Use QR simples apenas com clientes de confiança; com cliente novo, comece com valores menores.
- Um MED de valor alto é muito mais trabalhoso de reverter — o **QR Delay** (atraso programado) existe
  justamente para dar tempo de barrar uma fraude antes de o DePix ser liberado.
