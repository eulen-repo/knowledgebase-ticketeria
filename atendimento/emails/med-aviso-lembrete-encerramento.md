# E-mails de MED ao parceiro (aviso · lembrete · encerramento)

Três e-mails compõem o ciclo de um lote de MED, todos endereçados **apenas ao próprio parceiro**:

- **Aviso** — sai no dia seguinte ao relato e cobra a resposta.
- **Lembrete** — sai na véspera do prazo, quando falta um dia e o caso ainda não consta devolvido.
- **Encerramento** — sai no dia do prazo, tendo havido devolução ou não.

Os três de um mesmo lote carregam o **mesmo número de protocolo** e o **mesmo assunto**, o que os junta
numa busca só dos dois lados. Os três afirmam só o que o registro de devolução prova, e por isso **não**
dizem quem devolveu nem que o parceiro deixou de responder.

**Campos a substituir:** `{PROTOCOLO}`, `{PARCEIRO}`, `{DATA}`, `{PRAZO}`, `{ENDEREÇO}` (endereço de
devolução em DePix), `{ID_E2E}`, `{QR_ID}`, `{EUID}`, `{EMID}`, `{VALOR}`, `{DATA_DEPÓSITO}` e as linhas
da relação de casos.

> [!NOTE]
> `{ID_E2E}` é o identificador do depósito **sem** o prefixo da rail (ex.: `<rail>_E607…` sai como
> `E607…`, que é o que o parceiro reconhece). `{QR_ID}` sai como traço quando o depósito não nasceu de
> um QR nosso. `{EUID}` sai vazio quando o QR não tem pagador esperado gravado.

---

## Aviso

**Assunto:** Relatos de infração — {PARCEIRO} — Protocolo: {PROTOCOLO}

Olá,

Seguem as solicitações de MED (Mecanismo Especial de Devolução) recebidas em {DATA}, referentes a
depósitos de usuários de vocês.

O MED é o procedimento exclusivo do Pix para pedir a devolução de um pagamento quando há fundada
suspeita de fraude, como golpe, coerção ou acesso não autorizado à conta, ou quando houve falha
operacional.

| # | Data do depósito | Valor | ID E2E e QR ID | EUID | EMID |
|---|---|---|---|---|---|
| {N} | {DATA_DEPÓSITO} | R$ {VALOR} | E2E: {ID_E2E} / QR: {QR_ID} | {EUID} | {EMID} |

O prazo para a resposta de vocês é {PRAZO}. Cada caso admite uma de duas respostas.

**Fazer a devolução.** Se vocês estiverem em posse do valor, sugerimos devolver o valor do depósito em
DePix no endereço abaixo e responder a este e-mail com a hash da devolução e o ID E2E correspondente.

Endereço de devolução:

{ENDEREÇO}

**Contestar a solicitação.** Respondam a este e-mail com o que sustenta cada operação.

1. Houve indício de fraude na operação?
2. O que o usuário recebeu, e quando? Descrevam o serviço prestado ou o produto entregue, e em que
   plataforma, com o endereço dela (site ou aplicativo).
3. Qual é o identificador dessa entrega no sistema de vocês (ID da ordem, número de protocolo)?
4. Que evidência sustenta a entrega? (Comprovante de pagamento, extrato, contrato de prestação de
   serviço, registro de saque com os dados da transação…).
5. Quem é o usuário? Identificação cadastral e o histórico dele com vocês, incluindo se as transações
   dele são recorrentes.
6. Ainda há saldo em posse de vocês? Havendo qualquer saldo remanescente, recomendamos o bloqueio
   imediato.

O máximo de informação auxilia na defesa.

**Pedir o cancelamento do MED.** Sempre que possível, orientem o cliente final a pedir o cancelamento à
instituição em que fez o Pix. O cancelamento retira a marcação de fraude e impede a abertura de um novo
MED sobre a mesma transação.

A ausência de resposta pode acarretar prejuízo do caso e pesa na avaliação de vocês junto à Eulen.

Recomendamos marcar este e-mail como importante, para que os próximos cheguem à caixa de entrada de
vocês.

Protocolo: {PROTOCOLO}

Compliance
@Eulen
eulen.app depix.info

---

## Lembrete

**Assunto:** Relatos de infração — {PARCEIRO} — Protocolo: {PROTOCOLO}

Olá,

Falta um dia para o prazo das solicitações de MED (Mecanismo Especial de Devolução) recebidas em
{DATA}, que termina em {PRAZO}. Se vocês já responderam, desconsiderem este e-mail.

Daí em diante o corpo é o do aviso, palavra por palavra, **sem repetir a relação de casos**. O que liga
o lembrete aos casos é o assunto, idêntico nos três e-mails do mesmo protocolo.

> O lembrete **não enxerga** se o parceiro já respondeu (a resposta chega por e-mail e não é gravada em
> campo que a consulta leia) — por isso o texto abre pedindo que quem já respondeu desconsidere.

---

## Encerramento

**Assunto:** Relatos de infração — {PARCEIRO} — Protocolo: {PROTOCOLO}

Sai no dia do prazo para todo lote relatado, tendo havido devolução ou não. **Não** traz a relação de
casos (ela já saiu no aviso, sob o mesmo assunto).

Olá,

O prazo de resposta das solicitações de MED (Mecanismo Especial de Devolução) recebidas em {DATA}
terminou em {PRAZO}.

Estamos dando continuidade por aqui. Havendo qualquer devolutiva sobre estes casos, retornamos com os
detalhes.

Agradecemos pela colaboração de vocês durante a análise.

Para os casos sem devolução registrada, este e-mail comunica o fim do prazo. Se vocês responderam por
e-mail, a resposta continua valendo.

A ausência de resposta pode acarretar prejuízo do caso e pesa na avaliação de vocês junto à Eulen.

Qualquer dúvida, estamos à disposição.

Protocolo: {PROTOCOLO}

Compliance
@Eulen
eulen.app depix.info

**Variações condicionais:**

- **Abertura**, no lote em que todos os casos constam devolvidos, troca por:
  > Registramos a devolução das solicitações de MED (Mecanismo Especial de Devolução) recebidas em
  > {DATA}. Com isso, o atendimento delas está concluído deste lado.
- **Os dois parágrafos de fecho** são independentes: o agradecimento sai havendo qualquer caso
  devolvido; o aviso de fim de prazo, havendo qualquer caso sem devolução. No lote misto saem os dois,
  nessa ordem.
