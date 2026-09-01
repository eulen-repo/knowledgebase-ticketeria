# Glossário

Vocabulário operacional da ticketeria: o que cada termo significa e por que importa no
atendimento. Versão sanitizada — nomes de parceiros/processadoras, métricas de negócio e
referências internas foram generalizados ou removidos.

---

### DePix

O token no centro do produto: um token digital cujo valor acompanha um real brasileiro.
Explicitamente **não** é depósito bancário, valor mobiliário, contrato de investimento nem moeda
de curso legal. Roda na **Liquid**. Docs antigos chamam de `eBRL`; o nome atual é DePix.

### pix2depix / pix2depixd

`pix2depix` é o produto principal: o sistema que transforma um Pix recebido em DePix. `pix2depixd`
é o daemon de backend no núcleo — o serviço e o que os incidentes referenciam. Docs arquivados
antigos chamam de "o bot da Eulen": era a interface original no Telegram, não o sistema.

### MED — Mecanismo Especial de Devolução

O mecanismo Pix que o banco de uma vítima de fraude usa para puxar uma transação de volta (Res.
Conjunta nº 6/2023). Alguém reporta um pagamento como fraude e o dinheiro pode ser retirado da
cadeia recebedora **depois** de já termos entregue o cripto. É o principal risco financeiro da
operação, não uma nota de rodapé de compliance. (`Mecanismo`, não *Medida* — o termo do BACEN.)

### MED-rate

A fração das transações de um parceiro que voltam como MED. Alimenta a régua de tier e risco.
Tem duas formas que **não** são intercambiáveis:
- **observada** — MEDs reportadas na janela ÷ depósitos da janela; oportuna, mas superestima.
- **cohort** — segue o depósito; é o nível real.
Sempre declarar qual delas um card/relatório usa.

### RDR — Requisição de Devolução de Recursos

> [!CAUTION]
> A expansão acima **não** é a do Banco Central. No corpus do BC, RDR é o *Sistema de Registro de
> Demandas do Cidadão*. Não usar a expansão em texto para fora.

Na operação, o que se chama de RDR chega como um chamado de portal de uma **processadora**, em
geral semanal, uma transação por vez, e é respondido com um documento de evidência de blockchain
(prazo típico de **10 dias úteis** para responder o usuário e anexar a evidência).

### Banking node (nó bancário)

Uma entidade brasileira (CNPJ) atuando como gateway de pagamento Pix para uma exchange: detém a
conta bancária, integra com o sistema financeiro e liquida fiat. Não confundir com **Swap node**
(que registra a conversão BRL↔DePix) nem com **Operator node** (o parceiro que fica de frente para
o usuário final). *As entidades específicas que atuam como banking node são informação interna e
não aparecem aqui.*

### Parceiro (Operation Node)

O **negócio** no meio do modelo B2B2C da Eulen: opera o app, site ou bot que fica de frente para o
usuário final; a Eulen nunca alcança além dele. Um parceiro não é cliente nem fornecedor — opera
um nó da rede. Operam sob a forma jurídica que lhes convém (CNPJ, CPF de pessoa física, entidade
offshore, holding). **Um modelo que assume que todo parceiro é uma empresa brasileira está errado
sobre a maior parte da base.**

### Merchant

No fluxo de merchant-acquiring, o negócio que um *parceiro* atende — de um vendedor de rua a um
escritório de contabilidade — que por sua vez recebe dos seus próprios clientes via Pix. É o que
torna a cadeia de quatro níveis em vez de três. Identificado por um **EMID**.

### A cadeia — B2B2C, ou B2B2B2C com merchants

```
Fluxo P2P        Eulen ──► parceiro ─────────────────► usuário final    identidade: EUID
Fluxo merchant   Eulen ──► parceiro ──► merchant ────► cliente final    identidade: EMID (no merchant)
```

A Eulen lida **apenas** com o parceiro; tudo além dele é relação do próprio parceiro. O cliente
final nunca é atendido diretamente.

### EUID — Eulen User ID

A identidade de um **usuário final**, gerada quando ele faz onboarding e assina os termos via um
micro-Pix, e carregada nas transações seguintes. Usada em fluxos P2P/B2C.

### EMID — Eulen Merchant ID

A identidade de um **merchant**, por merchant (não por transação). Serve à granularidade: permite
bloquear um lojista problemático sem derrubar o parceiro inteiro. Carrega um tier de risco
alimentado por uma janela móvel de MED, e esse tier governa o QR Delay e os tetos do merchant.
Um valor EMID é um EUID — o do merchant — no mesmo formato `EU…` de dezessete caracteres.

### EPID — Eulen Partner ID

A identidade de um **parceiro**, emitida por nós. Fecha a cadeia que já tinha EMID (merchant) e
EUID (usuário final) e onde o nível de parceiro não tinha chave. Existe porque o campo antes usado
como chave de fato era raspado do pagamento de teste de onboarding e registrava o documento de
quem pagou, não da entidade parceira. A granularidade é a **entidade**, não a marca: uma
plataforma whitelabel e as marcas que rodam nela são vários parceiros, não um.

### Validação de CPF / screening (KYC)

Um **provedor externo de dados** é usado para validação de CPF, situação cadastral e triagem de
PEP/sanções sobre o pagador antes de um depósito ser aceito, devolvendo um veredito
(BLOCK / ALLOW / MANUAL_REVIEW), pareado com um provedor de risk scoring. *Os fornecedores
específicos são informação interna.*

### risk-sentinel

O serviço de antifraude que chama o provedor de validação e grava os vereditos no log de avaliação.

### edge gate (gate de borda)

O formato que a triagem antifraude deve ter: um ponto de decisão em cada uma das quatro fronteiras
— `pixin`, `pixout`, `depixin`, `depixout` — devolvendo allow/block/manual review antes de a
transação andar. O `risk-sentinel` é a implementação disso.

### QR Delay

Um atraso na entrega do DePix depois que o Pix já foi pago, configurável entre **1 e 720 horas**.
Compra tempo para uma MED aparecer antes de o cripto sumir — o controle antifraude mais barato
disponível. O piso é atrelado ao tier de risco do merchant.

### Pix2FA

Autenticação de dois fatores por Pix: um micro-pagamento de R$ 0,01 cuja descrição carrega um
código de verificação, que o pagador devolve para provar que controla a conta pagadora. Recurso
que pode estar pausado — confirmar o estado atual.

### KYC / KYB

*Know Your Customer* e *Know Your Business*: verificação de identidade e risco de uma pessoa
física e de uma empresa. KYC é a checagem do lado do pagador; KYB cobre o parceiro ou o merchant e
alcança o UBO.

### PEP — Pessoa Politicamente Exposta

Alguém em cargo público relevante, mais família próxima e associados. Não são barrados — são
triados com mais rigor: um hit de PEP dispara *due diligence* reforçada em vez de bloqueio.

### DICT — Diretório de Identificadores de Contas Transacionais

O registro do BACEN que liga chaves Pix a contas. O que importa aqui é a segunda coisa que ele
guarda: as marcações de fraude que cada participante Pix registra contra uma chave ou um CPF,
consolidadas por todo o ecossistema — quantas instituições marcaram, quantas vezes, por quanto.

### on-ramp / off-ramp

On-ramp é BRL entrando e DePix saindo (uma compra). Off-ramp é o inverso (uma venda). Os fluxos são
assimétricos em risco e em código — uma MED só atinge o lado on-ramp.

### TPV — Total Payment Volume

O dinheiro que passa pela plataforma. Métrica de negócio; **valores agregados não aparecem nesta
base.**

### Mike / Nick

**Não são pessoas.** São contas *userbot* do Telegram que leem os grupos de parceiro para o
pipeline de tickets. "Os grupos que o Mike vê" significa a visibilidade do bot, não de um colega.

### ATD / ENG / DEM

Prefixos de protocolo de ticket: suporte a parceiro (**ATD**), engenharia (**ENG**) e o board de
DEMANDAS (**DEM**).

### laranja

Termo brasileiro para *money mule* — uma conta aberta em nome de terceiro para receber produto de
fraude. Aparece sem tradução mesmo em docs em inglês.

### db2 / db4 / db5 / db6

IDs de banco no Metabase, usados como números crus ao longo dos docs de operação. Os principais:
**db2** = base transacional do `pix2depix` (a principal); **db4** = o CRM. *(Detalhes de host e
acesso são internos e não aparecem aqui.)*

### Outros acrônimos que você vai encontrar

| Acrônimo | Significado |
|---|---|
| **AML / CFT · PLD/FT** | *Anti-Money Laundering / Counter Financing of Terrorism*; o par em português é Prevenção à Lavagem de Dinheiro / Financiamento ao Terrorismo. |
| **BACEN** | Banco Central do Brasil. Regula o Pix, o SPI e o DICT. |
| **EDD / SDD** | *Enhanced / Simplified Due Diligence*: as trilhas de KYC pesada e leve, escolhidas por tier de risco. |
| **ISPB** | Identificador do Sistema de Pagamentos Brasileiro: o código numérico de cada instituição financeira. |
| **LGPD** | Lei 13.709/2018, a lei brasileira de proteção de dados. |
| **PSP** | Prestador de Serviço de Pagamento: instituição autorizada pelo BACEN a operar no Pix. |
| **PVASP** | Prestador de Serviços de Ativos Virtuais: o status regulado pelo BACEN para operar com cripto no Brasil. |
| **RSFN** | Rede do Sistema Financeiro Nacional, a rede privada entre o BACEN e as instituições. |
| **SAR / ROS** | *Suspicious Activity Report* / Relatório de Operação Suspeita: a comunicação obrigatória às autoridades diante de sinais de atividade ilícita. |
| **SI** | Segurança da Informação. |
| **Screening (fornecedores)** | Provedores externos de triagem de sanções/PEP/listas restritivas usados no antifraude. *Os nomes específicos são internos.* |
| **TaxID** | Número de identificação fiscal: CPF para pessoa, CNPJ para empresa. |
| **UBO** | *Ultimate Beneficial Owner*: a pessoa física que realmente controla uma empresa. Limite de referência: 10% de participação ou controle efetivo. |
| **Padrões internos de compliance** | Uma série numerada de padrões (tiers de risco; PEP e sanções; KYB e UBO; retenção e reporte; regras antifraude). *Conteúdo interno.* |
