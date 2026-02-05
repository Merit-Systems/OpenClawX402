# OpenClaw + x402 Sandbox

Run an autonomous AI agent with access to x402 payment APIs (USDC on Base). Works on local machines, EC2, or any remote server.

## Paste this into your agent

```text
Read https://github.com/Merit-Systems/OpenClawX402 and help me set up OpenClaw with x402.
```

## Agent Instructions

**Be maximally proactive.** Do everything on the user's behalf that doesn't require their input. Only ask questions when you genuinely need information you cannot determine yourself.

### Step 1: Gather Required Information

Use the **AskUserQuestion tool** to ask these questions upfront (combine into a single prompt when possible):

1. **Do you already have OpenClaw installed?** (Yes / No)
2. **Where is OpenClaw installed (or where do you want to install it)?**
   - Local machine (macOS/Linux)
   - AWS EC2 instance
   - Other remote server (VPS, DigitalOcean, etc.)
3. **If yes to #1: What is the path to your OpenClaw installation?** (e.g., `~/openclaw`, `/home/ubuntu/openclaw`)

### Step 2: Verify and Execute

Once you have the user's answers:

- **Verify the installation** by checking for `settings.json` and `.claude/` directory at the provided path
- **Run all commands yourself** - don't ask the user to run them unless you cannot (e.g., no SSH access)
- **Install dependencies automatically** - mcporter, skills, etc.
- **Create directories and symlinks** without asking
- **Only pause for user input when required** - e.g., funding the wallet (requires their action), or if a command fails and you need clarification

### What NOT to ask about

- Don't ask permission to install mcporter, skills, or create symlinks - just do it
- Don't ask which commands to run - execute them based on the user's environment
- Don't ask the user to copy/paste commands - run them yourself
- Don't ask for confirmation at each step - proceed through the guide automatically

---

## Prerequisites

Based on the user's answers, they need different prerequisites:

### All Users
1. **Telegram bot** via BotFather - [`./telegram.md`](./telegram.md)
2. **GitHub account** for the bot (not your main account) - [`./github.md`](./github.md)
3. **Anthropic API key** from [console.anthropic.com](https://console.anthropic.com)

### EC2 Users Only
4. **AWS CLI** configured with credentials

### Local/Other Remote Users
4. **Node.js 18+** installed
5. **Claude Code CLI** installed (`npm install -g @anthropic-ai/claude-code`)

## Setup Guides

| Guide | When to use |
|-------|-------------|
| [`./telegram.md`](./telegram.md) | All users - Create bot, get token, get your user ID |
| [`./github.md`](./github.md) | All users - Bot GitHub account, token, repo organization |
| [`./EC2.md`](./EC2.md) | **EC2 only** - AWS instance, SSH, security group, systemd |
| [`./x402.md`](./x402.md) | All users - mcporter, @x402scan/mcp, skills installation |

## Quick Start by Scenario

### Scenario A: Fresh Install on EC2
1. Complete [`./telegram.md`](./telegram.md)
2. Complete [`./github.md`](./github.md)
3. Follow [`./EC2.md`](./EC2.md) to provision instance and install Node.js
4. **Run the onboard command** (see [official docs](https://docs.openclaw.ai/start/wizard#non-interactive-mode)):
   ```bash
   openclaw onboard --non-interactive --accept-risk \
     --mode local \
     --auth-choice apiKey \
     --anthropic-api-key "$ANTHROPIC_API_KEY" \
     --gateway-port 18789 \
     --gateway-bind loopback \
     --install-daemon \
     --daemon-runtime node
   ```
5. Configure Telegram (see [Telegram docs](https://docs.openclaw.ai/channels/telegram)):
   ```bash
   openclaw plugins enable telegram
   openclaw channels add --channel telegram --token "$TELEGRAM_BOT_TOKEN"
   openclaw config set channels.telegram.dmPolicy allowlist
   openclaw config set channels.telegram.allowFrom '[YOUR_TELEGRAM_USER_ID]'
   systemctl --user restart openclaw-gateway
   ```
6. Follow [`./x402.md`](./x402.md) to add x402 support
7. Message your Telegram bot

### Scenario B: Fresh Install on Local Machine
1. Complete [`./telegram.md`](./telegram.md)
2. Complete [`./github.md`](./github.md)
3. **Skip EC2.md** - Install OpenClaw locally instead:
   ```bash
   git clone https://github.com/anthropics/claude-code.git openclaw
   cd openclaw
   npm install
   ```
4. Follow [`./x402.md`](./x402.md) to add x402 support
5. Run OpenClaw and message your Telegram bot

### Scenario C: Existing OpenClaw Installation (any environment)
1. Verify [`./telegram.md`](./telegram.md) is complete
2. Verify [`./github.md`](./github.md) is complete
3. **Skip to [`./x402.md`](./x402.md)** to add x402 support
4. Message your Telegram bot

## Repository Structure

```
.
├── README.md      # This file - overview and agent instructions
├── telegram.md    # Telegram bot setup via BotFather
├── github.md      # Bot GitHub account and token setup
├── EC2.md         # AWS EC2 instance provisioning (optional)
└── x402.md        # x402 payment integration
```

## Architecture

```
You (Telegram) --> OpenClaw Host --> Claude API
                        |
                        +--> mcporter --> @x402scan/mcp --> USDC payments
```

The "OpenClaw Host" can be:
- Your local machine (macOS/Linux)
- An EC2 instance
- Any server with Node.js 18+

## Cost

### All Environments
- Anthropic API: pay-per-use
- x402 calls: pay-per-call (USDC)

### EC2 Only
- EC2 t3.small: ~$15/month
- EBS 25GB: ~$2/month

### Local Machine
- No hosting costs

## Common Issues & Lessons Learned

### For Fresh Installs: Read the Official Docs

When installing OpenClaw from scratch, **read the official documentation first**:

- **Non-interactive onboarding**: https://docs.openclaw.ai/start/wizard#non-interactive-mode
- **Telegram channel setup**: https://docs.openclaw.ai/channels/telegram

### Use the onboard command, not manual configuration

For fresh installs, **don't manually configure** `openclaw.json`, systemd services, and gateway tokens piece by piece. The `openclaw onboard --non-interactive` command handles all of this correctly:

- Creates proper config structure
- Sets up gateway authentication
- Installs the systemd daemon (user-level service)
- Enables required plugins

### Telegram Setup Issues

1. **Enable the plugin first**: Before adding a Telegram channel, enable the plugin:
   ```bash
   openclaw plugins enable telegram
   ```

2. **Pairing vs Allowlist**: By default, `dmPolicy: "pairing"` requires users to be approved via:
   ```bash
   openclaw pairing list --channel telegram
   openclaw pairing approve <CODE> --channel telegram
   ```

   Alternatively, use `dmPolicy: "allowlist"` with explicit user IDs:
   ```bash
   openclaw config set channels.telegram.dmPolicy allowlist
   openclaw config set channels.telegram.allowFrom '[YOUR_TELEGRAM_USER_ID]'
   ```

3. **Finding your Telegram User ID**: Message `@getmyid_bot` on Telegram (not your bot's ID which appears at the start of the bot token).

4. **Restart after config changes**: Always restart the gateway after config changes:
   ```bash
   systemctl --user restart openclaw-gateway
   ```

### Gateway Token Errors

If you see `"Gateway auth is set to token, but no token is configured"`, the gateway will crash-loop. The `onboard` command sets this up automatically. If manually configuring:
```bash
openclaw config set gateway.auth.token "$(openssl rand -hex 32)"
```

### Systemd Service Location

The `onboard --install-daemon` creates a **user-level** systemd service at:
```
~/.config/systemd/user/openclaw-gateway.service
```

Manage it with:
```bash
systemctl --user start openclaw-gateway
systemctl --user status openclaw-gateway
journalctl --user -u openclaw-gateway -f
```

### Checking Logs

For detailed logs:
```bash
# Systemd logs
journalctl --user -u openclaw-gateway -n 50 --no-pager

# Full log file
tail -100 /tmp/openclaw/openclaw-$(date +%Y-%m-%d).log

# Telegram-specific logs
grep -i telegram /tmp/openclaw/openclaw-$(date +%Y-%m-%d).log | tail -30
```
