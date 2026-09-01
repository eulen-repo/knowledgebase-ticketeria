# Sistema de Tickets & SLA

Quando um parceiro reclama num grupo do Telegram, quando alguém acha um bug, ou quando cai uma demanda
financeira no board, tudo vira a mesma coisa: um **ticket** no CRM. Este documento explica como isso
funciona de ponta a ponta — de onde vêm os tickets, como são medidos, e o que confiar (ou não) nos
números. *(Versão sanitizada — hosts internos, IDs de dashboard, roster da equipe e métricas-foto foram
removidos.)*

Se os termos (ATD, ENG, DEM, MED) não forem óbvios, comece pelo [glossário](../glossario.md).

---

## Visão geral

```
   FONTES                          BACKBONE                         VISUALIZAÇÃO
 ──────────                    ─────────────────                ────────────────────
 Telegram (parceiros) ──┐
   watcher + backfill    │
                         ├──▶   CRM (objeto "Ticket")  ──────▶   Metabase (dashboard de SLA)
 Board Engenharia ───────┤      ATD-* · ENG-* · DEM-*           medianas / janelas / filas /
   (manual, Kanban)      │            │                          placar de cumprimento — ao vivo
 Board "DEMANDAS" ───────┘            └──▶ views/boards no próprio CRM
   sync one-way
```

Duas escolhas sustentam o desenho:

1. Atendimento de parceiro, tarefa de engenharia e demanda financeira são **todos o mesmo objeto**: muda
   o prefixo do protocolo e alguns campos, mas a **medição de SLA é a mesma** para todos.
2. O Metabase **nunca escreve** — ele lê o banco do CRM direto. O dashboard é sempre reflexo do ticket, e
   corrigir um número significa **corrigir o ticket, não o card**.

---

## 1. O backbone — objeto `Ticket`

### Famílias de protocolo

| Prefixo | Origem | Como entra | Sequência |
|---|---|---|---|
| **`ATD-AAAA-NNNNN`** | Atendimento a parceiro (Telegram) | watcher + backfill automáticos | própria |
| **`ENG-AAAA-NNN`** | Tarefa de engenharia | criado à mão / script | própria |
| **`DEM-AAAA-NNN`** | Demanda do board de demandas | sync one-way | própria |

Cada família tem **sequência própria** (não colidem). O grosso do volume é ATD (atendimento).

### Campos principais

| Campo | Tipo | Para que serve |
|---|---|---|
| `protocolo` | texto (**UNIQUE**) | identificador humano (ATD-/ENG-/DEM-) |
| `chaveThread` | texto (**UNIQUE**) | chave de dedup — re-rodar o pipeline não duplica |
| `name` | texto | título do ticket |
| `status` | enum | ciclo de vida (ver enums abaixo) |
| `severidade` | enum | `ROTINA · ALTA · CRITICO` — dirige a meta de SLA |
| `tipo` | enum | natureza do atendimento (ver abaixo) |
| `canal` | enum | `TELEGRAM · WHATSAPP · EMAIL` (hoje ~100% Telegram) |
| `prioridade` | enum | raia do board de engenharia (`P0…P3` + `FINALIZADO`) |
| `etapa` | enum | as listas do board de demandas |
| `dependencia` | enum | `INTERNA · EXTERNA` (depende de parceiro/terceiro?) |
| `medRelacionado` | boolean | o atendimento está ligado a uma MED? |
| `valor` | dinheiro | valor em disputa/atendimento |
| `abertoEm` | data | **idade real** do atendimento (≠ `createdAt`) |
| `slaDue` | data | prazo do SLA |
| `resolvidoEm` | data | quando foi resolvido (base do SLA de tempo) |
| `primeiraRespostaEm` | data | 1ª resposta (FRT) |
| `responsavelId` | relação | dono do ticket |
| `partnerId` | relação | parceiro/lojista associado |
| `identificadores` | texto | txid / end-to-end Pix extraídos (rastreabilidade) |
| `contexto` | texto longo | transcript / descrição preservada |
| `acaoTomada` | texto | desfecho — **é o que marca um ATD "resolvido de verdade"** no SLA |

> [!WARNING]
> **Idade = `abertoEm`, nunca `createdAt`.** O `createdAt` é a data do rebuild histórico (artificial). O
> `updatedAt` sofreu bulk-reset e também não serve como proxy de resolução.

> [!CAUTION]
> **PII no `contexto`.** O transcript preservado pode conter **PII de cliente final** num blob sem
> estrutura, sem timestamp por linha e sem prazo de retenção — não dá para expurgar seletivamente. Trate
> o campo como sensível.

### Valores dos enums (referência)

| Enum | Valores |
|---|---|
| `status` | `ABERTO` · `EM_ANALISE` · `AGUARDANDO_PARCEIRO` · `AGUARDANDO_EXTERNO` · `RESOLVIDO` · `FECHADO` |
| `severidade` | `ROTINA` · `CRITICO` · `ALTA` |
| `tipo` | `STATUS` · `DEVOLUCAO_DEPOSITO` · `ERRO_SAQUE` · `DIVERGENCIA_VALOR` · `LIMITE_DIARIO` · `AUMENTO_LIMITE` · `BLOQUEIO_CPF` · `UNDER_REVIEW` · `DEPIX_NAO_CHEGOU` · `QR_ESTATICO` · `OTC` · `OUTRO` |
| `canal` | `TELEGRAM` · `WHATSAPP` · `EMAIL` |
| `prioridade` | `P0` · `P1` · `P2` · `P3` · `FINALIZADO` |
| `etapa` | `PENDENCIA` · `EM_ANDAMENTO` · `A_REEMBOLSAR` · `AGUARDANDO_CLIENTE` · `AGUARDANDO_EXTERNO` · `CONCLUIDO` |
| `dependencia` | `INTERNA` · `EXTERNA` |

---

## 2. As três fontes de tickets

### 2.1 Telegram → `ATD-*` (automático)

O grosso do volume. Vem por **dois caminhos** complementares:

- **(a) Backfill — broad scan.** Contas *userbot* leem os grupos de parceiro e varrem o histórico; o
  material é consolidado, segmentado em "episódios", classificado (fronteiras / tipo / severidade /
  status) e vira `ATD-*` com dedup por `chaveThread` e `resolvidoEm` **real** (última msg do episódio).
  É backfill, não tempo-real.
- **(b) Go-forward — watcher.** Um bot **passivo e separado** que **lê todos os grupos e NUNCA responde**,
  gravando cada mensagem. Só cria episódio em grupos autorizados (company ATIVO/ONBOARDING no CRM);
  limpeza mecânica antes de qualquer chamada de IA; classificação em batch com teto por passada.
  Resolução = 1ª resposta de OPERADOR no grupo → `resolvidoEm` real.

### 2.2 Engenharia → `ENG-*` (manual)

Problemas de engenharia que não vêm de parceiro. Viram tickets num board Kanban agrupado por `prioridade`
(raias `P0 → P3` + `FINALIZADO`). **A raia é a verdade do "concluído":** um ENG está *aberto* enquanto
`prioridade ≠ FINALIZADO` e *pronto* quando entra na raia `FINALIZADO`.

### 2.3 Board "DEMANDAS" → `DEM-*` (sync one-way)

O board de demandas financeiras (deságios, depósitos a verificar, reembolsos) espelha no CRM, virando
tickets `DEM-*` com board Kanban próprio. Mapeamento: lista → `etapa`; card → `name`; descrição →
`contexto`; membro → `responsavelId`; id do card → `chaveThread` (dedup, idempotente).

---

## 3. As views/boards no CRM

| View | Tipo | Filtro | Para quê |
|---|---|---|---|
| **All Tickets** | TABLE | — | tabela mestra |
| **Engenharia · Prioridades** | KANBAN | `protocolo CONTAINS ENG-`, agrupa por `prioridade` | board de engenharia — arrastar para `FINALIZADO` fecha |
| **Demandas** | KANBAN | `etapa IS_NOT_EMPTY`, agrupa por `etapa` | isolar os `DEM-*` |
| **Fila por SLA** | TABLE | status aberto, ordena `slaDue ASC` | o que atacar primeiro |
| **SLA estourado** | TABLE | `slaDue IS_IN_PAST` + status aberto | fila de estouros |
| **Pendências** | TABLE | status aberto, ordena `severidade DESC, slaDue ASC` | backlog geral |

> [!NOTE]
> **Status "em aberto"** para as filas = `('ABERTO','EM_ANALISE','AGUARDANDO_EXTERNO','AGUARDANDO_PARCEIRO')`.
> Esquecer `AGUARDANDO_EXTERNO` **subconta** o backlog externo — CRM e Metabase precisam concordar nisso.

---

## 4. Roteamento & alertas de engenharia

Quando um ticket tem dono, o roteamento avisa a pessoa certa — **modular por assignee** ("o alerta depende
de quem está atribuído"):

- **E-mail** = e-mail do membro no CRM (automático).
- **Chat interno** = webhook do espaço de engenharia.
- **SLA estourado** (ticket aberto com `slaDue < agora`, exceto `AGUARDANDO_EXTERNO`) → alerta no Telegram.

O **papel** de cada pessoa importa: é ele que diz se um alerta faz sentido para ela.

---

## 5. SLA — metas e como é medido

### As metas

O relógio depende da **severidade** e da **família**. **Horário comercial** = seg–sáb, janelas
**09:00–12:00 e 13:30–20:00** (America/São_Paulo, sem feriados). `CRITICO` conta **relógio corrido**
(24/7); `ALTA` e `ROTINA` contam **só horário comercial**.

| Severidade | 🎫 Atendimento (ATD) | 🛠 Engenharia (ENG) |
|---|---|---|
| 🔴 **CRITICO** | ≤ **2h** (relógio corrido) | ≤ **1 dia útil** (9h comerciais) |
| 🟡 **ALTA** | ≤ **24h úteis** | ≤ **3 dias úteis** (27h comerciais) |
| 🟢 **ROTINA** | ≤ **48h úteis** | ≤ **5 dias úteis** (45h comerciais) |

> 1 dia útil = 9h. Logo 1d=9h, 3d=27h, 5d=45h de expediente.

> [!WARNING]
> **A janela de horário comercial vive em vários lugares (SQL dos cards + espelho Python) e já divergiu.**
> Mexeu numa, mexe em todas — divergir volta a dar SLA medido diferente do prometido.

### Como a resolução é medida

- **Atendimento (ATD).** Um ticket só conta como "resolvido" quando tem **`resolvidoEm` real** **+**
  `acaoTomada` contendo **"marcador de chat"** **+** `protocolo LIKE 'ATD-%'`. Tempo = `resolvidoEm −
  abertoEm` (útil, exceto CRÍTICO = relógio corrido).
- **Engenharia (ENG).** Resolvido = **entrou na raia `prioridade = FINALIZADO`** (o instante vem do
  histórico de auditoria). Aberto = `prioridade ≠ FINALIZADO`.
- **Demandas (DEM).** Acompanhadas pela `etapa`.

### Seção Sistema (banco transacional)

Duas métricas **não são tarefa humana** — são estado do sistema, lidas direto do `pix2depix`:
- **Saque em erro**: `transaction_status` com `domain='WITHDRAW'` e `status='ERROR'` — tempo até sair do erro.
- **Depósito em análise manual**: `domain='BANK'` e `status='MANUAL_REQUIRED'` — tempo até liberar.

> [!CAUTION]
> **Qualidade do dado de resolução.** `resolvidoEm` e `primeiraRespostaEm` só passaram a ser gravados de
> forma consistente a partir de meados de 2026 e cobrem uma amostra pequena. Por isso os cards de
> **cumprimento/tendência** rodam sobre amostra recente e enviesada; os cards de **backlog/worklist** (que
> só olham `status`/`slaDue`/raia) são confiáveis. A **primeira resposta** só é tempo de resposta real
> quando o episódio **abre com o parceiro** (`abertoEm` = a primeira mensagem do recorte).

---

## 6. Dashboard de SLA (Metabase)

O dashboard de SLA é a foto operacional: quanto demoramos para resolver, o que está aberto, o que estourou.
Blocos típicos: medianas de resolução (ATD), fila e velocidade (ENG), demandas em aberto (DEM), saúde da
fila (backlog sem dono, SLA estourado, cumprimento 7d), distribuição por severidade, placar por severidade
+ tendência semanal + worklist de estouros, e a seção de sistema (saque em erro, depósito em análise).

> [!TIP]
> Antes de citar qualquer "% de cumprimento de SLA", lembre da ressalva de qualidade do dado acima: o
> placar mede uma amostra pequena. Os cards de worklist e backlog são os confiáveis para o dia a dia.

---

_Relacionado: [Buscador universal](../observabilidade/metabase-buscador-universal.md) ·
[Playbook de atendimento](atendimento-parceiro-playbook.md) · [Processos de atendimento](../processos.md)._
