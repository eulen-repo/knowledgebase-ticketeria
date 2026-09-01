# Contestação de MED 2.0 — Guia de Operações

Guia para a equipe de Operações/Compliance. Explica o que é o MED 2.0, como ele afeta os parceiros
e o que fazer do início ao fim quando uma contestação chega. *(Versão sanitizada — dados da entidade
jurídica, do signatário e dos parceiros/bancos específicos foram removidos.)*

---

## 1. Resumo executivo

### O que é um MED 2.0?

O **MED (Mecanismo Especial de Devolução)** é um mecanismo do Banco Central do Brasil. Permite que o
banco do pagador solicite a devolução de valores de uma transação Pix em casos de fraude ou golpe
alegado pelo cliente do pagador.

No **MED 2.0**, o banco do **recebedor** (no nosso caso, o banco da conta que recebeu o Pix via a
**processadora contratada**) notifica a processadora e pergunta: "Você solicitou essa devolução?" A
resposta é sempre **não** — nós nunca iniciamos um MED. Responde-se com a **Declaração de Não
Solicitação de MED**, assinada pela sócia responsável.

Enquanto o processo não é resolvido, o banco do recebedor **bloqueia o saldo** da conta do recebedor.
O bloqueio **não tem duração fixa**: o Manual Operacional do DICT §20.1.7 lista os três únicos
momentos em que o desbloqueio pode ocorrer — depois de o banco do recebedor **rejeitar** a notificação
de infração, depois de **concluir** a solicitação de devolução, ou caso a Recuperação de Valores seja
**cancelada ou encerrada**.

> [!IMPORTANT]
> **Nenhum documento oficial do Pix fixa duração de bloqueio.** O que existe é prazo de etapa: 7 dias
> para os PSPs recebedores analisarem e fecharem as notificações, mais 72 horas para o PSP recuperador
> iniciar a devolução, e o DICT encerra sozinho a Recuperação de Valores se ela não for iniciada nesse
> prazo. **Não** repassar ao parceiro nenhum "prazo regulatório" de bloqueio — ele não existe.

### O que fazemos nesse cenário

✅ Gera-se um documento (Declaração de Não Solicitação de MED) com os dados exatos da transação.
✅ A sócia responsável assina eletronicamente.
✅ Entrega-se o PDF assinado ao parceiro para que ele repasse ao banco ou ao cliente final.
✅ Registra-se o caso completo (prints, conversas e PDFs) para auditoria.

### O que **não** fazemos

❌ Não solicitamos o MED — jamais.
❌ Não devolvemos o valor automaticamente.
❌ Não garantimos que o banco do recebedor desbloqueie o saldo — isso depende do banco.
❌ Não armazenamos credenciais da ferramenta de assinatura em nenhum sistema ou arquivo.

---

## 2. O que muda para operações

### Quem reporta e como

Os parceiros reportam casos de MED pelos canais habituais (Telegram / e-mail). O parceiro normalmente
encaminha:
- E-mail do banco com o E2E da transação e o prazo informado;
- ou JSON da API com os dados do saque;
- ou screenshot da notificação de bloqueio no app do recebedor.

### O que você entrega ao parceiro

Um **PDF assinado** da Declaração de Não Solicitação de MED contendo: a razão social/CNPJ/endereço da
entidade, o E2E da transação contestada, o valor e a data do Pix, e a assinatura eletrônica da sócia
responsável.

### O que dizer ao parceiro

> "Geramos a Declaração de Não Solicitação de MED para essa transação. Segue o PDF assinado. Você deve
> encaminhá-lo ao banco pelo e-mail informado na notificação. O banco tem até [prazo informado] para
> analisar."

Se o parceiro perguntar sobre o bloqueio do saldo:

> "O bloqueio é de responsabilidade do banco do recebedor enquanto a análise do MED está em curso. Com
> a declaração assinada, o banco tem a documentação necessária para desbloquear. O desbloqueio acontece
> quando o banco fecha a análise, conclui a devolução, ou o caso é cancelado."

---

## 3. Fluxo completo — passo a passo

```
1. Parceiro reporta o MED (Telegram / e-mail do banco)
         ↓
2. Coletar os dados obrigatórios: E2E · valor · data · e-mail do banco
         ↓
3. Confirmar no sistema que a transação existe e foi processada
         ↓
4. Gerar a Declaração de Não Solicitação de MED (.docx) a partir do template
         ↓
5. Obter o acesso à ferramenta de assinatura com o responsável interno
         ↓
6. Fazer upload do documento · configurar assinatura por link
         ↓
7. A sócia responsável assina
         ↓
8. Baixar o PDF assinado e enviar ao parceiro
         ↓
9. Registrar o caso com todos os dados, prints e PDF
```

### Dados obrigatórios para gerar o documento

| Campo | Obrigatório | Onde encontrar |
|-------|-------------|----------------|
| E2E (EndToEnd ID) | ✅ | E-mail do banco · JSON da API · comprovante de saque |
| Valor (R$) | ✅ | E-mail do banco · JSON: `payoutAmountInCents ÷ 100` |
| Data da transação | ✅ | E-mail do banco · JSON: `transferDate` (converter UTC → BRT) |
| E-mail do banco parceiro | ✅ | E-mail de notificação recebido pelo parceiro |
| Nome do recebedor | ℹ️ | Confirmação de que é a transação correta |

> [!NOTE]
> Os **dados fixos da entidade** (razão social, CNPJ, sede) e o **CPF do signatário** são preenchidos
> por quem já tem o acesso à ferramenta de assinatura; não vivem nesta base por serem dado sensível.

---

## 4. Prazos e urgência

> [!IMPORTANT]
> Os prazos que um banco informa (ex.: "48 horas úteis", "N dias de bloqueio") são **o que aquele
> banco disse**, não a norma. O Banco Central não fixa duração de bloqueio: o desbloqueio é por evento
> (DICT §20.1.7). **Não** repassar nenhum desses números ao parceiro como "prazo regulatório".

O prazo começa a contar a partir da **data da notificação ao parceiro**, não da data em que chegou até
nós. Priorize os casos com prazo próximo.

---

## 5. Ferramenta de assinatura — orientações

- A conta pertence à empresa. Solicitar acesso ao responsável interno.
- Configurar sempre como **"Assinar por link"** — a signatária não precisa ter conta própria.
- Nunca armazenar login, senha ou tokens em arquivos, memória ou sistemas.

---

## 6. Registro do caso

Cada caso deve ser registrado com: E2E, saque ID, valor, data, banco, parceiro e canal; timeline
completa; a conversa/e-mail do parceiro; o screenshot da notificação de bloqueio (se houver); o e-mail
do banco (se houver); o PDF assinado; e o status final.

---

## 7. FAQ e troubleshooting

**O parceiro disse que o banco não aceitou a declaração. E agora?**
Verifique se o banco exige documentação adicional (contrato, comprovante de prestação de serviço).
Alguns bancos exigem contrato com assinatura eletrônica validada além da declaração. Escale para o
compliance.

**O saldo do recebedor não foi desbloqueado mesmo com a declaração entregue.**
O desbloqueio é decisão do banco do recebedor. A declaração é a nossa parte. Oriente o parceiro a
acionar diretamente o banco informando que a documentação foi entregue e cobrar prazo de resposta.

**Chegou um MED de uma transação que não reconheço no sistema.**
Confirme o E2E no sistema antes de qualquer ação. Se a transação não existir, não gere o documento —
sinalize ao compliance.

**O prazo do banco já venceu antes de conseguirmos gerar o documento.**
Gere e entregue assim mesmo — o banco pode ou não aceitar fora do prazo. Registre a data de entrega e
o motivo do atraso.

**Posso gerar a declaração com dados incompletos?**
Não. E2E, valor e data são obrigatórios. Se o parceiro não tiver o E2E, ele pode buscar no app do banco
ou no extrato.

**Isso é o mesmo que um estorno (`/refund`)?**
Não. O MED é iniciado pelo banco do pagador e comunicado ao banco do recebedor. Nós não devolvemos
nenhum valor — apenas declaramos que não solicitamos o MED. O estorno (`/refund`) é uma operação
diferente, iniciada pela equipe de operações. Ver [Estorno de depósito](../estornos/estorno-de-deposito.md).
