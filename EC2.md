# EC2 Setup for OpenClaw Gateway

## Prerequisites

- AWS account
- AWS CLI installed (`brew install awscli` on macOS)
- SSH key pair
- Telegram bot token (from [@BotFather](https://t.me/BotFather))
- Anthropic API key (from [console.anthropic.com](https://console.anthropic.com))

---

**Next:** After completing this guide, continue to [x402.md](x402.md).

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

Save the security group ID (e.g., `sg-0123456789abcdef0`).

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

# Install OpenClaw globally
sudo npm install -g openclaw

# Verify
openclaw --version
```

## 7. Configure OpenClaw

```bash
# Set gateway mode
openclaw config set gateway.mode local

# Configure Telegram
openclaw config set channels.telegram.default.token "<your-telegram-bot-token>"

# Pair yourself (get your Telegram user ID by messaging @userinfobot)
openclaw pair add <your-telegram-user-id> --channel telegram
```

## 8. Create Systemd Service

```bash
sudo tee /etc/systemd/system/openclaw.service > /dev/null << 'EOF'
[Unit]
Description=OpenClaw Gateway
After=network.target

[Service]
Type=simple
User=ubuntu
Environment=NODE_ENV=production
Environment=ANTHROPIC_API_KEY=<your-anthropic-api-key>
Environment=OPENCLAW_GATEWAY_TOKEN=<choose-a-secret-token>
ExecStart=/usr/bin/openclaw gateway run --bind loopback --port 18789 --force
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF
```

Replace:
- `<your-anthropic-api-key>` with your Anthropic API key
- `<choose-a-secret-token>` with a random string (e.g., `openssl rand -hex 16`)

## 9. Start Service

```bash
sudo systemctl daemon-reload
sudo systemctl enable openclaw
sudo systemctl start openclaw

# Check status
sudo systemctl status openclaw
```

## 10. Verify

```bash
# Check gateway is running
openclaw channels status --probe
```

You should see:
```
- Telegram default: enabled, configured, running, works
```

Now message your Telegram bot - it should respond.

## Troubleshooting

### Gateway crash loop (lock file)
```bash
sudo systemctl stop openclaw
rm -rf /tmp/openclaw-*
sudo systemctl start openclaw
```

### Disk full
```bash
# Check disk usage
df -h /

# Clear npm cache
sudo rm -rf /root/.npm ~/.npm

# Trim journal logs
sudo journalctl --vacuum-size=20M
```

### Check logs
```bash
sudo journalctl -u openclaw -n 100 --no-pager
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
