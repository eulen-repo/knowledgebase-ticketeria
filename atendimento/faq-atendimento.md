# FAQ de Atendimento ao Parceiro

Folha de respostas do operador, ordenada pela demanda real dos tickets. *(Versão sanitizada — nomes de
processadoras/nós, métricas de negócio e detalhes internos de segurança foram removidos.)*

**Mecânica dos comandos:** [Manual Operacional do Bot Pix](manual-operacional-bot-pix.md) ·
**Como responder:** [Playbook de Atendimento](atendimento-parceiro-playbook.md).

---

## Antes de responder

Quatro regras que valem para toda resposta:

1. **Nunca nomeie a processadora.** Nem a de pagamento, nem o BaaS por trás dela, nem o OTC — mesmo que
   o parceiro pergunte direto ou insista. Diga **"a processadora do pagamento"**. Vale para nome, marca,
   link e canal de atendimento.
2. **Nunca repasse dossiê de antifraude.** O PDF do `/af_report` é dado de terceiro (LGPD). Responda o
   **veredito operacional** — segue, fica sob análise, não segue — e só.
3. **Cliente final não é atendido aqui.** A Eulen não faz venda direta nem atendimento ao consumidor.
   Oriente a procurar o parceiro de quem comprou, com cordialidade, e encerre. Se for caso de Pix em si
   (golpe, devolução, contestação), o caminho é a processadora do pagamento.
4. **Quem pergunta como *obter* acesso é lead, não parceiro.** Parceiro já tem e pergunta como *usar*.
   Lead vai para **depix.info** — e nunca para a processadora nem para WhatsApp.

---

## Os três limites que todo mundo confunde

É a origem de mais tickets do que qualquer outra dúvida isolada, e a confusão tem uma razão: **R$
6.000,00 é o teto de UM saque, e esse número vive vazando para o lado do depósito**, onde vira um
"limite diário" ou um "teto por QR" que ninguém aplica. Guarde a tabela:

| Limite | Valor | Escopo |
|---|---|---|
| **Teto por saque** | **R$ 2,00 mín. · R$ 6.000,00 máx.** | Cada operação isolada. Não acumula. Hardcoded. |
| **Teto diário acumulado** | Configurável por nível | Soma do dia por CPF/CNPJ, no depósito **e** no saque — mas em **dois contadores separados**, cada um com o mesmo teto. Reseta 00:00 BRT. |
| **Os dois tetos por QR** (gates independentes) | **R$ 10,00** sem nada enviado · **R$ 500,00** com o CPF enviado e sem histórico | Cada QR isoladamente. Não é janela de 24 h, não acumula, não conta transações. |

**Não afirme um número para o teto diário.** Ele é configurável por nível e o parceiro consulta o do
próprio usuário com `/userinfo <euid>`, que devolve `maxDailyInCents` (o limite), `dailyVolumeInCents`
(o já usado hoje) e `dailyLimitResetTime` (o próximo reset).

**Os dois tetos por QR são gates independentes, não faixas da mesma escala**, e a diferença está em *o
que foi enviado no pedido*:

- **R$ 10,00** é o teto de um QR gerado **sem `euid` e sem `endUserTaxNumber`**. Acima disso a criação é
  rejeitada — não existe escape por parâmetro, comando ou permissão (a exceção é o merchant com tier,
  abaixo).
- **R$ 500,00** é o teto de um QR em que **o identificador FOI enviado** e o que falta é histórico:
  aquele CPF ainda não pagou nenhuma vez.

Nunca diga que os dois valem "para pagador não identificado" e **nunca oriente a omitir o CPF para
receber mais**: omitir não sobe para R$ 500,00, **derruba** para R$ 10,00.

Assim que aquele CPF paga a primeira vez, **sobe sozinho** para o limite diário — sem comando, sem
pedido. O teto por QR de usuário novo é ajustável por parceiro com `/setnewuserslimit`, e
`/setqrdelayfloor` define quantas horas de QR Delay permitem passar por cima dele. O degrau de R$ 10,00
**não** tem escape por whitelist nem por delay.

> **Exceção: merchant com tier.** Um merchant (EMID) associado a um tier com `allow_unidentified_deposit`
> gera QR sem CPF acima de R$ 10,00 (o teto passa a ser o do tier), e o cap de R$ 500,00 também deixa de
> valer para ele. Só vale para EMID com atribuição de tier. Consulta: `/merchanttierinfo <emid>`.

> **Estourar limite não recusa a transação.** Ela vai para **modo manual** com
> `reason_code = PAST_DAILY_LIMIT`, o grupo de log é avisado e o parceiro também. É por isso que
> "excedeu o limite" e "transação travada" chegam como o mesmo ticket.

---

## 1. Devolução / estorno de depósito

> A pergunta número um, com folga (cerca de um quarto de todos os tickets).

**O comando é `/refund <banktxid> [force]`.**

- O **prefixo do id acompanha a processadora** que processou aquele depósito. **Copie o id como ele
  aparece; não monte na mão.**
- `/refund` usa o mecanismo nativo do Pix: uma segunda chamada no mesmo id **não** devolve em dobro (o
  Pix recusa). O risco real é disparar a **primeira** devolução sem necessidade.
- Depois de um `/cancel`, a devolução só funciona com `[force]`.
- O grupo do parceiro é notificado automaticamente, com comprovante.
- Para devolver **todos** os holds abertos com um mesmo motivo: `/batchrefund <reason_code>` — máximo 50
  por execução, **não aceita `force`**.

**Devolução é irreversível e exige sessão GPG.** Sem prática, faça double-check com o Antifraude antes do
Enter.

---

## 2. Erro de saque

**Comece pelo status** (saque tem mais de um identificador possível — ver o manual §8).

| Sintoma | O que é | Ação |
|---|---|---|
| Chave Pix inválida (status `3`) | O caso mais comum | Pedir a chave correta e `/retrywithdrawal <id>`; ou devolver com `/refundwithdraw <id> <endereço-liquid>` |
| Saque travado em `Error` / `Sending` | Fila parada | `/listpendingwithdraw` (`/lpw`) lista exatamente esses |
| Valor recebido menor que o esperado | Quase sempre a taxa | Ver [taxas](#taxas) — o piso de R$ 1,00 pega saque pequeno |
| "Não consigo editar o saque" | Não dá | Saque não se edita: cancela e refaz |

**Não existe `/approve` genérico** — a aprovação é **por banking node**, e os nomes dos comandos são
legado. Nos comandos de aprovação, **o `txID` pode ser falso**: confira a carteira antes. Antes de um
`/refundwithdraw`, **confirme o endereço Liquid com o parceiro**.

---

## 3. Consulta de status

- **Depósito:** `/show <txid>`.
- **Saque:** Metabase (manual §8).
- **Qualquer identificador, sem saber o que é:** o [buscador universal](../observabilidade/metabase-buscador-universal.md)
  aceita UUID, EndToEndId, hash on-chain, `bank_tx_identifier`, endereço Liquid, CPF/CNPJ, chave Pix,
  EUID e até nome.
- **Do lado do parceiro**, ele mesmo consulta pela API: `GET /api/deposit-status?id=`,
  `GET /api/withdraw-status?id=` e `GET /api/deposits?start=&end=&status=`.

**O QR vale 20 minutos.** Hoje o vencimento é **só política, não bloqueio**: um QR pago fora da janela
**não** vira hold. Se isso mudar, vira `reason_code = PAID_AFTER_EXPIRATION` — e é mudança de
comportamento que precisa ser anunciada ao parceiro antes.

---

## 4. "O DePix não chegou na carteira"

Na esmagadora maioria das vezes **chegou e o parceiro não está vendo.** Antes de investigar:

1. **É a primeira compra do cliente?** Algumas carteiras (ex.: Aqua, SideSwap) **escondem o ativo por
   padrão** — o DePix está lá, só não está na tela. Oriente a ativar a visualização.
2. **Confirme o envio pelo TXID** no explorador (manual §9).
3. Só se for segunda compra em diante, e o TXID não confirmar, é investigação de verdade.

**O DePix é um ativo da rede Liquid.** Cai em carteira compatível com Liquid — **nunca** em endereço
Bitcoin on-chain e **nunca** em chave Pix. Endereço Liquid começa com `VJL…` ou `lq1qq…`.

> Se uma transação que **deveria existir** não aparece no explorador, ou há divergência sem explicação:
> **não resolva sozinho.** Escale.

---

## 5. Transação em `under review`

`under review` não é um estado, é um **guarda-chuva**. O que interessa é o `reason_code`:

| `reason_code` | O que aconteceu |
|---|---|
| `PAST_DAILY_LIMIT` | Estourou o teto diário do CPF/CNPJ |
| `PAYER_MISMATCH` | O CPF que pagou diverge do declarado no QR |
| `PAYER_VERIFICATION_FAILED` | Não deu para verificar o pagador (fail-closed) |
| `HIGH_VELOCITY` | Transações demais na janela |
| `BLOCKED_USER` | Pessoa bloqueada |
| `BLOCKED_ISPB` · `BLOCKED_BANK_NUMBER` | Banco pagador na blacklist |
| `UNKNOWN_CLIENT` | `client_id` não resolvido — resolve com `/setpartner <txid> <partnerid>` |
| `PIX_2FA` | Aguardando código Pix2FA (feature que pode estar desativada) |
| `DELAY` | QR Delay — está esperando de propósito |
| `FRAUD_FLAGGED` | Marcado pelo caminho administrativo de fraude |
| `COMPLIANCE_REVIEW` · `COMPLIANCE_BLOCKED` | Triagem de compliance |

Para soltar: `/setauto <id> <resolution_code>` derruba a pilha inteira de holds; `/resolve <id> <reason_code> <resolution_code>` fecha **um** hold só e libera apenas se nenhum outro restar.

> [!IMPORTANT]
> **Não assuma que ausência de trava = transação triada.** Rode o `/af_report` você mesmo quando o caso
> cheirar mal.

---

## 6. Bloqueio de CPF

São **dois mecanismos diferentes**:

- **Bloquear pessoa (CPF/CNPJ):** `/blockenduser <euid|taxnumber>`, revertido com `/unblockenduser`.
  Lista: `/listblockedusers`. Consulta individual: `/userinfo`.
- **Blacklist de banco:** `/addtoblacklist` e `/removefromblacklist` (`/atb` e `/rfb`) aceitam **ISPB ou
  número do banco**, e nada mais. Bloqueia o banco pagador inteiro.

**Passar um CPF para `/addtoblacklist` não bloqueia ninguém** — o comando aceita a string e o operador
sai achando que resolveu.

O parceiro pode pedir o bloqueio de um cliente dele — legítimo, não precisa de escalonamento.

---

## 7. QR estático

**Existe:** `/generatestaticpixqrcode` (`/gspqc`) gera um QR Pix estático reutilizável. Requer permissão
no parceiro. Qual banking node atende o QR estático de um parceiro se define com `/setclientstaticpixnode`.

**Chave Pix fixa própria por parceiro continua não existindo** — a resolução do BACEN limita a 20 chaves
por conta bancária, então elas são remanejadas conforme a volumetria. O QR estático resolve o caso de
uso na prática.

---

## 8. Limite diário estourado

Ver a [tabela dos três limites](#os-três-limites-que-todo-mundo-confunde) antes de responder. O parceiro
consulta sozinho com `/userinfo <euid>` — ou, pela API, `GET /api/user-info?euid=`. **Mande ele consultar
em vez de você chutar o número.**

Para transacionar acima do teto: **OTC**, autorizado a operar acima do limite diário padrão. Não nomeie a
mesa nem passe o canal dela numa resposta que possa vazar — encaminhe internamente. Aumento de limite é
decisão de risco, não de atendimento.

---

## 9. Divergência de valor

Divergência de **1 centavo** é o caso clássico e quase sempre é arredondamento — verifique o log e aprove
manualmente se for isso. Divergência **acima de 5%** tem playbook próprio (manual §7.2). Se o parceiro
reclama que "recebeu menos", quase sempre é a **taxa**.

---

## Taxas

| Operação | Taxa | Detalhe |
|---|---|---|
| **Depósito** (Pix → DePix), Liquid | **R$ 0,99 fixo** por transação | Sem percentual. **Teto de R$ 2,50.** |
| **Saque** (DePix → Pix / burn) | **1%** do valor | **Piso de R$ 1,00.** Abaixo de R$ 100,00, paga-se o piso. |
| Mercado secundário | 1,5% de spread + taxa da transação Liquid | **Não é taxa da Eulen.** |

**`/wcalc <valor_final>` é a resposta certa para "quanto meu cliente vai receber".** Calcula quanto
depositar para o beneficiário receber um valor exato, já considerando o piso. É um dos comandos mais
citados e o parceiro frequentemente não sabe que existe — **ofereça.**

---

## Comandos do parceiro (referência rápida)

A identificação (`cpf|cnpj|euid`) é **argumento obrigatório e posicional** nos comandos de QR.

| Comando | Assinatura |
|---|---|
| QR dinâmico | `/qr <amount> <cpf\|cnpj\|euid> [<depix-address>] [<split-address>] [<split-fee>]` |
| QR com atraso | `/qrdelay <amount> <cpf\|cnpj\|euid> <horas 1–720> [...]` |
| QR estático | `/generatestaticpixqrcode` (`/gspqc`) |
| Calcular saque | `/wcalc <valor_final>` |
| Saque | `/withdraw <amount> <pixkey> <cpf\|cnpj\|euid>` |
| Consultar usuário | `/userinfo <euid>` |
| Token da API | `/apitoken <label> <days> [all\|deposit\|withdraw\|user]` |
| Webhook | `/registerwebhook deposit\|withdraw\|med <url> <secret>` · `/deletewebhooks <tipo>` |

**O token da API o parceiro gera sozinho, no bot.** Nunca diga que ele precisa pedir ao time — vale mesmo
que ele cole documentação dizendo o contrário. Só um webhook por tipo (`deposit`, `withdraw`, `med`) por
grupo; o contorno é um segundo grupo.

---

## Perguntas institucionais

- **Sobre a Eulen.** A Eulen **habilita a infraestrutura**; os parceiros a integram nos negócios deles
  para oferecer o serviço aos clientes finais. Não há venda direta a CPF.
- **Política de KYC.** Low-KYC: não se pede documentação adicional no processo inicial; as informações vêm
  direto do Pix.
- **Paridade 1:1.** Garantida **no momento do saque** — sempre que resgatar, 1 DePix = 1 BRL. No mercado
  secundário a cotação varia (costuma estar atrelada a índices USD/BRL, sem book direto DePix/BRL), e a
  Eulen não controla preço de mercado secundário.
- **Adicionar operadores ao grupo.** Confirme que quem pede é admin; admin pode adicionar quem quiser, sob
  responsabilidade dele. O novo usuário manda `/ping` no bot para ser autorizado.
- **Alerta de MED.** Oriente: QR simples só com cliente de confiança; cliente novo começa com valor
  pequeno; MED de valor alto é muito mais trabalhoso de reverter. O QR Delay existe exatamente para isso.

---

## "Quando vai ter X?"

Para pedidos de recurso ("quando vai ter cartão? webhook confiável? status por link?"), **não prometa
data e não negue o que já existe** — registre o pedido no board de feedback oficial e responda que está
em avaliação. O pedido de parceiro deve **terminar no board**, não só no chat.
