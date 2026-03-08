# EC2 Setup for OpenClaw Gateway

## Prerequisites

- AWS account
- AWS CLI installed (`brew install awscli` on macOS)
- SSH key pair
- Telegram bot token (from [@BotFather](https://t.me/BotFather))
- Anthropic API key (from [console.anthropic.com](https://console.anthropic.com))

---

**Next:** After completing this guide, continue to [`./x402.md`](./x402.md).

## 1. AWS CLI Setup

*Skip if you already have AWS CLI configured.*

```bash
# Install
brew install awscli

# Configure
aws configure
# Enter: Access Key ID, Secret Access Key, region (us-east-1), output format (json)

# Verify
aws sts get-caller-identity
```

## 2. Create Security Group

```bash
# Create security group
aws ec2 create-security-group \
  --group-name openclaw-gateway \
  --description "OpenClaw gateway SSH access"

# Allow SSH from anywhere (or restrict to your IP)
aws ec2 authorize-security-group-ingress \
  --group-name openclaw-gateway \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0

# Get the security group ID
aws ec2 describe-security-groups \
  --group-names openclaw-gateway \
  --query 'SecurityGroups[0].GroupId' \
  --output text
```

**SAVE** the security group ID (e.g., `sg-0123456789abcdef0`) - you'll need it for launch.

## 3. Import SSH Key

```bash
# Import your existing public key
aws ec2 import-key-pair \
  --key-name openclaw-key \
  --public-key-material fileb://~/.ssh/id_ed25519.pub
```

Or create a new key pair:
```bash
aws ec2 create-key-pair \
  --key-name openclaw-key \
  --query 'KeyMaterial' \
  --output text > ~/.ssh/openclaw-key.pem
chmod 400 ~/.ssh/openclaw-key.pem
```

## 4. Launch Instance

```bash
# Find latest Ubuntu 24.04 AMI (us-east-1)
AMI_ID=$(aws ec2 describe-images \
  --owners 099720109477 \
  --filters "Name=name,Values=ubuntu/images/hvm-ssd-gp3/ubuntu-noble-24.04-amd64-server-*" \
  --query 'Images | sort_by(@, &CreationDate) | [-1].ImageId' \
  --output text)

# Launch t3.small with 25GB disk
aws ec2 run-instances \
  --image-id $AMI_ID \
  --instance-type t3.small \
  --key-name openclaw-key \
  --security-group-ids <your-sg-id> \
  --block-device-mappings '[{"DeviceName":"/dev/sda1","Ebs":{"VolumeSize":25,"VolumeType":"gp3"}}]' \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=openclaw-gateway}]' \
  --query 'Instances[0].InstanceId' \
  --output text
```

Save the instance ID.

## 5. Get Public IP

```bash
# Wait for instance to start, then get IP
aws ec2 describe-instances \
  --instance-ids <instance-id> \
  --query 'Reservations[0].Instances[0].PublicIpAddress' \
  --output text
```

## 6. SSH and Install OpenClaw

```bash
# SSH into instance
ssh ubuntu@<public-ip>

# Install Node.js 22
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install OpenClaw using the official installer (recommended)
curl -fsSL https://openclaw.ai/install.sh | bash

# Verify
openclaw --version
```

## 7. Configure OpenClaw with Onboard Wizard

The recommended way to configure OpenClaw is using the **onboard wizard**, which handles auth, gateway settings, daemon installation, and channel configuration:

```bash
# Interactive mode (recommended for first-time setup)
openclaw onboard --install-daemon

# Or non-interactive mode for scripted setups:
export ANTHROPIC_API_KEY="your-anthropic-api-key"
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice apiKey \
  --anthropic-api-key "$ANTHROPIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback \
  --install-daemon \
  --daemon-runtime node
```

See [Onboarding Wizard docs](https://docs.openclaw.ai/start/wizard#non-interactive-mode) for all options.

## 8. Add Telegram Channel

After onboarding, add your Telegram bot:

```bash
# Add Telegram channel via CLI
openclaw channels add --channel telegram --token "$TELEGRAM_BOT_TOKEN"

# Configure DM policy (choose one):
# Option A: Pairing (default) - approve users via pairing codes
openclaw pairing list telegram  # See pending requests after someone DMs the bot
openclaw pairing approve telegram <CODE>

# Option B: Allowlist - allow specific user IDs
openclaw config set channels.telegram.dmPolicy allowlist
openclaw config set channels.telegram.allowFrom '[YOUR_TELEGRAM_USER_ID]'

# Restart gateway to apply changes
systemctl --user restart openclaw-gateway
```

**Finding your Telegram User ID** (no third-party bot needed):
```bash
# DM your bot, then check the logs:
openclaw logs --follow
# Look for "from.id" in the output
```

## 9. Configure Git for Bot Account

Configure git for your bot account (see [`./github.md#3-configure-git-on-ec2`](./github.md#3-configure-git-on-ec2)):

```bash
git config --global user.name "yourname-clawdbot"
git config --global user.email "yourname+clawdbot@gmail.com"
git config --global credential.helper store
```

## 10. Verify Gateway Status

The `onboard --install-daemon` command creates a **user-level** systemd service:

```bash
# Check gateway status
openclaw gateway status
openclaw status --deep  # More detailed probe

# Manage the service
systemctl --user status openclaw-gateway
systemctl --user restart openclaw-gateway

# View logs
openclaw logs --follow
# Or via journalctl:
journalctl --user -u openclaw-gateway -f
```

**Note:** The service is at `~/.config/systemd/user/openclaw-gateway.service` (user-level, not system-level).

## 11. Verify

```bash
# Check gateway and channels status
openclaw status --deep
openclaw channels status

# Run health check
openclaw health

# Run diagnostics if something seems wrong
openclaw doctor
```

Now message your Telegram bot - it should respond (approve pairing if using pairing mode).

## Troubleshooting

### Gateway crash loop (lock file)
```bash
systemctl --user stop openclaw-gateway
rm -rf /tmp/openclaw-*
systemctl --user start openclaw-gateway
```

### Disk full
```bash
# Check disk usage
df -h /

# Clear npm cache
rm -rf ~/.npm

# Trim journal logs
sudo journalctl --vacuum-size=20M
```

### Check logs
```bash
# Preferred: use the CLI
openclaw logs --follow

# Or via journalctl (user-level service)
journalctl --user -u openclaw-gateway -n 100 --no-pager
```

### Run diagnostics
```bash
openclaw doctor
```

## Instance Management

```bash
# Stop instance (keeps data, stops billing for compute)
aws ec2 stop-instances --instance-ids <instance-id>

# Start instance (IP may change)
aws ec2 start-instances --instance-ids <instance-id>

# Terminate instance (deletes everything)
aws ec2 terminate-instances --instance-ids <instance-id>
```

## Cost Summary

| Resource | Monthly Cost |
|----------|-------------|
| t3.small (2GB RAM) | ~$15 |
| 25GB gp3 EBS | ~$2 |
| Data transfer | ~$1 |
| **Total** | **~$18/month** |
