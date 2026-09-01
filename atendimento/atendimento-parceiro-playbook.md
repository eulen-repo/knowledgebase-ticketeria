# Playbook de Atendimento ao Parceiro — Operação Pix

> [!NOTE]
> Este doc é a **camada de comunicação**: o que o parceiro costuma pedir e como responder. A
> **mecânica do bot** (sintaxe, status, validações) está no
> [Manual Operacional do Bot Pix](manual-operacional-bot-pix.md); as respostas-padrão de FAQ estão
> no [FAQ de atendimento](faq-atendimento.md).

> [!IMPORTANT]
> Enquanto você não tiver prática, **todo comando que mexe em dinheiro passa por double-check** com
> o responsável de Antifraude antes do Enter (ver [§4](#4-escalonamento-o-que-fazer-sozinho-vs-escalar)
> e o [Manual §10](manual-operacional-bot-pix.md)).

---

## Índice

1. [Princípios](#1-princípios)
2. [Pedidos comuns → o que fazer](#2-pedidos-comuns--o-que-fazer)
3. [Quando o parceiro só manda "dá uma olhada"](#3-quando-o-parceiro-só-manda-dá-uma-olhada)
4. [Escalonamento: o que fazer sozinho vs escalar](#4-escalonamento-o-que-fazer-sozinho-vs-escalar)
5. [Docs relacionadas](#5-docs-relacionadas)

---

## 1. Princípios

> [!NOTE]
> **Onde o atendimento acontece:** nos grupos B2B, a comunicação com o parceiro é pela conta
> operacional (userbot **Mike**) e os comandos do parceiro (`/qr` etc.) pelo bot público de
> parceiro. Os comandos de **operação** (devolução, aprovação, etc.) ficam no bot de operação
> privado, no grupo de log — ver [Manual §2](manual-operacional-bot-pix.md).

- **Responder já acalma.** Mesmo antes de resolver, dar uma resposta ("tô verificando, já te
  retorno") reduz muito a ansiedade do parceiro e tira o sufoco do time.
- **Identifique antes de agir.** Todo atendimento começa pela triagem (Manual §5): que código é
  esse? Depósito ou saque? Qual o status?
- **Confirme tudo.** Conferir duas vezes (carteira, status, ID) evita o erro irreversível. No
  início, **double-check** com o Antifraude nos comandos críticos.
- **O parceiro só negocia; a Eulen faz antifraude e compliance.** Pedidos que envolvem flexibilizar
  trava, limite ou bloqueio passam pelo critério da operação, não são automáticos.

---

## 2. Pedidos comuns → o que fazer

| O parceiro pede… | Verificar | Ação no bot | O que responder |
|---|---|---|---|
| **"Qual o status desse pagamento?"** | Identificar o tipo do código (Manual §3) | Depósito: `/show <id>`. Saque: Metabase | Informar o estado (recebido / DePix enviado / `under review` + motivo) |
| **"O cliente não recebeu o DePix"** | É a 1ª compra? Carteira pode esconder o ativo por padrão | Confirmar envio via TXID/explorador (Manual §9) | Orientar a **ativar a visualização do DePix** na carteira; se 2ª compra+, investigar |
| **"Devolve esse pagamento" (depósito)** | Confirmar que a devolução é legítima | `/refund <banktxid> [force]` — **double-check** | Confirmar a devolução (o grupo é notificado automaticamente, com comprovante) |
| **"Deu erro de chave Pix / o saque não caiu"** | Status `3` no Metabase (Manual 7.1) | Reenviar: `/retrywithdrawal`. Devolver: `/refundwithdraw <id> <endereço>` — **double-check** | Pedir a chave correta (reenvio) **ou** o endereço Liquid de devolução. Não dá para editar o saque — tem que refazer |
| **"Aumenta o limite do meu cliente para R$ X"** | É o limite de **novos usuários** do parceiro | `/setnewuserslimit <partnerid> <centavos>` (R$ 500 = `50000`) | Confirmar o novo limite. Lembrar que acima do teto exige identificação/DeleID |
| **"Bloqueia esse CPF"** | O parceiro pode bloquear clientes dele | `/blockenduser <euid\|cpf>` | Confirmar o bloqueio |
| **"A transação está travada (`under review`)"** | Ver o **motivo** (Manual §4.1) | Conforme o motivo; escalar se houver dúvida | Explicar o motivo (divergência de pagador, limite diário, alta frequência, CPF/ISPB bloqueado) |
| **"Quero transacionar acima de R$ 6.000/dia"** | Acima do limite diário | — (não é comando do bot) | Direcionar para o **canal de OTC** (mesa interna) — ver [FAQ](faq-atendimento.md) |
| **"Manda o comprovante desse saque"** | Buscar o **TXID** no Metabase de saques | — | Enviar o comprovante/print do envio confirmado |
| **"Quero uma chave Pix fixa/estática"** | Limitação BACEN (máx. 20 chaves/conta) | QR estático (ver comandos de QR estático) | Explicar a limitação + a solução de QR estático — ver [FAQ](faq-atendimento.md) |
| **"Recebi um MED / contestação nesse pagamento"** | É **contestação bancária** (Pix), **não** devolução operacional | — (não é comando do bot) | Caso de **compliance/Antifraude**: a régua que decide devolver ou contestar segue o processo interno de devolução em caso de MED. **Não** disparar `/refund` sem passar por essa decisão. Ver [contestação de MED](med-contestacao-guia-operacional.md) |
| **"Esse cliente é confiável?" / "dá uma olhada nesse CPF"** | É pedido de **triagem antifraude**, não de status | `/af_report <cpf\|cnpj\|euid>` (alias `/screen`) — o PDF vem **por DM**, exige sessão GPG | **Não repassar o dossiê ao parceiro**: é dado de terceiro (LGPD). Responder só o veredito operacional — se segue, se fica sob análise, se não segue |

---

## 3. Quando o parceiro só manda "dá uma olhada"

Acontece muito: o parceiro cola um código (ou print) e diz só *"dá uma olhada"* / *"resolve aí"*,
sem dizer o que quer. **Não adivinhe.** Peça o mínimo para agir:

1. **O identificador** (E2E, TXID, QrId ou ID de saque) — se veio em imagem, transcreva.
2. **O que ele quer**: status? devolução? reenvio? comprovante?
3. **Para quem** (se for sobre um cliente final): CPF ou euid.

Com isso você roda a triagem (Manual §5) e responde — ou escala, se for caso crítico.

---

## 4. Escalonamento: o que fazer sozinho vs escalar

| Faixa | O que entra | Conduta |
|---|---|---|
| 🟢 **Sozinho** (após pegar prática) | Consultas (`/show`, `/userinfo`, `/check`, `/report`, Metabase), orientar carteira, responder status, pedir contexto | Pode tocar direto |
| 🟡 **Double-check** (sempre, no início) | `/refund`, `/refundwithdraw`, `/retrywithdrawal`, `/approve<banking-node>` (inclusive com `force`), `/setauto`, `/deletewithdrawal` | Resumir o caso para o Antifraude e **confirmar antes do Enter** |
| 🔴 **Acionar o time** | Transação que deveria existir e **não aparece** no explorador; suspeita de fraude/golpe; divergência sem explicação; qualquer coisa fora dos playbooks | **Não resolver sozinho** — escalar |

A régua existe porque **todo comando é irreversível** (Manual §10). Na dúvida, suba de faixa.

> [!IMPORTANT]
> **Não confie na ausência de bloqueio automático.** Um depósito que passou sem trava não
> necessariamente foi triado — pode só não ter sido barrado. Quando o caso cheirar mal, rode o
> `/af_report` você mesmo em vez de assumir que o sistema já olhou.

---

## 5. Docs relacionadas

- [Manual Operacional do Bot Pix](manual-operacional-bot-pix.md) — mecânica: comandos, status, triagem, Metabase, auditoria
- [FAQ de atendimento](faq-atendimento.md) — a folha de respostas ordenada pela demanda real dos tickets: limites, taxas, QR estático, `reason_code`, paridade 1:1 e o que não pode ser dito ao parceiro
- [Contestação de MED — guia operacional](med-contestacao-guia-operacional.md) — quando entra uma contestação (MED)
- [Modelos de e-mail](emails/) — templates de resposta por e-mail
