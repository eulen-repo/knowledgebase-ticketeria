# Create Your Own Telegram Bot

This guide explains how to create your own Telegram Bot using **BotFather**, retrieve the bot token, test the bot, and prepare it for integration into Eulen partner workflows.

---

## 1. Start a Chat With BotFather

1. Open Telegram.  
2. Search for **@BotFather**.  
3. Open the chat and press **Start**.

---

## 2. Create a New Bot

Send the command:

```
/newbot
```

BotFather will ask:

1. **Bot name** – human‑readable name (e.g., `Partner DEV Bot`).  
2. **Bot username** – must end in `bot` (e.g., `partnerdev_bot`).

If the username is available, BotFather creates your bot.

---

## 3. Receive Your Bot Token

BotFather will respond with a message containing something like:

```
Use this token to access the HTTP API:
1234567890:ABCdefGHI_jklMNOPqrstUVWXyZ123456
```

This is your **Bot Token**, required for all API calls.

⚠️ **Keep this token secret**  
Anyone with this token controls your bot.

---

## 4. Test Your Bot

Send a simple request using your browser or curl:

```
https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getMe
```

If everything is correct, the API returns:

```json
{
  "ok": true,
  "result": {
    "id": 7430591373,
    "is_bot": true,
    "first_name": "partnerp2pdev_bot",
    "username": "partnerp2pdev_bot"
  }
}
```

---

## 5. Add the Bot to a Group (Optional)

If the bot will operate inside a partner group:

1. Create the group.  
2. Add your bot by its username (e.g., `@partnerp2pdev_bot`).  
3. Send any message in the group.  
4. Call:

```
https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getUpdates
```

Telegram will return a `my_chat_member` event with:

- Group name  
- Group ID (`chat.id`)  
- Bot membership status  

You will need this ID in your `config.yaml`.

---

## 6. Regenerate Your Token (If Needed)

If your token is compromised, regenerate it:

```
/revoke
```

Or:

```
/token
```

BotFather will issue a new one immediately.

---

## 7. Summary

By following this guide, you will:

- Create your own Telegram bot  
- Retrieve a secure bot token  
- Test the bot via Telegram’s HTTP API  
- Optionally add it to a partner group  
- Prepare the bot for backend integration  

Your bot is now ready to participate in partner workflows, Eulen automations, and internal tools.
