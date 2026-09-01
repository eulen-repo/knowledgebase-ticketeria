# FAQ por Sintoma — o que o parceiro escreve → o que é → como respondemos

Este FAQ é organizado pela **forma como o parceiro escreve**, não pela categoria interna. A base é a
mesma que o pipeline de tickets usa para detectar um atendimento: o `ticketing.py` marca uma mensagem de
parceiro como pedido de suporte quando ela dispara um **sinal**, e a classificação depois a encaixa num
**tipo**. Aqui cada sinal vira uma entrada, com a resposta canônica e o link para o procedimento.

> Fontes: `HELP_KEYWORDS` e a detecção de candidato do `ticketing.py`; o enum `tipo` da classificação;
> e as respostas dos docs desta base. Para a folha de respostas por categoria/volume, ver
> [FAQ de atendimento](faq-atendimento.md).

---

## Como o sistema enxerga um ticket (e o que conta como "respondido")

**O que abre um ticket.** A mensagem de um parceiro vira candidata a atendimento quando dispara **qualquer
sinal** (a rede é ampla de propósito — quem decide "é atendimento mesmo?" é a classificação depois):

- **pergunta** — a mensagem tem `?`;
- **hash** — tem uma sequência de 16+ caracteres hex (um txid / EndToEndId / hash de liquidação);
- **keyword** — contém uma das palavras de pedido de ajuda (a lista abaixo);
- **menção** — cita o time (por @handle ou clicando no nome).

**As palavras que disparam (`HELP_KEYWORDS`):**

> erro · não caiu · não chegou · pendente · travou · travado · não consigo · suporte · ajuda · problema ·
> parado · sumiu · cadê · não funciona · demora · demorou · **estorno · estornar · devolução · reembolso** ·
> bloqueado · bloqueio · fora do ar · não recebi · urgente · depix não

**O que conta como "respondido/resolvido".** O pipeline considera a mensagem **atendida** quando um
operador humano **responde, reage ou fica ativo** na conversa numa janela de tempo (reply, reação, ou
follow logo depois). Consequência prática para quem atende:

- **Uma resposta já "fecha" o sinal** — mesmo um "tô verificando, já te retorno" conta como engajamento.
  Responder cedo tira o ticket da fila de pendências e acalma o parceiro.
- **Cortesia pura fecha sem ação.** "Obrigado / vlw / show / fechou / tmj", sem `?` e sem id, é tratado
  como **encerramento** — não precisa de resposta nem vira pendência.
- **Todo id ajuda.** O sistema extrai automaticamente EUID, withdrawId, txid, EndToEndId e endereço
  Liquid do texto — peça o identificador cedo (ver o [playbook](atendimento-parceiro-playbook.md), "dá uma
  olhada").

---

## Mapa rápido: sinal → tipo → resposta

| O parceiro escreve algo como… | Sinal | `tipo` provável | Como respondemos |
|---|---|---|---|
| "cadê meu depósito", "não caiu", "não chegou", "o DePix sumiu", "depix não veio" | keyword | `DEPIX_NAO_CHEGOU` / `STATUS` | [§1](#1-cadê-o-depix-não-caiu-não-chegou-sumiu) |
| "deu erro no saque", "não consigo sacar", "chave não encontrada" | keyword + hash | `ERRO_SAQUE` | [§2](#2-erro-de-saque-não-consigo-sacar) |
| "tá pendente", "travou", "parado", "em análise", "under review" | keyword | `UNDER_REVIEW` / `STATUS` | [§3](#3-pendente-travado-em-análise-under-review) |
| "qual o status", cola só um txid/E2E/hash | hash | `STATUS` | [§4](#4-status-o-parceiro-só-cola-um-código) |
| "estorna", "devolve", "reembolso", "recebi um MED" | keyword | `DEVOLUCAO_DEPOSITO` (ou MED) | [§5](#5-estorno-devolução-reembolso-e-o-caso-med) |
| "meu cliente está bloqueado", "bloqueia esse CPF" | keyword | `BLOQUEIO_CPF` / `UNDER_REVIEW` | [§6](#6-bloqueado-bloqueio-de-cpf) |
| "recebeu menos", "valor diferente", "faltou dinheiro" | keyword | `DIVERGENCIA_VALOR` | [§7](#7-divergência-de-valor-recebeu-menos) |
| "demorou", "está demorando", "quando cai" | keyword | `STATUS` / `DELAY` | [§8](#8-demora-quando-cai) |
| "aumenta o limite", "quero transacionar mais" | — | `LIMITE_DIARIO` / `AUMENTO_LIMITE` / `OTC` | [§9](#9-limite-aumento-de-limite-otc) |
| "quero chave fixa", "QR estático" | keyword | `QR_ESTATICO` | [§10](#10-qr-estático-chave-pix-fixa) |
| "fora do ar", "nada funciona", "urgente" (vários parceiros ao mesmo tempo) | keyword | incidente | [§11](#11-fora-do-ar-urgente-possível-incidente) |

---

## 1. "Cadê o DePix", "não caiu", "não chegou", "sumiu"

**Na maioria das vezes chegou e o parceiro não está vendo.** Antes de investigar:

1. **É a primeira compra do cliente?** Algumas carteiras escondem o ativo por padrão — o DePix está lá,
   só não na tela. Oriente a **ativar a visualização** do DePix.
2. **Confirme o envio pelo TXID** no explorador Liquid.
3. Só na 2ª compra em diante, e sem confirmação no explorador, é investigação real.

O DePix é ativo **Liquid** — nunca cai em endereço Bitcoin on-chain nem em chave Pix. Se for depósito em
**chave estática** sem dono, ver [depósito em chave estática](deposito-em-chave-estatica.md). Detalhe:
[FAQ §4](faq-atendimento.md#4-o-depix-não-chegou-na-carteira).

## 2. "Erro de saque", "não consigo sacar"

Comece pelo **status do saque** no Metabase (saque não aparece no `/show`).

| Sintoma | O que é | Ação |
|---|---|---|
| Chave Pix inválida (status `3`) | O caso mais comum | Pedir a chave correta e `/retrywithdrawal <id>`, ou devolver com `/refundwithdraw <id> <endereço>` |
| Travado em `Error`/`Sending` | Fila parada | `/listpendingwithdraw` (`/lpw`) |
| "Recebeu menos" | Quase sempre a taxa | ver [§7](#7-divergência-de-valor-recebeu-menos) |

Saque **não se edita**: cancela e refaz. Procedimento completo: [manual §7.1](manual-operacional-bot-pix.md#71-chave-pix-inválida-status-3)
e [erros conhecidos de saque](withdraw-docs.md).

## 3. "Pendente", "travado", "em análise", "under review"

`under review` é guarda-chuva — o que importa é o **`reason_code`** (por que travou): limite diário,
divergência de pagador, alta frequência, CPF/banco bloqueado, QR Delay, compliance… A tabela completa e o
que fazer para soltar (`/setauto`, `/resolve`) estão em
[FAQ §5](faq-atendimento.md#5-transação-em-under-review) e [manual §4/§6.3](manual-operacional-bot-pix.md#41-depósito).

## 4. "Status" (o parceiro só cola um código)

Se veio só um identificador, **não adivinhe** — identifique o tipo (E2E, txid, QrId, id de saque) e rode:

- **Depósito:** `/show <txid>`.
- **Saque:** Metabase.
- **Sem saber o que é:** o [buscador universal](../observabilidade/metabase-buscador-universal.md) aceita
  qualquer identificador.

Peça o mínimo: o identificador, o que ele quer (status/devolução/reenvio/comprovante) e, se for sobre um
cliente, o CPF/euid.

## 5. "Estorno", "devolução", "reembolso" — e o caso MED

**Distinção que decide tudo:**

- **Devolução operacional de depósito** → comando `/refund <banktxid> [force]`. Irreversível, exige GPG,
  double-check com Antifraude. O modelo de dados e a prova estão em [estorno de depósito](../estornos/estorno-de-deposito.md).
- **Estorno de saque** ("o saque não foi") → depende de o PIX ter saído ou não; ver
  [quanto devolver](../estornos/estorno-de-saque-quanto-devolver.md).
- **"Recebi um MED / contestação"** → **não** é devolução operacional, é contestação bancária. **Não**
  dispare `/refund` — segue o processo de [contestação de MED](med-contestacao-guia-operacional.md).

## 6. "Bloqueado", bloqueio de CPF

Dois mecanismos diferentes: **bloquear pessoa** (`/blockenduser <euid|cpf>`, revertido com
`/unblockenduser`) e **blacklist de banco** (`/addtoblacklist` aceita só ISPB/número do banco — passar um
CPF ali **não bloqueia ninguém**). O parceiro pode pedir o bloqueio de um cliente dele — legítimo. Ver
[FAQ §6](faq-atendimento.md#6-bloqueio-de-cpf) e [listas de controle](listas-de-controle.md).

## 7. "Divergência de valor", "recebeu menos"

- **1 centavo** → quase sempre arredondamento: confira o log e aprove manualmente.
- **Acima de 5%** → o bot ignora o pagamento e a transação fica em revisão; tem
  [playbook próprio](manual-operacional-bot-pix.md#72-divergência-de-valor-acima-de-5).
- **"Recebeu menos" em saque** → quase sempre é a **taxa** (1%, piso de R$ 1,00). Ofereça o `/wcalc` ao
  parceiro. Ver [taxas](faq-atendimento.md#taxas).

## 8. "Demora", "quando cai"

Pode ser **QR Delay** (atraso de propósito, `reason_code = DELAY`) ou status normal. Confirme o estado
(`/show` ou buscador) e explique. O QR vale ~20 minutos; hoje o vencimento é política, não bloqueio. Ver
[FAQ §3](faq-atendimento.md#3-consulta-de-status).

## 9. "Limite", aumento de limite, OTC

Ver a [tabela dos três limites](faq-atendimento.md#os-três-limites-que-todo-mundo-confunde) **antes** de
responder — é a maior fonte de confusão. O parceiro consulta o do próprio cliente com `/userinfo <euid>`;
**não chute o número**. Acima do teto diário → **OTC** (não nomeie a mesa; encaminhe internamente).
Aumento de limite é decisão de **risco**, não de atendimento.

## 10. "QR estático", chave Pix fixa

Chave Pix fixa própria por parceiro **não existe** (limite BACEN de 20 chaves/conta). O que resolve é o
**QR estático** (`/generatestaticpixqrcode`), que requer permissão. Ver [FAQ §7](faq-atendimento.md#7-qr-estático).

## 11. "Fora do ar", "urgente" (possível incidente)

Quando **vários parceiros** reclamam ao mesmo tempo de indisponibilidade, é provável incidente. Não
responda caso a caso improvisando: siga o [comunicado de incidente](comunicado-de-incidente.md) (formato
único no grupo de avisos) e a triagem por sintoma dos [runbooks](runbooks.md). Se for suspeita de
fraude/golpe ou divergência sem explicação, **não resolva sozinho — escale.**

---

## Regras de resposta que valem sempre

Do [playbook](atendimento-parceiro-playbook.md) e das regras de "antes de responder" do
[FAQ](faq-atendimento.md#antes-de-responder):

1. **Responder já acalma** — mesmo antes de resolver.
2. **Identifique antes de agir** — tipo do código → status → ação.
3. **Nunca nomeie a processadora**, nem repasse dossiê de antifraude (LGPD), nem atenda cliente final
   direto.
4. **Todo comando que mexe em dinheiro é irreversível** — sem prática, double-check com o Antifraude antes
   do Enter.
