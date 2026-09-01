# Estorno de depósito — modelo de dados e como provar um estorno

Perguntas sobre devolução de depósito chegam de três lugares diferentes e exigem respostas
diferentes. O parceiro quer o comprovante de um estorno específico. Uma instituição quer prova
documental de que o dinheiro voltou. Um painel quer o volume devolvido do mês. As três são
respondidas por fontes distintas, e trocar uma pela outra produz resposta errada com aparência de
certeza.

Este documento reúne o modelo de dados do estorno no banco transacional e o que sustenta uma prova de
estorno quando o identificador oficial da devolução não existe. A operação do comando `/refund` está
no [Manual Operacional do Bot Pix](../atendimento/manual-operacional-bot-pix.md).

## De onde sai o volume devolvido

A fonte de verdade é `bank_tx.refunded_value_in_cents`. Somar essa coluna já cobre todos os tipos de
devolução, e é o que os cards de volume usam.

Existem duas fontes de refund, e as duas escrevem nessa mesma coluna. `automatic_by_processor` é a
devolução executada pela **processadora**. `manual_by_operation` é o caminho da nossa plataforma.

> [!CAUTION]
> `manual_by_operation` **não** quer dizer que alguém devolveu na mão. A maior parte dessas linhas tem
> latência de poucos segundos — é automação, não operador. O rótulo distingue **quem executou** (nós ou
> a processadora), e não humano de máquina. Nenhuma coluna distingue.

A tabela `pix_deposit_refund` é mais nova e parcial: serve para **rotular a fonte** do refund, não para
medir volume histórico. A `bank_tx` cobre 100% dos refunds concluídos e é muito maior. Como
consequência, a separação entre devolução da processadora e devolução nossa só é confiável a partir de
meados de 2026; num gráfico longo ela jogaria quase tudo no balde errado.

Bloqueio de consulta cadastral **não é refund**. Ele é uma tentativa de QR barrada antes do pagamento
existir, vive em `generated_pix_qr_code_attempts` com veredito `BLOCK`, e não entra em volume devolvido.

O teste limpo para separar depósito devolvido de depósito entregue não está em nenhuma dessas colunas:
é a **ausência de linha em `sending_depix`**. Em depósitos estornados, quase nenhum tem linha ali,
porque o DePix nunca chegou a ser emitido. Nos entregues, a taxa de ausência é zero.

## Por que uma fração relevante dos depósitos volta

Existe um motivo dominante de estorno: **pagador divergente**. O QR declara qual pagador é esperado; se
quem pagou foi outro CPF, a transação volta — a taxa de estorno desse motivo é de 100%, sem nenhuma
entrega. Isso é a **regra de identificação do pagador entrando em vigor** (uma das mudanças obrigatórias
[comunicadas aos parceiros](../atendimento/comunicado-parceiros-3-mudancas-obrigatorias.md)), não fraude
nem quebra: a adoção do "pagador esperado" e os estornos por divergência sobem passo a passo, semana a
semana.

> [!WARNING]
> Duas armadilhas de query enganam aqui. A divergência de pagador testada por
> `generated_pix_qr_code.payer_tax_number` dá 0%, porque a coluna que a regra usa é
> `expected_payer_euid`. E `reason_for_manual_mode` embute dois documentos dentro da string, então cada
> caso vira um grupo único e some num `GROUP BY` com `LIMIT`; agrupe por prefixo.

> [!NOTE]
> O caminho da **processadora** não grava motivo. Em janelas onde a execução do estorno migrou para a
> processadora, por alguns dias deixou de existir resposta para "por que estes depósitos estão voltando"
> — a cobertura de `reason_for_manual_mode` cai e depois volta. Ao investigar um período, confira a
> cobertura do motivo antes de concluir.

## Provar um estorno quando não existe identificador de devolução

O Banco Central identifica uma devolução por um `returnId`. Em algumas rails esse valor **não é
guardado** em lugar nenhum: a API da processadora devolve o `returnId` no corpo da resposta ao pedido
de refund, mas o daemon apenas escreve uma linha de log de nível DEBUG e não o persiste. No webhook de
estorno é pior: o identificador de devolução chega vazio.

A cobertura do identificador de devolução **varia muito por rail** — algumas preenchem em quase todos
os casos, outras são nulas por construção. Meça na hora, por rail, em vez de assumir.

### O que sustenta a prova, então

Três artefatos juntos, e nenhum deles sozinho:

1. `refunded_at` no banco transacional, que dá a data e a hora do nosso lado.
2. O lançamento de refund no **ledger da processadora**, que traz saldo antes e depois, referência ao
   depósito e uma descrição já formatada. O carimbo de criação desse lançamento é o único lugar onde a
   hora do estorno aparece do lado dela.
3. O **comprovante do depósito** emitido pela processadora, que quando o depósito foi estornado sai com
   situação "Devolvido". Ele não traz a data do estorno nem o `returnId`.

### Prova negativa: o grupo do parceiro

O bot posta no grupo do parceiro uma **confirmação com recibo** a cada estorno efetivado. A **ausência**
dessa mensagem para uma transação específica, num grupo que tem dezenas delas, é prova negativa forte —
e vale mais do que `refunded_value_in_cents` isolado, porque o comando grava a coluna **antes** de saber
se o banco aceitou.

O grupo interno de log é a outra testemunha: é o único lugar onde o histórico de comando e a mensagem
de erro que o bot devolveu ficam registrados lado a lado, com segundo. Use a busca do servidor pelo
identificador em vez de varrer por janela de data.

## Saque é outro modelo

Saque **não tem coluna de refund**. O desfecho vive em `current_state`, e vale conhecer os estados
porque mais de um deles tem data de transferência preenchida sem ter sido concluído.

| Estado | Significado |
|---|---|
| 0 | requisitado, pendente |
| 1 | enviando |
| 2 | efetivado, e o único com `transfer_date` por direito |
| 3 | falha: chave Pix errada, timeout, rejeição do banco |
| 4 | cancelado |
| 5 | substituído por retentativa (contar junto duplica volume) |
| 6 | estornado pelo operador, devolução on-chain |

Alguns saques em estado 3, 4 ou 6 têm `transfer_date` porque foram enviados e depois revertidos. Card
que faz recorte por essa data sem filtrar estado conta esses como volume efetivado.

Para saber se o DePix realmente chegou antes de um saque sair, `receiving_txid` nulo **não prova nada**:
ele é preenchido de forma desigual por cliente. O teste que decide é on-chain — cada saque tem um
endereço de recebimento próprio; decodificado e consultado no explorador, um endereço com zero
transações (confirmadas e em mempool) quer dizer que nenhum DePix chegou. Rode sempre um controle no
mesmo lote, porque zero em **todos** os endereços é sintoma de decodificador quebrado, não achado.

---

_Relacionado: [Estorno de saque](estorno-de-saque-quanto-devolver.md) ·
[Manual Operacional do Bot Pix](../atendimento/manual-operacional-bot-pix.md) ·
[Depósito em chave estática](../atendimento/deposito-em-chave-estatica.md)._
