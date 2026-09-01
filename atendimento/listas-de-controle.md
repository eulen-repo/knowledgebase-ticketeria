# As listas de controle que atendem por "whitelist"

"O parceiro está na whitelist" não decide nada sozinho. Vários controles diferentes carregam esse
apelido no caminho de um depósito, e eles apontam para lados opostos. Um dispensa a triagem
antifraude, outro barra o pagador, outro apenas empurra a transação para o modo manual, e outro
libera um teto de valor. Trocar um pelo outro numa conversa de operação faz alguém prometer ao
parceiro um efeito que a lista pedida não produz.

Este documento diz qual lista responde a qual pergunta, onde ela mora e para que lado ela falha
quando algo dá errado.

## Qual lista responde a qual pergunta

| Lista | Onde mora | A pergunta que ela responde |
|---|---|---|
| Whitelist antifraude | tabela `antifraud_whitelist_events` | este CPF ou CNPJ dispensa a triagem na criação do QR? |
| `blocked_tax_numbers` | tabela homônima | este pagador está barrado? |
| `blocked_bank_identifiers` e IPs bloqueados | tabela e `config.yaml` | a instituição de origem exige revisão manual? |
| Permissão `qrcodewhitelist` | coluna `permissions` do parceiro, em JSON | este parceiro pode gerar QR acima do teto de usuário novo? |
| Autenticação do bot | `config.yaml`, chave `authentication` | esta pessoa pode rodar este comando de operador? |
| Allowlist de IP do bank-watcher | configuração do webhook | esta origem pode entregar webhook nesta fila? |

## Para que lado cada uma falha

Esta é a parte que decide se a ausência de uma lista abre ou fecha a porta.

- **A whitelist antifraude libera.** Um assunto presente nela recebe um ALLOW determinístico na
  criação do QR dinâmico. É um log de eventos append-only, e a associação vigente é o último evento
  por número de documento: `ADD` = liberado; `REMOVE` ou evento nenhum = a triagem normal roda.
- **As duas listas de bloqueio falham fechado, de propósito.** Erro de banco ao consultar é devolvido
  em vez de engolido; o webhook aborta e é reentregue, porque tratar o erro como "não bloqueado"
  deixaria passar uma instituição bloqueada. Recusada a retomada, a transação vai para o **modo
  manual** em vez de ser descartada.
- **A allowlist de IP do bank-watcher falha aberto.** Quando um webhook não declara endereço nenhum,
  o filtro fica desligado e toda origem é aceita. Uma origem fora da lista também não recebe erro: a
  mensagem vai para a fila `.blocked` e o chamador recebe 200.

## Dois nomes que enganam

- A lista de "IPs bloqueados" do `config.yaml` na verdade guarda **identificadores de instituição
  financeira** (ISPB e número COMPE do banco), apesar do que o nome sugere, e um casamento coloca a
  transação em **modo manual** em vez de recusá-la. A autoridade é a tabela do banco; a lista do
  arquivo sobrevive como transição.
- A permissão `qrcodewhitelist` é um **atributo do parceiro**: autoriza o parceiro a mandar
  `whitelist=true` na criação do QR, dispensando o teto de usuário novo. Atenção à **armadilha de
  unidade**: o limite do parceiro é lido em centavos e o padrão da configuração é multiplicado por
  100 — os dois campos não aceitam o mesmo número.

Dois efeitos colaterais dessa permissão costumam surpreender: um QR **com atraso** liga o mesmo flag
por dentro (o atraso já contém usuário novo); e o teto por transação do tier do merchant **não sobe
junto**, porque ele exige identificação real do pagador.

## Quem opera o bot

O bot de operação tem duas condições encadeadas. Comando marcado como **protegido** exige desafio
GPG (estar em `config.yaml`, chave `authentication`, com chave e grupo declarados). Além disso, o
grupo da pessoa precisa constar da lista do comando, que por padrão é apenas `admin`; o portão nega
por omissão para que um grupo novo nunca ganhe comando de graça. Comando **não protegido** não passa
por isso — por isso a marcação de proteção de um comando importa.

## Quem muda o quê

A whitelist antifraude é mantida por comando de operador protegido por GPG; como o log é
append-only, remover alguém grava um evento `REMOVE` em vez de apagar linha. A permissão do parceiro
é gravada na coluna `permissions` em JSON. A allowlist do bank-watcher e os grupos do bot vivem em
configuração, então mudam por deploy.

## Relacionado

- [FAQ de atendimento](faq-atendimento.md) — os tetos por QR na forma em que o operador precisa
  deles.
- [Runbooks](runbooks.md) — o índice por sintoma.
