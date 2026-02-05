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
1. **Telegram bot** via BotFather - [telegram.md](telegram.md)
2. **GitHub account** for the bot (not your main account) - [github.md](github.md)
3. **Anthropic API key** from [console.anthropic.com](https://console.anthropic.com)

### EC2 Users Only
4. **AWS CLI** configured with credentials

### Local/Other Remote Users
4. **Node.js 18+** installed
5. **Claude Code CLI** installed (`npm install -g @anthropic-ai/claude-code`)

## Setup Guides

| Guide | When to use |
|-------|-------------|
| [telegram.md](telegram.md) | All users - Create bot, get token, get your user ID |
| [github.md](github.md) | All users - Bot GitHub account, token, repo organization |
| [EC2.md](EC2.md) | **EC2 only** - AWS instance, SSH, security group, systemd |
| [x402.md](x402.md) | All users - mcporter, @x402scan/mcp, skills installation |

## Quick Start by Scenario

### Scenario A: Fresh Install on EC2
1. Complete [telegram.md](telegram.md)
2. Complete [github.md](github.md)
3. Follow [EC2.md](EC2.md) to provision instance
4. Follow [x402.md](x402.md) to add x402 support
5. Message your Telegram bot

### Scenario B: Fresh Install on Local Machine
1. Complete [telegram.md](telegram.md)
2. Complete [github.md](github.md)
3. **Skip EC2.md** - Install OpenClaw locally instead:
   ```bash
   git clone https://github.com/anthropics/claude-code.git openclaw
   cd openclaw
   npm install
   ```
4. Follow [x402.md](x402.md) to add x402 support
5. Run OpenClaw and message your Telegram bot

### Scenario C: Existing OpenClaw Installation (any environment)
1. Verify [telegram.md](telegram.md) is complete
2. Verify [github.md](github.md) is complete
3. **Skip to [x402.md](x402.md)** to add x402 support
4. Message your Telegram bot

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
