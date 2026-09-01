# Documentação de Saques Automatizados

> **Status:** API e Telegram ativos
> **Última atualização:** 30/07/2026

> [!NOTE]
> Esta página é o **catálogo de erros conhecidos** do saque. Os comandos de operação do saque
> (aprovar, retentar, cancelar, devolver, listar travados) vivem no
> [Manual Operacional do Bot Pix §6.4](manual-operacional-bot-pix.md).

## Descrição

Este documento registra o funcionamento e os possíveis erros conhecidos no sistema de **saques
automatizados** da plataforma.

Um saque entra por **dois caminhos**, não um: o endpoint de saque da API, e o comando `/withdraw`
do bot de parceiro — mais `/ww`, o saque interno do grupo de operação. A partir daí o tratamento é
o mesmo pipeline. Logs de erros são distribuídos entre **Telegram**, **Metabase** e tabelas
específicas do banco de dados.

Um saque que estoura o **teto diário por CPF/CNPJ** não é recusado: vai para **modo manual** com
`reason_code = PAST_DAILY_LIMIT`, e tanto o grupo de log quanto o parceiro são avisados. O teto é a
mesma tabela `levels` usada no depósito.

---

## Estrutura geral do processo

1. A API recebe uma requisição de saque.
2. O sistema executa as validações e tenta processar a operação.
3. Em caso de falha, o erro é logado no canal apropriado (Telegram, Metabase ou banco).
4. É necessário **tratamento manual** para solucionar os erros.

---

## Casos de Erro Conhecidos

| Código / Mensagem de Erro | Origem do Log | Causa Provável | Ação Recomendada |
|-----------------------------|----------------|----------------|------------------|
| `does not match expected amount` | Telegram | Divergência entre o valor esperado e o recebido; geralmente diferença de 1 centavo. | Verificar log do Telegram e aprovar manualmente o saque se a diferença for mínima. |
| `error generating pix out: failed to make request: Post "https://api.<processadora>[...] (Client.Timeout exceeded while awaiting headers)` | Metabase | Timeout ao tentar criar a saída PIX (problema de rede ou lentidão na API externa). | Verificar conectividade e repetir a operação. |
| `new deposit on closed address` | Tabela `invalid_deposits` | Depósito recebido em endereço que já foi encerrado. | Realizar batimento com dados de transações para decidir sobre estorno. |
| `unsupported asset` | Tabela `invalid_deposits` | Ativo não suportado no fluxo atual. | Contatar parceiro para entender o ocorrido e ajustar suporte no sistema. |

---

## Onde ficam os logs

- **Telegram:** Logs operacionais e de validação.
- **Metabase:** Monitoramento de erros de integração externa (ex.: Pix API).
- **Banco de Dados:** Tabelas de controle (`invalid_deposits`, `processed_withdrawals`, etc.).

---

> [!NOTE]
> O **quadro de processadoras muda** ao longo do tempo (entram e saem). O **formato** do erro de
> timeout vale para qualquer uma; só o host da URL muda. Por isso a mensagem acima está
> generalizada como `api.<processadora>`.

## Próximos Passos

- Adicionar novos casos de erro conforme identificados.
- Registrar políticas de reprocessamento automático e alertas.
