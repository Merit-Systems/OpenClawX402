# OpenClaw + x402 Sandbox

Run an autonomous AI agent on EC2 with access to x402 payment APIs (USDC on Base).

## Prerequisites

Before starting, prepare:

1. **Telegram bot** via BotFather - [telegram.md](telegram.md)
2. **GitHub account** for the bot (not your main account) - [github.md](github.md)
3. **Anthropic API key** from [console.anthropic.com](https://console.anthropic.com)
4. **AWS CLI** configured with credentials

## Setup Guides

| Guide | What it covers |
|-------|----------------|
| [telegram.md](telegram.md) | Create bot, get token, get your user ID |
| [github.md](github.md) | Bot GitHub account, token, repo organization |
| [EC2.md](EC2.md) | AWS instance, SSH, security group, systemd |
| [x402.md](x402.md) | mcporter, @x402scan/mcp, skills installation |

## Quick Start

1. Complete [telegram.md](telegram.md) first (5 min)
2. Complete [github.md](github.md) (5 min)
3. Follow [EC2.md](EC2.md) to provision instance (15 min)
4. Follow [x402.md](x402.md) to wire up x402 tools (10 min)
5. Message your Telegram bot

## Architecture

```
You (Telegram) --> EC2 Gateway --> Claude API
                        |
                        +--> mcporter --> @x402scan/mcp --> USDC payments
```

## Cost

- EC2 t3.small: ~$15/month
- EBS 25GB: ~$2/month
- Anthropic API: pay-per-use
- x402 calls: pay-per-call (USDC)
