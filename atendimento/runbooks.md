# Runbooks — porta de entrada por sintoma

Às quatro da manhã quem responde tem um **sintoma**, não o nome do componente. Esta página é a
porta de entrada por sintoma. Cada linha aponta para onde o procedimento está documentado.

> [!NOTE]
> Muitos runbooks de infraestrutura (carteira, filas, cluster, observabilidade) vivem no
> repositório interno de infra e **não** foram trazidos para esta base. As linhas abaixo marcadas
> como *(interno)* apontam para lá; as demais têm o doc nesta base.

## Dinheiro

| O sintoma | Onde está |
|---|---|
| Provar que um estorno aconteceu, ou que nenhum DePix foi emitido | [Estorno de depósito](../estornos/estorno-de-deposito.md) |
| Quanto devolver num estorno de saque, e quando não devolver | [Estorno de saque](../estornos/estorno-de-saque-quanto-devolver.md) |
| Contestar um MED | [Contestação de MED — guia operacional](med-contestacao-guia-operacional.md) |
| Pagamento em chave estática sem dono / creditado errado | [Depósito em chave estática](deposito-em-chave-estatica.md) |
| Erros conhecidos de saque | [Documentação de saques](withdraw-docs.md) |

## Parceiro

| O sintoma | Onde está |
|---|---|
| Comando do bot, triagem, status de saque, auditoria manual | [Manual operacional do bot Pix](manual-operacional-bot-pix.md) |
| O parceiro perguntou e a resposta já existe pronta | [FAQ de atendimento](faq-atendimento.md) · [Playbook de atendimento](atendimento-parceiro-playbook.md) |
| Comunicar um incidente ao parceiro | [Comunicado de incidente](comunicado-de-incidente.md) |
| Ler um identificador / achar onde ele aparece | [Buscador universal](../observabilidade/metabase-buscador-universal.md) |
| Não ler um número errado na consulta | [Armadilhas de consulta](armadilhas-de-consulta-pix2depix.md) · [Armadilhas do Metabase](../observabilidade/metabase-armadilhas.md) |

## Infraestrutura e plataforma *(interno)*

| O sintoma | Onde está |
|---|---|
| Carteira travada, rotação de carteira, nó Liquid degradado | *runbook interno de infra* |
| Mensagem não chegou ao consumidor; webhook entregou e nada apareceu | *runbook interno de infra* |
| Onde o daemon roda e onde está o log | *runbook interno de infra* |
| Acesso à AWS e diagnóstico de cluster | *runbook interno de infra* |

## Relacionado

- [Processos de atendimento](../processos.md) — por onde cada tipo de trabalho entra.
