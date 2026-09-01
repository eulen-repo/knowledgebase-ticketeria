# Comunicado de incidente ao parceiro

Um comunicado de incidente é escrito sob pressão, por quem está no meio do problema, e é lido por
todos os parceiros ao mesmo tempo. O formato existe e está estabelecido, mas só no histórico do
grupo de avisos, o que significa que quem precisa dele às três da manhã tem de sair procurando uma
mensagem antiga para copiar. Escrever do zero nessa hora é como detalhe demais vaza.

Este documento fixa o formato, o que o corpo pode dizer e onde o comunicado é publicado. O que
fazer com o incidente em si fica nos [runbooks](./runbooks.md) e o registro posterior vira um
arquivo em `incidents/`.

## O formato

```
**🚨 [INCIDENTE EM ANDAMENTO] Serviços temporariamente indisponíveis 🚨**
Incidente: `<uuid>`

Caros parceiros,

<duas a quatro frases: o que houve, o que o time está fazendo, quando vem a próxima notícia>

- time Eulen
```

Três detalhes de forma decidem se a mensagem parece a mesma de sempre. O título vai em negrito. A
linha `Incidente:` vem imediatamente abaixo dele, e não no meio do corpo. O identificador do
incidente vai em monoespaçado, porque é o que o parceiro vai copiar e citar depois.

Os emoji ficam no título e em nenhum outro lugar. O par de 🚨 abre e fecha o rótulo enquanto o
incidente está em andamento, e dá lugar a ✅ quando ele é encerrado. O resto do comunicado é seco.

O comunicado de abertura e o de encerramento carregam o mesmo identificador. É esse par que
permite ao parceiro amarrar as duas mensagens, e trocar o identificador no encerramento desfaz o
único fio que ele tem.

## O que o corpo não diz

O limite do detalhe é mais estreito do que a vontade de explicar. Três coisas ficam de fora, e
cada uma por um motivo diferente.

O serviço não se nomeia. Escrever o nome do componente transforma um comunicado em diagnóstico
público de uma peça que o parceiro não opera e não pode conferir.

A mecânica da falha não se descreve. Frases como impedir a subida da aplicação, ou toda a operação
parada, são precisas demais para um canal de broadcast e viram citação em outro contexto.

O fornecedor não se nomeia. O teto do detalhe é falha em provedor externo. Essa é a mesma regra que
vale para atribuição de culpa em conciliação com parceiro, e aqui ela vale mais forte, porque a
mensagem não tem destinatário único.

## Onde se publica

O comunicado vai no grupo de avisos, o canal de broadcast a que só parceiros têm acesso, e nunca no
canal privado de um parceiro. A separação é a mesma de sempre: o que é individual vai no privado, o
que é anúncio vai no grupo. Um comunicado publicado num privado avisa um parceiro e deixa os outros
sem saber, e depois obriga a repetir a mensagem com carimbos de hora diferentes.

Antes de rascunhar, ler as últimas mensagens do grupo. O formato acima é o que está em uso, e o
histórico é a fonte que o mantém consistente quando ele evoluir.

## Relacionado

- [Governança das contas e dos grupos de Telegram](./telegram-contas-e-grupos.md) — o parque de
  canais em volta, e por que autodestruição de mensagem destrói evidência de comunicado.
- [Plantão](./plantao.md) — quem escreve o comunicado quando o incidente acontece fora do horário.
- [Runbooks](./runbooks.md) — a porta de entrada por sintoma, para o que o comunicado está
  contando.
- [Playbook de atendimento ao parceiro](./atendimento-parceiro-playbook.md) — o que responder
  quando um parceiro pergunta em separado sobre o incidente já comunicado.
