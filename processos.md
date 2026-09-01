# Processos de Atendimento

Como uma demanda de atendimento entra, como é acompanhada e para onde escala. Versão sanitizada e
recortada para a ticketeria — a estrutura organizacional interna, os donos por área, contratações,
cultura e ferramentas internas foram deixados de fora.

## O contrato compartilhado

Vale independentemente da ferramenta que cada frente usa:

- **Dois dias úteis** para rotear qualquer coisa que chega. Recusar, despriorizar ou pedir mais
  tempo contam como rotear; silêncio não conta.
- Conversa (chat/DM/WhatsApp) **não** é processo. Quem recebe um pedido por ali converte para a
  porta certa.
- Cair na área errada não é problema de quem pediu. Quem recebe redireciona.
- Um *quick win* não vira papelada. Se é mais rápido fazer do que registrar, faça.

## Suporte e atendimento (Customer & Operations)

O atendimento é a área com o único SLA formal da operação. As responsabilidades e os documentos
que as sustentam:

| Responsabilidade | Onde está o processo/guia |
|---|---|
| Comercial: prospecção, qualificação, pipeline, fechamento | fluxos de leads (interno) |
| Onboarding, coordenando o gate de Risco e a integração de Tech | [processo de onboarding](onboarding/onboarding-process.md) |
| Suporte e escalações urgentes | [playbook de atendimento ao parceiro](atendimento/atendimento-parceiro-playbook.md) · [manual do bot Pix](atendimento/manual-operacional-bot-pix.md) |
| Triagem da demanda que chega | [sistema de tickets e SLA](atendimento/sistema-tickets-sla.md) |
| Ouvidoria e canais de reclamação: entrada, resposta, fechamento | *processo em compliance (interno)* |
| Handoff de casos de fraude para Risco & Compliance; escalação | ver [§ Escalação](#escalação-e-handoffs) |

### Onboarding (mecânica)

- [Processo de onboarding](onboarding/onboarding-process.md) — a visão de ponta a ponta.
- [Ativar um parceiro](onboarding/add-partner.md) · [criar o grupo](onboarding/create-partner-group-chat.md) · [criar um bot](onboarding/create-telegram-bot.md).
- [Antifraude — sinais de risco na entrevista](onboarding/antifraude-sinais-de-risco.md) — a lente
  de risco antes da decisão de viabilidade.

### Encontrar dados e responder

- [Buscador universal no Metabase](observabilidade/metabase-buscador-universal.md) — "onde este
  identificador aparece?".
- [Armadilhas de consulta no pix2depix](atendimento/armadilhas-de-consulta-pix2depix.md) e
  [armadilhas do Metabase](observabilidade/metabase-armadilhas.md) — para não ler número errado.
- Respostas padrão: [modelos de e-mail](atendimento/emails/).

### Estornos e devoluções

- [Estorno de depósito](estornos/estorno-de-deposito.md) — o modelo de dados e como provar que uma
  devolução aconteceu.
- [Estorno de saque: quanto devolver](estornos/estorno-de-saque-quanto-devolver.md) — os dois casos
  e o portão de estado.

## Escalação e handoffs

- **Fraude / MED / golpe** → é caso de **Risco & Compliance**. A régua que decide devolver ou
  contestar, e o que responder ao parceiro, vive nos processos de MED (interno). **Não** disparar
  comando de refund sem passar por essa decisão. Ver
  [contestação de MED — guia operacional](atendimento/med-contestacao-guia-operacional.md).
- **Incidente técnico de produção** → é caso de **Tech**. O que comunicar ao parceiro está em
  [comunicado de incidente](atendimento/comunicado-de-incidente.md); a triagem por sintoma, nos
  [runbooks](atendimento/runbooks.md).
- **Relação que virou operação recorrente** deixa de ser negociação e passa a ser atendimento.

## Como registrar

Incidentes e reuniões são registrados por convenção própria da operação (fora do escopo desta
base). Para o atendimento, o registro do caso vive no sistema de tickets — ver
[sistema de tickets e SLA](atendimento/sistema-tickets-sla.md).
