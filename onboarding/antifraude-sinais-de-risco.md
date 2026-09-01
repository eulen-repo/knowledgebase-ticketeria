# Antifraude — Sinais de Risco em Entrevista de Onboarding

Checklist reutilizável para ler a gravação/transcrição de uma entrevista de lead **pela lente de
antifraude**, antes da decisão de viabilidade de negócio (compliance). Destilado de entrevistas reais,
anonimizado. O objetivo não é reprovar — é **nomear o que observar** e **o que condicionar** quando um
lead é aprovado.

> Este documento é genérico. Dossiês nominais de leads (com PII) **não** vivem aqui — ficam fora do git,
> em área privada, conforme a política de PII do repo.

---

## Como usar

1. Assista/leia a entrevista com esta lista ao lado.
2. Marque os sinais presentes (🟢 positivos, 🟡/🟠 de atenção). Cite o trecho verbatim de cada um.
3. Classifique o risco (§7) e escreva o parecer — **seguir / seguir com ressalvas / reprovar** — sempre com a **condição de monitoramento** correspondente, não só o veredito.
4. Lembre: sinais isolados quase nunca decidem. O que decide é o **agrupamento** (ex. A1+A2+A3 abaixo).

---

## Sinais de atenção

| Cód. | Sinal | Como aparece na fala | Sev. | Leitura antifraude |
|---|---|---|---|---|
| A1 | **Caso de uso ancorado em privacidade fiscal** | O lead descreve o mercado-alvo como pessoas que querem "privacidade do que recebem / do que declaram". | 🟠 | Beira facilitação de subdeclaração de renda. Principal ponto a pesar. Ver arquétipo §3. |
| A2 | **Sondagem repetida sobre fiscalização/reporte** | Pergunta várias vezes "tem quem fiscalize?", "é reportado?", "aparece no meu CPF?". | 🟠 | Curiosidade legítima **ou** mapeamento do que "vaza" pro fisco. Combinado com A1/A3 vira padrão. |
| A3 | **Normalização de subdeclaração** | Frases como "o cara declara só uma parte", tratando sonegação como praxe. | 🟠 | Sinaliza que parte da demanda-alvo é motivada por privacidade fiscal. |
| A4 | **Renda informal já convertida em DePix** | Relata receber serviço "por fora" e converter em cripto como caixinha. | 🟡 | Renda possivelmente não declarada. Peso menor se valores baixos e comportamento passado. |
| A5 | **Valor-alto × teto diário** | O caso de uso pede tickets acima do limite por usuário final. | 🟡 | Risco de **estruturação (smurfing)**: fracionar por dias / múltiplos usuários finais. Ver §4. |
| A6 | **Pressa por "vender já" / white label pronto** | Quer marca e vendas rápidas, sem estrutura/controles próprios. | 🟡 | Risco operacional (comunicação apressada, controles fracos), não necessariamente má-fé. |
| A7 | **Resistência a identificar contraparte** | Reclama de ter que capturar CPF/CNPJ do usuário final ("a pessoa não gosta"). | 🟡 | Atrito com a trava anti-laranja. Avaliar se é conveniência ou intenção. |

## Sinais positivos

| Cód. | Sinal | Como aparece na fala |
|---|---|---|
| G1 | **Rejeição explícita a ilícito** | "Não vou dar suporte pra nada ilícito", "não quero me meter com nada errado". |
| G2 | **Sem intenção dos no-gos** | Nada de pirâmide, bet, golpe ou serviço/produto inexistente. |
| G3 | **Alinhamento com educação/soberania** | Concorda com comunicação transparente; sem prometer anonimato. |
| G4 | **Uso legítimo comprovado** | Já usa para poupança de longo prazo (DCA), onboardou familiares de forma correta. |
| G5 | **Entende privacidade legítima** | Argumenta segurança pessoal (sequestro, vazamento de dados) — o mesmo racional que a Eulen usa. |
| G6 | **Transparência sobre imaturidade** | Não infla o projeto; admite não ter estrutura ainda. |
| G7 | **Inbound qualificado e tecnicamente literado** | Chegou sozinho, conhece Lightning/Liquid/onchain — reduz custo de evangelização e risco de desalinhamento por ignorância. |

---

## Arquétipo "aluguel-privacidade"

Padrão recorrente que merece nome próprio: **intermediar recebimento de aluguéis** com o enquadramento de
dar ao locador "privacidade do que recebe". Aparece em regiões de mercado imobiliário aquecido.

Distinguir **dois motivos que costumam vir embrulhados juntos**:

- ✅ **Legítimo:** quem está posicionado em imóveis quer **diluir/desmobilizar para BTC** (DCA para fora de
  imóvel). Uso normal do DePix.
- 🟠 **Elisão:** locador quer **esconder renda de aluguel** (renda tributável na PF). É o vetor a controlar.

O tratamento não é reprovar o lead — é **script de comunicação** (§5) + **monitoramento de estruturação**
(§4). O que taint­a o caso é a *comunicação* na ponta, não a existência do caso de uso.

---

## Estruturação (smurfing)

Quando o ticket-alvo supera o **limite diário por usuário final**, o operador pode ser tentado a **fracionar**
— dividir um pagamento por vários dias ou distribuir por múltiplos "usuários finais". Sinais:

- Múltiplos usuários finais recorrentes com **mesma origem ou destino** compatível com um único pagador real.
- Sequência de transações no **teto diário** batendo repetidamente.

Candidato a **regra no risk-sentinel**. Começar como alerta de painel (24h) e promover a tempo real só se
o sinal aparecer — não pré-otimizar.

---

## Guidance de comunicação (o que reforçar no onboarding)

Alinhado ao que a operação comercial já faz bem na entrevista. Reforçar com o parceiro aprovado:

- **Nunca** comunicar "Pix anônimo", "não declarar", "fuja do fisco" ou privacidade como sinônimo de sonegação.
- DePix **não é anônimo**: identificação por CPF/CNPJ em toda transação; reporte ao regulador. Privacidade
  existe **a partir da Liquid**, não antes.
- Comunicação enganosa **prejudica o próprio parceiro**: atrai topo de funil errado (que não permanece) e
  gera retrabalho/desconfiança pro ecossistema.
- O parceiro **emite/repassa o informe** de compra de cripto; usuário final **deve declarar**. Reforçar que
  o lead entende esse dever — reduz o risco de A1/A3.

---

## Nota de reconciliação de limites

Ao falar de limites com o lead, **não confundir dois regimes diferentes**:

- **Limite diário por usuário final** (teto de valor/dia por CPF-CNPJ pagador).
- **Travas de identificação por QR** (patamares sem identificação completa, por QR, sem contagem diária).

São coisas distintas e mudam de forma independente. **Confirmar a config atual** antes de prometer qualquer
número ao parceiro. Ver [modelo de limites de depósito] na referência de produto/pix2depix.

---

## Classificação e parecer

| Classe | Quando | Encaminhamento |
|---|---|---|
| **Baixo** | Só sinais 🟢, ou 🟡 isolados e explicados. | Seguir. |
| **Baixo–Médio** | Agrupamento de 🟠 **com** mitigantes fortes (G1–G6). | Seguir **com ressalvas** — anexar condições de monitoramento. |
| **Médio–Alto** | 🟠 sem mitigantes, ou intenção explícita de burlar trava. | Aprofundar / reprovar. Documentar motivo. |
| **Reprovar** | No-go direto (G2 violado): pirâmide, bet, golpe, laranja. | Reprovar. |

**Template mínimo de parecer:** classe de risco · sinais citados (verbatim) · mitigantes · **condições se
aprovado** (comunicação, regra de estruturação, confirmação de dever fiscal) · itens a reconciliar (limites).
