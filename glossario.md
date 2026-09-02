# Glossário

Termos que aparecem ao usar o DePix e a integração da Eulen.

### DePix

Um token digital cujo valor acompanha **1 real brasileiro** (paridade 1:1). Circula na rede
**Liquid**. Não é depósito bancário, investimento nem moeda de curso legal — é um ativo digital
lastreado em real. Docs antigos podem chamá-lo de `eBRL`; o nome atual é DePix.

### Liquid

A rede em que o DePix circula (uma sidechain do Bitcoin). Um endereço Liquid começa com `lq1` ou
com `VJL`/`VL`. O DePix **só** é recebido em uma carteira compatível com Liquid — nunca em um
endereço Bitcoin comum nem em uma chave Pix.

### Pix → DePix (depósito)

O fluxo em que um pagamento Pix recebido vira DePix. É a compra: entra real, sai DePix.

### DePix → Pix (saque)

O fluxo inverso: o DePix é resgatado e vira um pagamento Pix em reais.

### QR dinâmico

Um QR Code de cobrança gerado para **uma** transação, com valor definido. É o QR usado no dia a dia
para receber um pagamento.

### QR estático

Um QR Code **reutilizável** (copia-e-cola mais imagem), que pode receber vários pagamentos. Resolve
o caso de quem quer uma cobrança fixa. Precisa estar habilitado na conta do parceiro.

### QR Delay (atraso programado)

Um atraso na entrega do DePix depois que o Pix já foi pago, configurável entre **1 e 720 horas**. É
uma janela de segurança: dá tempo de uma contestação (MED) aparecer antes de o DePix ser liberado.

### EUID (identidade do usuário final)

O identificador de um **cliente final** na Eulen. É gerado quando o cliente faz o cadastro e é usado
para identificar quem paga em uma cobrança.

### EMID / `merchantId` (identidade do merchant)

O identificador de um **merchant** (o negócio que o parceiro atende e que recebe de seus próprios
clientes). Identificar cada merchant permite tratar um problema pontual sem afetar os demais.

### MED — Mecanismo Especial de Devolução

Um mecanismo do Pix pelo qual o banco de quem pagou pode pedir a devolução de uma transação em caso
de alegação de fraude ou golpe. Quando um MED chega, o valor recebido pode ficar bloqueado enquanto
o caso é analisado. Ver [Situações comuns](situacoes-comuns.md#recebi-um-med-uma-contestação).

### E2E (End-to-End ID)

O identificador único de um pagamento Pix, atribuído pelo Banco Central. Começa com `E` seguido de
32 dígitos. É o código que identifica um depósito específico.

### Paridade 1:1

A garantia de que **1 DePix equivale a 1 real** no momento do resgate (saque). Em mercado secundário
(uma exchange, por exemplo) a cotação pode variar, porque ali o preço é de mercado.

### On-ramp / off-ramp

*On-ramp* é entrar em cripto (real → DePix, uma compra). *Off-ramp* é sair (DePix → real, uma venda).
