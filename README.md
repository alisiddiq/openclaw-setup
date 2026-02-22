# OpenClaw Hardened Deployment (Ansible)

Based on [Next-Kick/openclaw-hardened-ansible](https://github.com/Next-Kick/openclaw-hardened-ansible) with additional adjustments for Tailscale access, skill installation, and provider-conditional routing.

## Step 1: Install Ansible

Ansible runs on your local machine and configures the server remotely.

**macOS:**
```bash
brew install ansible
```

**Ubuntu/Debian:**
```bash
sudo apt update
sudo apt install -y ansible
```

**Pip (any OS with Python):**
```bash
pip install ansible
```

Verify it's installed:
```bash
ansible --version
```

## Step 2: Generate an SSH Key

An SSH key lets your machine connect to the server securely without a password. Skip this if you already have one at `~/.ssh/id_ed25519`.

```bash
ssh-keygen -t ed25519 -C "your-email@example.com"
```

When prompted:
- **File location**: press Enter to accept the default (`~/.ssh/id_ed25519`)
- **Passphrase**: press Enter for no passphrase (or set one if you prefer)

This creates two files:
- `~/.ssh/id_ed25519` — your **private key** (never share this)
- `~/.ssh/id_ed25519.pub` — your **public key** (this goes on the server)

## Step 3: Copy Your Key to the Server

The server needs your public key so it lets you connect without a password:

```bash
ssh-copy-id root@<SERVER_IP>
```

You'll be asked for the server's root password once. After this, password-free login is set up.

Test it works:
```bash
ssh root@<SERVER_IP>
```

If you get in without a password prompt, you're good. Type `exit` to disconnect.

## Step 4: Install Tailscale

Tailscale creates a private network between your devices. The OpenClaw dashboard is only accessible through Tailscale — it's never exposed to the public internet.

### Create an account

Go to [https://login.tailscale.com/start](https://login.tailscale.com/start) and sign up (free for personal use).

### Install on your local machine

macOS:
```bash
brew install tailscale
```
Or download from [https://tailscale.com/download/mac](https://tailscale.com/download/mac).

Ubuntu/Debian:
```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

Windows: Download from [https://tailscale.com/download/windows](https://tailscale.com/download/windows).

### Install on your server

```bash
ssh root@<SERVER_IP>
curl -fsSL https://tailscale.com/install.sh | sh
tailscale up
```

Follow the link it prints to authorize the machine in your Tailscale admin console. Type `exit` to return to your local machine.

### Verify

Open [https://login.tailscale.com/admin/machines](https://login.tailscale.com/admin/machines) — you should see both your local machine and the server listed.

## Step 5: Deploy

```bash
git clone <this-repo>
cd openclaw-hardened-ansible
chmod +x deploy.sh
```

Interactive mode (prompts for everything):
```bash
./deploy.sh
```

Or pass everything on the command line:
```bash
./deploy.sh \
  --target <SERVER_IP> \
  --ssh-user root \
  --ssh-key ~/.ssh/id_ed25519 \
  --provider anthropic \
  --model claude-opus-4-6 \
  --key <ANTHROPIC_API_KEY> \
  --github-token <GITHUB_PAT> \
  --github-user <GITHUB_USERNAME> \
  --agentmail-key <AGENTMAIL_API_KEY> \
  --non-interactive
```

The `--github-token`, `--github-user`, and `--agentmail-key` flags are optional. Only include them if you have the keys.

The first deploy takes 5-10 minutes (building the container image). Subsequent deploys are faster.

## Step 6: Access the Dashboard

The dashboard is only accessible via your Tailscale network. Open in your browser:

```
https://<hostname>.tailXXXXXX.ts.net:18789
```

You can find the hostname in the [Tailscale admin console](https://login.tailscale.com/admin/machines).

Get your gateway token to log in:
```bash
ssh openclaw@<SERVER_IP> "cat ~/openclaw-docker/.env | grep OPENCLAW_GATEWAY_TOKEN"
```

Paste this token into the dashboard when prompted.

## Step 7: Approve Your Device

The first time you connect from a new browser, you need to approve it from the server. This prevents unauthorized access even if someone has the token.

SSH into the server:
```bash
ssh openclaw@<SERVER_IP>
```

List pending devices and approve yours:
```bash
podman exec openclaw-agent openclaw devices list
podman exec openclaw-agent openclaw devices approve --device <DEVICE_ID> --role operator
```

Refresh the browser and you should be in.

## Troubleshooting

**"Permission denied" when SSHing:**
Your SSH key isn't on the server. Run `ssh-copy-id root@<IP>` or use `--ask-pass` with the deploy script.

**Dashboard not loading:**
Make sure Tailscale is running on both your local machine and the server. The dashboard is only accessible via Tailscale, not the public IP.

**"pairing required" in the dashboard:**
You need to approve your device from the server CLI. See Step 7.

**Agent not responding to messages:**
Check the agent logs: `podman logs openclaw-agent`. Verify your API key is correct and the model name matches your provider.

## License

Provided as-is. OpenClaw with LLMs carries inherent prompt injection risks. Use dedicated accounts and API keys with minimal permissions.
