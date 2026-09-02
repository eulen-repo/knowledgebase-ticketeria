# Integração / API

A documentação técnica completa e sempre atualizada de campos e endpoints está em
**[docs.eulen.app](https://docs.eulen.app)**. Este documento resume o que os parceiros mais
perguntam.

## Identificação obrigatória (desde 18/06/2026)

Três pontos que antes eram opcionais passaram a ser **obrigatórios**:

1. **Identificação do pagador** — em cobranças a cliente final, é preciso informar o **`euid`** do
   cliente ou, na ausência dele, o **`endUserTaxNumber`** (CPF) acompanhado de **`endUserFullName`**.
   Cobranças sem isso são recusadas.
2. **Identificação do merchant** — em cobranças de merchant, é preciso enviar o **`merchantId`** (o
   EUID do merchant que recebe). Cobranças de merchant sem `merchantId` são recusadas.
3. **Delay em pagamentos de merchant** — pagamentos de merchant passam a ter um **piso mínimo de
   atraso** (`delayDepixInHours`), aplicado conforme o perfil de risco do merchant. A liquidação
   imediata deixou de estar disponível para merchants; o parceiro pode aumentar o atraso, mas não
   ficar abaixo do piso.

## Criar uma cobrança

Uma cobrança é criada por `POST /deposit`, informando o valor em centavos e a identificação do
pagador. Exemplo (cliente final):

```jsonc
{
  "amountInCents": 50000,
  "euid": "<EUID do cliente>"        // ou: "endUserTaxNumber": "<CPF>" + "endUserFullName": "<nome>"
}
```

Cobrança de merchant inclui o `merchantId`:

```jsonc
{
  "amountInCents": 150000,
  "merchantId": "<EUID do merchant>"
}
```

O campo `delayDepixInHours` (1 a 720) atrasa a entrega do DePix; se omitido em cobrança comum, a
entrega é imediata (para merchant, vale o piso obrigatório).

## Consultar status

O parceiro consulta o estado das próprias transações pela API:

- `GET /api/deposit-status?id=` — status de um depósito.
- `GET /api/withdraw-status?id=` — status de um saque.
- `GET /api/deposits?start=&end=&status=` — lista de depósitos por período.
- `GET /api/user-info?euid=` — limite diário do cliente, quanto já foi usado no dia e o horário do
  reset.

## Token de API

O **token de API o próprio parceiro gera** (não é preciso pedir ao time da Eulen). É possível
escopar o token (depósito, saque, usuário) e definir a validade.

## Webhooks

É possível registrar um webhook por tipo de evento — **`deposit`, `withdraw` e `med`** —, um de cada
por grupo. Para ter mais de um webhook do mesmo tipo, usa-se um segundo grupo com webhook próprio.

## QR estático e chave Pix fixa

O **QR estático** (reutilizável) está em **liberação gradual, habilitado por parceiro** — pode ainda
não estar disponível para todos. Uma **chave Pix fixa própria por parceiro não existe**: a regra do
Banco Central limita a 20 chaves por conta bancária, então as chaves são remanejadas conforme o
volume. O QR estático é a solução para o caso de uma cobrança fixa reutilizável.
