# Adicionar um Parceiro

Como ativar um novo parceiro na plataforma usando o comando `/addpartner`, e como desabilitar/reabilitar
um parceiro quando necessário.

> Em onboarding operacional este comando é **montado e enviado automaticamente** pela automação (depois de
> capturar o endereço Liquid do parceiro). Este guia cobre o uso **manual** de `/addpartner`.

---

## Sintaxe do comando

```
/addpartner <partnerid> "<name>" <telegram-group-id> <pixkey-for-deposit> <depix-address-for-deposit> <address-for-withdrawal> <pixkey-for-withdrawal>
```

| Parâmetro | Descrição |
|-----------|-----------|
| `partnerid` | Identificador único do parceiro (kebab-case, sem espaços) |
| `name` | Nome legível do parceiro (entre aspas duplas) |
| `telegram-group-id` | ID do grupo Telegram do parceiro (inteiro negativo, ex.: `-1001234567890`) |
| `pixkey-for-deposit` | Chave PIX que recebe depósitos — em geral o e-mail Depix do parceiro |
| `depix-address-for-deposit` | Endereço Depix para onde os fundos depositados são roteados |
| `address-for-withdrawal` | Endereço da rede Liquid usado para saques do parceiro (começa com `lq1` ou `VL`). Use o **endereço padrão de saque** definido pela operação, salvo se o parceiro fornecer o próprio |
| `pixkey-for-withdrawal` | Chave PIX usada para saques — em geral o e-mail Depix do parceiro |

---

## 1. Reunir a informação necessária

Antes de rodar o comando, colete do parceiro:

- **Partner ID** — um slug kebab-case único (ex.: `acme`)
- **Nome do parceiro** — nome de exibição (ex.: `"Acme"`)
- **Telegram group ID** — ver [como obter](#obter-o-telegram-group-id) abaixo
- **E-mail Depix** — usado como chaves PIX e endereço Depix na maioria dos casos (ex.: `parceiro@depix.info`)
- **Endereço Depix para depósito** — o endereço de roteamento fornecido pelo parceiro
- **Endereço da rede Liquid para saque** — começa com `lq1` ou `VL`; use o endereço padrão salvo se o
  parceiro fornecer o próprio

---

## Obter o Telegram Group ID

### Opção A — Copiar link do grupo

1. Abra o grupo Telegram do parceiro.
2. Vá a qualquer tópico dentro do grupo.
3. Toque nos **três pontos** no canto superior → **Ver info do tópico**.
4. Toque nos **três pontos** em **Mais** → **Copiar link do grupo**.
5. O link terá o formato `https://t.me/c/<telegram-group-id>/1`.
6. Extraia o número entre `/c/` e `/1` — esse é o `telegram-group-id`.

### Opção B — Habilitar peer IDs nas configurações do Telegram

1. Telegram → **Configurações** → **Avançado** → **Configurações Experimentais**.
2. Habilite **Mostrar peer IDs no perfil**.
3. Abra o grupo do parceiro — o ID ficará visível na descrição do grupo.

---

## 2. Rodar o comando `/addpartner`

Envie o comando no **grupo de log de operação** (onde o Eulen Bot responde).

**Exemplo:**

```
/addpartner acme "Acme" -1001234567890 acme@depix.info acme@depix.info <endereço-liquid-padrão> acme@depix.info
```

> O `<endereço-liquid-padrão>` é o endereço de saque padrão da operação — obtenha-o com a operação; não
> vive nesta base por ser endereço de carteira.

---

## 3. Verificar que o parceiro foi adicionado

1. Confirme que o bot responde com uma mensagem de confirmação no grupo de log.
2. Envie uma mensagem de teste no **grupo Telegram do parceiro** para confirmar que o bot responde.

---

## 4. Desabilitar e reabilitar um parceiro

No grupo de log:

```
/disable <partnerid>
```

```
/enable <partnerid>
```

---

## 5. Boas práticas

- Sempre confirme que o Telegram group ID está correto antes de rodar o comando — erros roteiam mensagens
  para o grupo errado.
- Armazene as credenciais do parceiro (endereço Depix, endereço Liquid) com segurança antes de rodar o
  comando.
- Teste a integração num ambiente de dev antes de ativar em produção.
