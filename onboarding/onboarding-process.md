# Onboarding de Parceiros (B2B) — processo

## Visão Geral

O processo de **onboarding de parceiros B2B** tem como objetivo identificar, qualificar e integrar
novos parceiros interessados em usar a infraestrutura **Eulen / DePix** para projetos próprios,
operações P2P ou plataformas.

**Objetivo principal:** garantir **segurança, autenticidade e alinhamento comercial** antes de liberar
qualquer integração técnica ou parceria formal.

---

## 1. Captação do Parceiro

Os parceiros chegam **exclusivamente** pelos formulários dos sites oficiais (eulen.app / depix.info).

**Campos obrigatórios do formulário:** nome completo do solicitante · e-mail válido · handle do
Telegram ativo · descrição inicial do interesse/projeto.

Recebimento e triagem: os formulários caem num Kanban no CRM (lista "Novos Leads B2B"), e um operador
faz a triagem rápida (coerência, duplicações, relevância).

---

## 2. Primeiro Contato (Telegram)

Mensagem padrão (via a conta operacional de atendimento):

```
Olá, tudo bem? Eu sou da eulen.app / depix.info!

Recebemos um formulário com seu Telegram ID:
"[COLAR COMENTÁRIO]"

Confirma que foi você?
```

**Aguardar confirmação explícita.**

---

## 3. Qualificação Inicial

```
Pretende usar DePix para:
-  Uso próprio (PF)
-  P2P/plataforma ativa
-  Começar operação estruturada?
```

**Somente B2B avança.**

---

## 4. Pré-Reunião

1. Apresentação da empresa/token.
2. **Agendar reunião** (30 min).

---

## 5. Reunião (30 min)

Perguntas obrigatórias:
1. Como conheceu Eulen/DePix?
2. Qual projeto deseja estruturar?
3. Já comprou DePix?
4. Volume esperado?

Fechamento: "Quer criar grupo Telegram?"

---

## 6. Grupo + Teste Inicial

1. **Criar grupo Telegram** (automatizado): adiciona as **contas operacionais** de atendimento e de
   antifraude como admins, cria o tópico `bot` do fórum e adiciona o parceiro.
2. **Cadastro R$10** (teste).
3. **1ª transação** → **KYC-mínimo automático** (ver 6.1).

### 6.1. KYC-mínimo automático — dados coletados

A 1ª transação de teste dispara a **captura automática da identidade do pagador**, extraída da
confirmação do bot DePix — nada é digitado à mão. Os dados vão para o CRM:

| Dado | Origem | Observação |
|---|---|---|
| Nome do pagador | confirmação do bot DePix | — |
| CPF/CNPJ | confirmação do bot DePix | **mascarado** (padrão Receita, ex.: `***.456.789-**`) |
| EUID | confirmação do bot DePix | id Eulen do parceiro (`EU…`) |
| Bank ID | confirmação do bot DePix | identificador no banco/processador |
| Endereço Liquid | mensagem do parceiro no grupo | `lq1…` / `VJ…` / `VL…` |
| Nome · e-mail · @Telegram · grupo | formulário + operador | registrados no início do onboarding |

> A identidade vem da **confirmação bancária do pagamento de teste** (CPF/CNPJ já mascarado), não de
> coleta manual de documentos. Em alguns casos ela é complementada por verificação num **provedor de
> validação** externo.

---

## 7. Onboarding Concluído

Após a 1ª transação aprovada: grupo Telegram ativo · CRM marcado "Onboarding Concluído" · documentos
técnicos liberados.

---

## 8. Fluxo Visual

```
Formulário → CRM → Telegram → {Confirma?}
  ↓ SIM                     ↓ NÃO
Qualificação → Reunião → Grupo → R$10 → 1ª Transação(KYC) → ✅ ONBOARDING
```

---

## 9. Regras Críticas

| Situação | Ação |
|----------|------|
| Sem confirmação de ID | ❌ Arquivar |
| Apenas uso pessoal | Qualificar PF |
| Sem resposta (7d) | "Sem Retorno" |
| 1ª transação falha | Máx. 2 tentativas |

**O operador deve registrar tudo no CRM.**

---

## 10. KPIs

| Métrica | Meta |
|---------|------|
| Tempo total | ≤ 5 dias |
| Reunião → Onboarding | ≥ 60% |
| 1ª transação OK | ≥ 90% |

---

_Relacionado: [Criar grupo de parceiro](create-partner-group-chat.md) · [Criar um bot](create-telegram-bot.md) · [Antifraude — sinais de risco](antifraude-sinais-de-risco.md)._
