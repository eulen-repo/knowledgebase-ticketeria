# knowledgebase-ticketeria

Base de conhecimento **para o parceiro**, consultada por um **chatbot (LLM)** para responder às
dúvidas de integração e operação com o DePix.

> [!IMPORTANT]
> **O que esta base é — e o que ela não é.** Ela contém **apenas informação de referência** que o
> parceiro precisa: fatos sobre o produto, limites, taxas, integração e situações comuns. Ela **não**
> contém: instruções para o modelo, procedimentos internos, comandos de operação, detalhes de
> arquitetura ou de sistemas internos. Tudo aqui é escrito de forma **declarativa** (fatos), não como
> passo a passo operacional.

## Conteúdo

| Documento | Para responder sobre |
|-----------|----------------------|
| [Glossário](glossario.md) | O que é DePix, Liquid, QR dinâmico/estático, QR Delay, EUID, EMID, MED, E2E, paridade 1:1. |
| [Limites e Taxas](limites-e-taxas.md) | Teto por saque, tetos por QR (com/sem identificação), limite diário, taxas de depósito e saque. |
| [Integração / API](integracao-api.md) | Identificação obrigatória (`euid`/CPF, `merchantId`, delay), criar cobrança, consultar status, token de API, webhooks, QR estático. |
| [Situações Comuns](situacoes-comuns.md) | "DePix não chegou", saque com erro/travado, depósito expirado/pendente/comprovante enviado, transação em análise, diferença de valor, recebi menos, chave fixa, recebi um MED. |
| [Sobre a Eulen e o DePix](sobre-a-eulen.md) | Como o DePix é comercializado, identificação/privacidade, paridade, como declarar. |

Para especificação técnica completa e atualizada de campos e endpoints, a fonte é
**[docs.eulen.app](https://docs.eulen.app)**.
