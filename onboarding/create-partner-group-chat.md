# Create Your Own Partner Group

This guide explains how to create a dedicated **Telegram group** that includes both your Telegram user and the project’s Telegram Bot, and how to correctly configure this new partner group inside the `config.yaml`.  
This setup is required for development, partner onboarding, and automated bot workflows.

---

## 1. Create a New Telegram Group

1. Open Telegram.  
2. Tap **New Message** → **New Group**.  
3. Add **your own Telegram account**.  
4. Add the **project’s bot** using its bot username `@dev_kdoe2dcwqbot`.  

Your partner-specific group is now created. 🎉

---

## 2. Confirm Bot Membership via Telegram API

After adding the bot, Telegram sends a `my_chat_member` update such as:

```json
{
  "my_chat_member": {
    "chat": {
      "id": -5012345678,
      "title": "[DEV] YOUR_PARTNER_NAME <> Eulen",
      "type": "group"
    },
    "new_chat_member": { "status": "member" }
  }
}
```

This confirms the following:
- The bot is inside the group  
- The value `chat.id` (`-5012345678`) is the **Telegram Group ID** used in configuration  

---

## 3. Retrieve the Group Chat ID

1. Send a message in the group.  
2. Query the Bot API:

```
https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getUpdates
```

3. Locate the chat object:

```json
"chat": {
  "id": -5012345678
}
```

This number becomes the **`telegram-group-id`**.

---

## 4. Update the `config.yaml` for the New Partner

### A. Locate the `clients:` section

Example:

```yaml
clients:
  some-client:
    telegram-group-id: -100111111111
    whitelist-telegram-id:
      - 123456789
```

### B. Copy an existing client block

```yaml
clients:
  new-partner-name:
    telegram-group-id: REPLACE_ME
    whitelist-telegram-id:
      - REPLACE_ME
```

### C. Change the partner name

```yaml
clients:
  your-partner-name-dev:
```

### D. Replace the `telegram-group-id`

```yaml
telegram-group-id: -5012345678
```

### E. Add your Telegram User ID to `whitelist-telegram-id`

```yaml
whitelist-telegram-id:
  - 1234567890   # your Telegram ID
```

You can obtain your Telegram ID via:

- `@userinfobot`  
- `@getmyid_bot`  
- or your bot’s own `getUpdates` output

---

## 5. Final Example Configuration

```yaml
clients:
  your-partner-name-dev:
    telegram-group-id: -5012345678
    whitelist-telegram-id:
      - 1234567890
    enabled: true
    environment: dev
```

---

## 6. Best Practices

- Create **one group per partner**  
- Use naming convention:  
  **`[DEV|STAGE|PROD] PartnerName <> Eulen`**  
- Ensure the group ID matches the correct environment  
- Ensure your Telegram ID is included in the whitelist  

---

## 7. Summary

By following this guide, you will:

- Create the Telegram partner group  
- Add the bot and verify its presence  
- Retrieve the group’s Telegram ID  
- Register the partner inside `config.yaml`  
- Add yourself to the whitelist  
- Prepare the environment for partner integrations

This process is required for onboarding any new partner or environment.
