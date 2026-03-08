# GitHub Setup for ClawdBot

---

**Next:** After completing this guide, continue to [`./EC2.md`](./EC2.md).

---

## Why a Separate Account?

Create a **dedicated GitHub account** for your bot. This protects your main account from:

- Rogue agent behavior
- Prompt injection attacks
- Accidental commits to wrong repos

Your bot gets its own identity with limited blast radius.

## Steps

### 1. Create Bot GitHub Account

1. Go to [github.com/signup](https://github.com/signup)
2. Use a dedicated email (e.g., `yourname+clawdbot@gmail.com`)
3. Username suggestion: `yourname-clawdbot` or similar

### 2. Generate Classic Personal Access Token

1. Log into the bot account
2. Go to **Settings > Developer settings > Personal access tokens > Tokens (classic)**
3. Click **Generate new token (classic)**
4. Set expiration (90 days recommended for security)
5. Select scopes:
   - `repo` (full control of private repos)
   - `workflow` (if using GitHub Actions)
6. Copy the token immediately (you won't see it again)

### 3. Configure Git on EC2

After EC2 setup, configure git with the bot account:

```bash
git config --global user.name "yourname-clawdbot"
git config --global user.email "yourname+clawdbot@gmail.com"
git config --global credential.helper store
```

First push will prompt for credentials:
- Username: `yourname-clawdbot`
- Password: `<your-classic-token>`

> **Note:** `credential.helper store` saves credentials in plaintext at `~/.git-credentials`. This is acceptable for a dedicated bot account with limited repo access.

### 4. Repository Organization

Use this directory pattern on all machines:

```
~/Code/<owner>/<repo>
```

Examples:
```
~/Code/merit-systems/OpenClawX402
~/Code/yourname-clawdbot/my-project
~/Code/openclaw/openclaw
```

This keeps repos organized by owner and makes collaboration cleaner.

## Security Notes

- Never give the bot account access to sensitive repos
- Review bot commits periodically
- Rotate the token every 90 days
- Consider repo-scoped tokens for tighter control
