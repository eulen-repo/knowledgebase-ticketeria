# knowledgebase-ticketeria

Base de conhecimento **para atendimento / ticketeria** (agentes humanos e IA) da operação Eulen.
Contém apenas material operacional **genérico e seguro**, curado a partir do repositório interno
`eulen-docs`.

> [!CAUTION]
> **Procedência e sensibilidade.** Este conteúdo foi extraído de `eulen-docs`, o SSoT interno da
> Eulen, que **contém informação sensível** — dados de negócio (volumes, receita, limites,
> estratégia), dados de parceiros (nomes, PII, integrações), e incidentes com detalhe sensível
> (fraude, valores, segurança, infraestrutura). **Nada disso deve entrar aqui.** Cada documento
> abaixo passou por uma curadoria manual: só entrou o que **já era genérico e seguro**, sem editar
> o conteúdo. Ver [§ O que foi filtrado](#o-que-foi-filtrado).

## Como esta base foi montada

- **Origem:** `eulen-docs` → `operations/`, `glossary.md`, `processes.md` (escopo definido para a
  ticketeria).
- **Regra de filtragem:** *só curar documentos já seguros*. Um documento só entra se **não** menciona
  parceiro específico, valor/número de negócio, PII, credencial, host de infraestrutura, metodologia
  de antifraude sensível, ou detalhe de incidente sensível. Documentos que misturam conteúdo útil com
  qualquer um desses itens foram **descartados inteiros** (não foram editados/redigidos).
- **Data da curadoria:** 2026-09-01.

## Índice

| # | Documento | Área | Resumo | Palavras-chave |
|---|-----------|------|--------|----------------|
| 1 | [Comunicado de incidente ao parceiro](atendimento/comunicado-de-incidente.md) | Atendimento | Formato fixo do comunicado de incidente no grupo de avisos; o que o corpo pode e não pode dizer; onde publicar. | incidente, comunicado, broadcast, parceiros, status |
| 2 | [Comunicado — 3 mudanças obrigatórias na integração](atendimento/comunicado-parceiros-3-mudancas-obrigatorias.md) | Atendimento / Integração | Comunicado oficial (já enviado aos parceiros) sobre identificação do pagador (`euid`/CPF), `merchantId` e delay obrigatório em pagamentos de merchant. | integração, euid, merchantId, delayDepixInHours, MED, antifraude, API |
| 3 | [Estorno de saque: quanto devolver](estornos/estorno-de-saque-quanto-devolver.md) | Estornos | Diferença entre "PIX nunca saiu" (devolve DePix cheio, `/refundwithdraw`) e "PIX saiu e voltou" (devolve o valor em reais, taxa retida); portão de estado do saque. | estorno, saque, refundwithdraw, devolução, taxa, estados |
| 4 | [Criar grupo de parceiro no Telegram](onboarding/create-partner-group-chat.md) | Onboarding | Como criar o grupo Telegram do parceiro, obter o `telegram-group-id` e configurar o `config.yaml`. | telegram, grupo, onboarding, config.yaml, group-id |
| 5 | [Criar um bot no Telegram](onboarding/create-telegram-bot.md) | Onboarding | Guia genérico de BotFather: criar bot, obter/regenerar token, testar via API. | telegram, botfather, bot token, getMe, onboarding |
| 6 | [Antifraude — sinais de risco em entrevista de onboarding](onboarding/antifraude-sinais-de-risco.md) | Onboarding / Risco | Checklist anonimizado para ler entrevista de lead pela lente de antifraude; sinais de atenção/positivos, estruturação, comunicação. | antifraude, onboarding, risco, estruturação, smurfing, checklist |
| 7 | [Armadilhas do Metabase ao rodar consulta](observabilidade/metabase-armadilhas.md) | Observabilidade | Teto de 2.000 linhas do `/api/dataset`, fuso `America/Sao_Paulo` em `to_timestamp`, status `202` como sucesso. | metabase, sql, api, timezone, armadilha, query |

## Notas de curadoria por documento

- **Doc 3 (estorno de saque):** contém referências a caminhos de código-fonte interno (`src/pkg/...`)
  e um exemplo numérico ilustrativo. Sem parceiro, sem valor de negócio real.
- **Doc 6 (antifraude sinais de risco):** é **metodologia de antifraude** (embora escrito como material
  genérico/anonimizado pela própria equipe). Reveja se você quer critérios de detecção de fraude numa
  base pesquisável por IA antes de publicar.
- **Links quebrados são esperados:** vários documentos mantêm links relativos para o `eulen-docs`
  original (ex.: `../glossary.md`, `../compliance/...`). Eles apontam para o repositório privado e
  **não** existem aqui — foram preservados por não editarmos o conteúdo. Não seguem para conteúdo que
  esteja nesta base.

## O que foi filtrado

Do escopo pedido (`operations/` = 64 arquivos + `glossary.md` + `processes.md`), **7 entraram** e o
restante foi descartado. Motivos mais comuns:

- **Parceiro/processadora específica** citada (Sqala, Fitbank, Xend, Transfero, TechBnk, Deflow, Plebz,
  Joltz, Satsails, etc.) — ex.: `atendimento-parceiro-playbook`, `withdraw-docs`, `sqala-known-errors`,
  `mike-faq`, `manual-operacional-bot-pix`, `sistema-tickets-sla`, `med-contestacao/*`.
- **Valores/números de negócio** (volume da plataforma, receita, MEDs por parceiro, limites) — ex.:
  `estorno-de-deposito`, `metabase-*` (mapa, alertas, dashboards, risks-cockpit, buscador),
  `volume-da-plataforma`, `dono-do-parceiro-metodo`.
- **PII real de leads/parceiros** — ex.: `onboarding/approve-lead-to-partner` (nomes de empresas,
  pessoas e handles reais).
- **Credencial / conta operacional / infraestrutura** — ex.: `onboarding/add-telegram-account`
  (conta operacional específica + onde mora a senha 2FA), `zammad-n0-assistant` (nomes de segredos,
  endpoints internos, caminhos AWS).
- **Metodologia sensível de antifraude** — ex.: `chain-analysis` (rastreio on-chain de MED).
- **Notas internas de engenharia / projeto** fora de escopo de atendimento — ex.:
  `eulen-assistant-docs/*`, `mvp-onboarding-ai`, `telegram-multi-bot-coexistence`.

### `glossary.md` e `processes.md` (pedidos, porém retidos)

Foram pedidos no escopo, mas **não passaram** na regra "só curar docs seguros": o `glossary.md` cita
parceiros e enquadra risco de negócio, e o `processes.md` nomeia donos/parceiros. Como o modo escolhido
foi curar **sem editar**, eles não foram copiados. Opções para incluí-los:
1. produzir versões **redigidas/sanitizadas** (modo "curar + redigir"); ou
2. escrever um **glossário novo, nativo desta base**, só com termos genéricos de produto (DePix, Pix,
   MED, EUID, QR, estorno, saque, depósito).

Qualquer uma pode ser feita sob demanda.
