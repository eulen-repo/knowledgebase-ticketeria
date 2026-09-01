# knowledgebase-ticketeria

Base de conhecimento **para atendimento / ticketeria** (agentes humanos e IA) da operação Eulen.
Contém material operacional curado e **sanitizado** a partir do repositório interno `eulen-docs`.

> [!CAUTION]
> **Procedência e sensibilidade.** Este conteúdo deriva de `eulen-docs`, o SSoT interno da Eulen, que
> **contém informação sensível** — dados de negócio (volumes, receita, limites, estratégia), dados de
> parceiros/processadoras (nomes, PII, integrações) e incidentes com detalhe sensível (fraude, valores,
> segurança, infraestrutura). **Nada disso deve entrar aqui.** Cada documento passou por curadoria e
> redação manual: nomes de parceiros/processadoras foram generalizados, PII/credenciais/hosts internos
> e números de negócio foram removidos, e detalhes sensíveis de incidentes foram anonimizados.

## Como esta base foi montada

- **Origem:** `eulen-docs` → `operations/`, `glossary.md`, `processes.md`.
- **Modo:** *curar + redigir*. Documentos úteis ao atendimento entraram; dentro deles, tudo que
  identifica parceiro/processadora, expõe número de negócio, PII, credencial, host de infraestrutura ou
  metodologia/estado de antifraude sensível foi **generalizado ou removido**, **sem** alterar as
  instruções operacionais.
- **Data da curadoria:** 2026-09-01.

### Convenções de redação usadas

- Processadoras e nós bancários específicos → **"a processadora" / "o banking node"**.
- Nomes de parceiros, pessoas e leads → papel genérico ou removidos.
- Volumes/receita/contagens agregadas de plataforma → removidos; **limites e taxas de produto**
  (partner-facing) foram mantidos por serem regra operacional.
- Hosts internos, nomes de segredos, caminhos de credencial → removidos.
- Links para incidentes/reuniões internas → removidos; a lição genérica foi preservada.

## Índice

### Atendimento
| Documento | Resumo | Palavras-chave |
|-----------|--------|----------------|
| [Playbook de atendimento ao parceiro](atendimento/atendimento-parceiro-playbook.md) | O que o parceiro pede e como responder; régua de escalonamento. | playbook, atendimento, escalonamento, double-check |
| [FAQ de atendimento](atendimento/faq-atendimento.md) | Folha de respostas: os três limites, taxas, `reason_code`, QR estático, paridade 1:1, o que não dizer. | faq, limites, taxas, reason_code, qr, paridade |
| [Manual Operacional do Bot Pix](atendimento/manual-operacional-bot-pix.md) | Mecânica completa: identificadores, status, triagem, todos os comandos, playbooks. | bot, comandos, /refund, /show, saque, triagem, help |
| [Sistema de Tickets & SLA](atendimento/sistema-tickets-sla.md) | Objeto ticket (ATD/ENG/DEM), campos, enums, metas de SLA e como é medido. | ticket, sla, atd, eng, dem, metas |
| [Contestação de MED — guia operacional](atendimento/med-contestacao-guia-operacional.md) | O que é MED 2.0, fluxo, o que dizer ao parceiro, FAQ. | med, contestação, devolução, dict, bacen |
| [Comunicado de incidente ao parceiro](atendimento/comunicado-de-incidente.md) | Formato fixo do comunicado no grupo de avisos; o que o corpo pode dizer. | incidente, comunicado, broadcast |
| [Comunicado — 3 mudanças obrigatórias](atendimento/comunicado-parceiros-3-mudancas-obrigatorias.md) | Identificação do pagador (`euid`/CPF), `merchantId`, delay obrigatório. | integração, euid, merchantId, delay, api |
| [Depósito em chave estática](atendimento/deposito-em-chave-estatica.md) | Quem é o dono do dinheiro num pagamento em chave estática; armadilhas. | chave estática, metadata.code, dono, atribuição |
| [Documentação de saques](atendimento/withdraw-docs.md) | Catálogo de erros conhecidos do saque. | saque, erro, timeout, invalid_deposits |
| [Armadilhas de consulta (pix2depix)](atendimento/armadilhas-de-consulta-pix2depix.md) | Consultas que devolvem menos linha em vez de erro. | sql, uuidv7, tx_date, e2e, schema |
| [Listas de controle ("whitelist")](atendimento/listas-de-controle.md) | As seis listas que atendem por "whitelist" e para que lado cada uma falha. | whitelist, blocklist, controle, modo manual |
| [Runbooks — porta por sintoma](atendimento/runbooks.md) | Índice por sintoma para as demais docs. | runbook, sintoma, índice |
| [E-mails de MED (aviso/lembrete/encerramento)](atendimento/emails/med-aviso-lembrete-encerramento.md) | Templates dos três e-mails de MED ao parceiro. | email, med, template, aviso |
| [Resposta ao formulário do site](atendimento/emails/resposta-form-depix.md) | Template de resposta a quem preencheu o formulário. | email, formulário, lead |

### Estornos
| Documento | Resumo | Palavras-chave |
|-----------|--------|----------------|
| [Estorno de depósito](estornos/estorno-de-deposito.md) | Modelo de dados e como provar um estorno; estados de saque. | estorno, depósito, refund, prova, estados |
| [Estorno de saque: quanto devolver](estornos/estorno-de-saque-quanto-devolver.md) | Os dois casos e o portão de estado. | estorno, saque, refundwithdraw, taxa |

### Onboarding
| Documento | Resumo | Palavras-chave |
|-----------|--------|----------------|
| [Processo de onboarding](onboarding/onboarding-process.md) | Captação → qualificação → grupo → teste R$10 → KYC. | onboarding, lead, kyc, r$10 |
| [Adicionar um parceiro](onboarding/add-partner.md) | Uso manual de `/addpartner`; obter o group-id. | addpartner, group-id, ativar |
| [Criar grupo de parceiro no Telegram](onboarding/create-partner-group-chat.md) | Criar o grupo e configurar o `config.yaml`. | telegram, grupo, config.yaml |
| [Criar um bot no Telegram](onboarding/create-telegram-bot.md) | BotFather: criar bot, token, testar. | botfather, token, bot |
| [Antifraude — sinais de risco na entrevista](onboarding/antifraude-sinais-de-risco.md) | Checklist anonimizado para ler entrevista de lead pela lente de risco. | antifraude, risco, estruturação, checklist |

### Observabilidade
| Documento | Resumo | Palavras-chave |
|-----------|--------|----------------|
| [Buscador universal (Metabase)](observabilidade/metabase-buscador-universal.md) | Cola qualquer identificador e lista todas as ocorrências. | buscador, identificador, e2e, cpf, uuid |
| [Armadilhas do Metabase](observabilidade/metabase-armadilhas.md) | Teto de 2.000 linhas, fuso, status 202. | metabase, api, timezone, armadilha |

### Base
| Documento | Resumo |
|-----------|--------|
| [Glossário](glossario.md) | Vocabulário operacional (DePix, MED, EUID, EMID, QR Delay, KYC/KYB…). |
| [Processos de atendimento](processos.md) | Como uma demanda entra, é acompanhada e escala. |

## Notas de curadoria

- **Verbatim (já seguros na origem, copiados sem edição):** comunicado-de-incidente,
  comunicado-parceiros-3-mudancas, estorno-de-saque, metabase-armadilhas, create-partner-group-chat,
  create-telegram-bot, antifraude-sinais-de-risco.
- **Redigidos (sanitizados a partir do original):** todos os demais.
- **Antifraude — sinais de risco:** é metodologia (embora escrita como material anonimizado). Reveja se
  quer critérios de detecção de fraude numa base pesquisável por IA antes de dar acesso amplo.
- **Links quebrados** para o `eulen-docs` original foram substituídos por referências internas desta
  base ou removidos.

## O que **não** entrou

Ficaram de fora, por não serem sanitizáveis sem perder o essencial ou por não serem conteúdo de
atendimento: metodologia de rastreio on-chain de antifraude; método de identificação de dono de parceiro;
dashboards com métricas de negócio (mapa/alertas/cockpit de risco); volume da plataforma; modelo/roadmap
do CRM interno; guia com conta operacional + localização de senha 2FA; notas internas de engenharia do
assistente; arquitetura interna de bots/infra; e os registros nominais de MED.
