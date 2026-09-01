# Buscador Universal (Metabase)

Ferramenta de investigação do time de operações/suporte: **cola-se qualquer identificador** num único
filtro e o painel lista **todas as ocorrências** na base transacional, com contexto legível e links
para a linha de origem e para o explorador da Liquid. Um card só reconhece automaticamente **14
famílias de input**.

> [!NOTE]
> O link preserva a busca, então dá para **compartilhar uma investigação** colando a URL.

## O que dá para colar (famílias de input)

| Família | Formato/exemplo | Observações |
|---|---|---|
| **UUID** | `97bc8015-44fc-…`, sem traços, MAIÚSCULO, `{chaves}`, `urn:uuid:` | normalizado; acha tx master, QR, saque, auth, refund, extrato provider… |
| **EndToEndId (E2E)** | `E…`/`D…` 32 chars | aceita **1ª letra minúscula**; o miolo continua case-sensitive (cole como veio do comprovante) |
| **Hash on-chain** | 64-hex (Liquid) | aceita maiúsculas; um txid de **lote** retorna todas as transações do lote |
| **`bank_tx_identifier`** | `<rail>_E…` ou cru | colar o E2E cru também alcança os prefixados |
| **Endereço Liquid** | `lq1…`, `VJL…`, `ex1…` (e `ark1`/`bc1`) | resumo com nº de envios, **R$ total em depósitos**, período, parceiros + envios recentes + QRs pendentes ativos |
| **Handle DePix** | ex.: `<handle>` | mesmo fluxo do endereço |
| **CPF/CNPJ** | 11/14 dígitos, com ou sem pontuação | resumo de depósitos do pagador, saques, usuário Eulen (euid), chaves Pix, MEDs, **blocklist** (⛔ se bloqueado) |
| **Chave Pix** | EVP, CPF, CNPJ, email, fone (com/sem `+55`) | saques por chave + dono da chave |
| **EUID** | `EU` + 15 dígitos | usuário Eulen + QRs ligados (com o pagador esperado) + divergências de pagador |
| **Nome de pessoa** | 2+ palavras | **acento- e caixa-insensível** (`joao goncalves` acha `João Gonçalves`) |
| **Grupo/usuário Telegram** | `-100…` ou id numérico | parceiro dono do grupo, whitelist, auths |
| **Nº de documento da MED** | ex. `1023999286` | acha a MED direto |
| **`conciliation_id`** | uuid, `FFFF…` (HEX35) ou `2171TG…` | QR e reembolso |
| **Outros** | token de API (valor exato), EMV copia-e-cola (`000201…`), key de withdraw_request (`grupo_msgid`) | |

## O que volta (anatomia do resultado)

Colunas na ordem: **nº · id · contexto · criado_em · tabela · coluna · on-chain**. A tabela é uma
**linha do tempo**: o **evento mais de cima é o mais recente da cadeia**.

- **nº** — posição do evento; **nº 1 = mais recente** (topo).
- **🔗 id** — clicável: abre a **linha na tabela de origem**. Linhas de resumo não linkam de propósito.
- **contexto** — resumo legível com **parceiro**, valores em R$, e **estados por nome** (saque:
  `CRIADO/ENVIANDO/ENVIADO/ERRO/CANCELADO/SUBSTITUÍDO/DEVOLVIDO`; DePix:
  `ENVIANDO/ENVIADO/CANCELADO`). `⚠️ MANUAL` só aparece quando é modo manual.
- **linhas 📊 resumo** — agregados (endereço, CPF, nome): contagem, R$ total, período, parceiros; o
  detalhe é **capado** (⚠️ avisa quando mostra só os N mais recentes).
- **⛓️ on-chain** — tx no explorador da Liquid.
- **transação (linha única)** — para qualquer perna colada, uma linha `🧭 transação · status externo=…`
  mostra **o que o parceiro vê** pela API (`pending`/`delayed`/`sent`/`unsent`…) — resposta padrão para
  o ticket "cadê meu depósito" quando há delay de política.

## Latências esperadas (sem índice na réplica)

| Busca | Tempo |
|---|---|
| E2E, CPF, chave, telegram, conciliation, doc MED | ~1–2 s |
| Endereço/handle | ~2–6 s |
| UUID | ~4–6 s |
| Nome | ~13 s |
| EUID | ~15 s |
| EMV | ~20 s |

A base é **read-replica** (não dá para criar índice por lá). Com índices no primário tudo cai para
sub-segundo.

## Limitações conhecidas

- **QRs expirados/cancelados não são buscáveis por endereço** (só os pendentes **ativos**; os pagos
  aparecem pelo fluxo `sending_depix`).
- CPF de **QR não pago** fora — pagos são cobertos via `bank_tx`; tentativas aparecem via `attempts`.
- **E2E com o miolo em caixa trocada** não casa (só a 1ª letra é normalizada).
- UUIDs **dentro de JSON** (`metadata`, payloads de provider, blobs `*_json`) não são varridos.
- Input de **1 caractere** não busca (mínimo 2).

---

_Relacionado: [Armadilhas do Metabase](metabase-armadilhas.md) ·
[Sistema de Tickets & SLA](../atendimento/sistema-tickets-sla.md)._
