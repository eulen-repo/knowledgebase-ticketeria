# Limites e Taxas

## Limites

### Teto por saque

Cada saque (DePix → Pix) tem um valor **mínimo de R$ 2,00** e **máximo de R$ 6.000,00**. Esse teto
vale por operação isolada — **não** acumula.

### Tetos por QR (dependem da identificação do pagador)

Quando uma cobrança é criada, o valor máximo permitido depende de **quem foi identificado**:

- **Sem identificação do pagador:** o QR aceita até **R$ 10,00**. Acima disso, é necessário informar o
  pagador.
- **Com o CPF informado, mas sem histórico** (o pagador ainda não comprou nenhuma vez): o QR aceita até
  **R$ 500,00**.
- **Depois da primeira compra daquele CPF:** o limite sobe automaticamente para o **limite diário** —
  sem precisar de nenhuma solicitação.

Esses tetos valem por QR, cada um isoladamente. Não são uma janela de 24 horas nem acumulam.

### Limite diário

Cada pagador (CPF/CNPJ) tem um **limite diário acumulado**, que soma os valores do dia e **reseta à
meia-noite** (horário de Brasília). Esse limite é configurável e pode variar por cliente — o valor
vigente e o quanto já foi usado no dia podem ser **consultados pela API** (endpoint de informações do
usuário).

### Acima do limite diário

Operações acima do limite diário são tratadas por um **canal de OTC** (balcão). Se um cliente precisa
transacionar acima do limite, o caminho é acionar esse canal pelo contato de suporte — não é algo que
se resolve no fluxo padrão.

## Taxas

| Operação | Taxa | Detalhe |
|---|---|---|
| **Depósito** (Pix → DePix) | **R$ 0,99 fixo** por transação | Sem percentual. Teto de R$ 2,50. |
| **Saque** (DePix → Pix) | **1%** do valor | Piso de R$ 1,00. Abaixo de R$ 100,00, aplica-se o piso. |
| **Mercado secundário** | 1,5% de spread + a taxa da transação na rede Liquid | Não é uma taxa da Eulen — é do mercado onde a troca acontece. |

Para saber **quanto depositar** para que o beneficiário receba um valor exato (já considerando as
taxas), a API oferece um cálculo próprio de saque.
