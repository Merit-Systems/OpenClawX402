# Telegram Bot Setup

Create a Telegram bot to interact with your OpenClaw agent.

---

**Next:** After completing this guide, continue to [`./github.md`](./github.md).

---

## 1. Create Bot via BotFather

1. Open Telegram and search for [@BotFather](https://t.me/BotFather)
2. Start a chat and send `/newbot`
3. Follow the prompts:
   - **Name:** Your bot's display name (e.g., "My ClawdBot")
   - **Username:** Must end in `bot` (e.g., `myclawdbot` or `my_clawd_bot`)
4. BotFather will give you a **token** like:
   ```
   1234567890:ABCdefGHIjklMNOpqrsTUVwxyz
   ```
5. **Save this token** - you'll need it for EC2 setup

## 2. Get Your User ID

You need your Telegram user ID to pair with the bot.

1. Search for [@userinfobot](https://t.me/userinfobot) on Telegram
2. Start a chat and send any message
3. It will reply with your user ID (a number like `123456789`)
4. **Save this ID** - you'll need it for pairing

## 3. Optional: Set Bot Profile Picture

1. Go back to [@BotFather](https://t.me/BotFather)
2. Send `/setuserpic`
3. Select your bot
4. Send an image

## 4. Optional: Set Bot Description

1. Send `/setdescription` to BotFather
2. Select your bot
3. Enter a description (shown when users first open the bot)

## What You'll Need Later

When you reach the EC2 setup, you'll use these values:

```bash
# Configure Telegram token
openclaw config set channels.telegram.default.token "<your-bot-token>"

# Pair your user ID
openclaw pair add <your-user-id> --channel telegram
```

## Security Notes

- Never share your bot token publicly
- Anyone with the token can control your bot
- If compromised, regenerate via `/revoke` in BotFather
