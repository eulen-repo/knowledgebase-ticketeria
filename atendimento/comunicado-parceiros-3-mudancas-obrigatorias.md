# Comunicado Oficial — Três mudanças obrigatórias na integração Eulen

**Para:** Parceiros de integração (plataformas B2B)
**De:** Equipe Eulen
**Data:** 03/06/2026

> [!IMPORTANT]
> **Prazo de adaptação: 15 dias.** As três mudanças descritas abaixo passam a ser **obrigatórias a partir de 18/06/2026**. A partir dessa data, operações que não estiverem em conformidade serão recusadas. Recomendamos iniciar a adaptação assim que possível — o time de integração está à disposição durante toda a janela.

Prezado parceiro,

Estamos padronizando a **identificação das operações** e reforçando a **prevenção a fraudes** no fluxo Pix → DePIX. Para isso, três pontos que hoje são opcionais passarão a ser obrigatórios. Este comunicado explica o que muda, por que muda e o que você precisa fazer.

---

## Índice

1. [Resumo](#1-resumo)
2. [Por que estamos fazendo isso](#2-por-que-estamos-fazendo-isso)
3. [Mudança 1 — Identificação do pagador no QR](#3-mudança-1--identificação-do-pagador-no-qr)
4. [Mudança 2 — Identificação do merchant (`merchantId`)](#4-mudança-2--identificação-do-merchant-merchantid)
5. [Mudança 3 — Delay obrigatório em pagamentos de merchant](#5-mudança-3--delay-obrigatório-em-pagamentos-de-merchant)
6. [Cronograma](#6-cronograma)
7. [O que acontece após o prazo](#7-o-que-acontece-após-o-prazo)
8. [Suporte](#8-suporte)

---

## 1. Resumo

| # | Mudança | Aplica-se a | O que você passa a enviar | A partir de |
|---|---|---|---|---|
| 1 | **Identificação do pagador** | Cobranças de cliente final | `euid` (ou `endUserTaxNumber` = CPF) do cliente | 18/06/2026 |
| 2 | **Identificação do merchant** | Cobranças de merchant | `merchantId` = **EUID do merchant** | 18/06/2026 |
| 3 | **Delay obrigatório** | Pagamentos de merchant | piso de `delayDepixInHours` aplicado pela Eulen | 18/06/2026 |

---

## 2. Por que estamos fazendo isso

Estas mudanças atendem a **exigências do regulador e das instituições de pagamento** com as quais operamos, que vêm reforçando os requisitos de identificação e rastreabilidade nas transações. Adotamos uma referência **semelhante à já praticada no Paraguai**, onde a **identificação ocorre no momento do pagamento**.

No fluxo Pix → DePIX, a liquidação é praticamente imediata. Quando uma fraude é identificada **depois** da liquidação, não há mais o que reverter. Identificar quem paga e quem recebe, e introduzir uma janela mínima de segurança, é o que nos permite **interceptar a fraude antes da perda**.

Na prática, isso traz benefícios diretos para a sua operação:

- **Menos contestações e MEDs** (Mecanismo Especial de Devolução), que hoje pressionam tanto a sua operação quanto a relação de todos nós com o sistema bancário.
- **Resposta cirúrgica a fraudes:** com a identificação correta, conseguimos agir sobre o ponto problemático específico, **sem impactar operações legítimas** suas ou de outros merchants.
- **Previsibilidade:** regras claras e padronizadas, iguais para todos os parceiros.

---

## 3. Mudança 1 — Identificação do pagador no QR

**O que muda.** Toda cobrança destinada a um **cliente final (P2P)** passará a exigir a identificação do pagador. Não haverá mais cobranças sem identificação.

**O que você faz.** Ao criar a cobrança (`POST /deposit`), informe o **`euid`** do cliente (a identidade Eulen do usuário) — ou, na ausência de EUID, o **`endUserTaxNumber`** (CPF), acompanhado de `endUserFullName`. Hoje esses campos são **opcionais**; passam a ser **obrigatórios** em cobranças destinadas ao cliente final.

```jsonc
// POST /deposit — cobrança de cliente final: inclua a identificação do pagador
{
  "amountInCents": 50000,
  "euid": "<EUID do cliente>"          // ✅ ou: "endUserTaxNumber": "<CPF>" + "endUserFullName": "<nome>"
}
```

**Efeito.** A cobrança fica **vinculada ao pagador identificado**, dificultando que um terceiro pague no lugar da vítima — proteção direta contra golpes de redirecionamento.

> [!NOTE]
> Clientes ainda sem EUID podem ser identificados pelo onboarding (assinatura via micro-PIX), que gera o EUID e o devolve ao seu sistema. Fale com o time de integração para habilitar o fluxo, se ainda não o utiliza.

---

## 4. Mudança 2 — Identificação do merchant (`merchantId`)

**O que muda.** Toda cobrança ou pagamento de um **merchant** passará a exigir a identificação do merchant que recebe.

**O que você faz.** Informe o **EUID do merchant** no campo **`merchantId`** em todas as cobranças daquele merchant. O campo `merchantId` já existe na API (hoje **opcional**) e passa a ser **obrigatório** para cobranças de merchant.

```jsonc
// POST /deposit — cobrança de merchant: inclua o EUID do merchant
{
  "amountInCents": 150000,
  "merchantId": "<EUID do merchant>"   // ✅ obrigatório
}
```

**Como obter o EUID do merchant.** O EUID do merchant é gerado no **onboarding do merchant** (a mesma identidade EUID, aplicada a quem recebe) e devolvido ao seu sistema. Você o armazena e passa a enviá-lo em `merchantId`. Detalhes em [docs.eulen.app](https://docs.eulen.app).

**Efeito.** Com cada merchant identificado, problemas pontuais podem ser tratados **no merchant específico**, sem afetar os demais merchants da sua plataforma.

---

## 5. Mudança 3 — Delay obrigatório em pagamentos de merchant

**O que muda.** Hoje o atraso da conversão é opcional e definido por você, pelo parâmetro **`delayDepixInHours`** (1 a 720 horas); se omitido, a conversão é imediata. A partir da data de corte, para **pagamentos de merchant** a Eulen passa a aplicar um **piso mínimo de atraso, obrigatório**, conforme o **perfil de risco** de cada merchant. A liquidação imediata deixa de estar disponível para merchants.

**O que você faz.** Você pode continuar enviando `delayDepixInHours` (e **aumentar** o atraso, se quiser), mas **não poderá ficar abaixo do piso** definido pela Eulen para aquele merchant.

**Como funciona, na prática:**

- A janela varia tipicamente de **D+1 a D+7**, conforme o histórico do merchant.
- **Bom histórico reduz a janela**; aumento de contestações/MEDs aumenta a janela.
- Ao fim da janela, a liquidação ocorre **automaticamente**.
- Merchants com risco elevado podem ter a operação **suspensa para revisão**.

> [!NOTE]
> O delay é uma janela de proteção: é nela que conseguimos devolver um valor fraudado **antes** da perda. Ele protege o merchant e a saúde da sua operação como um todo.

---

## 6. Cronograma

| Etapa | Data |
|---|---|
| Publicação deste comunicado | 03/06/2026 |
| Janela de adaptação (suporte ativo do time de integração) | 03/06 → 17/06/2026 |
| **Mudanças obrigatórias em produção** | **18/06/2026** |

As mudanças **já podem ser adotadas desde já** — você não precisa esperar o prazo final para começar.

---

## 7. O que acontece após o prazo

A partir de **18/06/2026**:

- Cobranças de cliente final **sem `euid` ou `endUserTaxNumber`** serão recusadas.
- Cobranças de merchant **sem `merchantId`** serão recusadas.
- Para merchants, passa a valer o **piso de atraso** definido pela Eulen — a liquidação imediata deixa de estar disponível.

As recusas retornam um **erro claro e legível**, indicando o campo ausente, para facilitar o ajuste.

---

## 8. Suporte

Estamos à disposição durante toda a janela de adaptação para tirar dúvidas e apoiar a sua equipe técnica:

- **Seu grupo de suporte B2B** (Telegram) — canal mais rápido.
- **Documentação técnica de integração** (campos e endpoints exatos do seu ambiente): [docs.eulen.app](https://docs.eulen.app)

Conte com a gente para uma transição tranquila.

Atenciosamente,
**Equipe Eulen**
eulen.app
