# OpenClaw Hardened Deployment (Ansible)

Based on [Next-Kick/openclaw-hardened-ansible](https://github.com/Next-Kick/openclaw-hardened-ansible) with additional adjustments for Tailscale access, skill installation, and LiteLLM proxy routing.

# 0. Set up local ssh keys

From terminal

`ssh-keygen`

Use all defualt settings, this will create a key in `~/.ssh` folder

Copy the contents of the public key, will be a file ending in `.pub` in the `~/.ssh` folder

# 1. Spin up a VCS

1. Sign up to Hetzner https://www.hetzner.com/
2. Once signed up go to the console https://console.hetzner.com/
3. Start a new project
4. Create a new resource
   - Use Regular Performance
   - CPX32 (8GB, 4VCPUs)
   - Location, whatever is cheapest (I chose Helsinki)
   - Use Ubuntu
   - In SSH keys section, add a ssh key, copy and paste the contents of your public key in here (copied from step 0)
   - Leave everything else the same
5. This will spin up a new server for you, copy its IP address


Test it by using command `ssh root@<IP_ADDRESS>`

You should be able to login using yours ssh keys (password not needed)


# 2. Tailscale set up

- Sign up to https://tailscale.com/
- Create an account 
- Go to `admin console -> Settings -> Keys` and generate a new auth key (check "Reusable" if deploying multiple servers), note it down for the deploy command
- Download the app locally, and add your local device to your network

# 3. Skills setup

Highly recommended, but not necessary, also I have not tested this without these skills

- Agentmail, give your its own email address, just sign up and note down the api key
- github, again sign up, and generate a PAT token from github that gives the agent full access to admin the repos

# 4. Local setup

1. Clone this repo
2. Make sure you have Ansible locally

**macOS:**
```bash
brew install ansible
```
**Pip (any OS with Python):**
```bash
pip install ansible
```

# 5. Build and set up the server


```aiignore
git clone <this-repo>
cd openclaw-setup
chmod +x deploy.sh

./deploy.sh \
    --target <SERVER_IP> \
    --ssh-user root \
    --ssh-key ~/.ssh/<PRIVATE_SSH_KEY_FILE_NAME (from step 0)> \
    --provider anthropic \
    --model claude-opus-4-6 \
    --key <ANTHROPIC_API_KEY> \
    --tailscale-key <TAILSCALE_AUTH_KEY (from step 2)> \
    --github-token <GITHUB_PAT> \
    --github-user <GITHUB_USERNAME> \
    --agentmail-key <AGENTMAIL_API_KEY>
```

This will automatically set up Tailscale on the server using your auth key.

It will print out some success message with key info including:
- The SSH command to connect: `ssh -i ssh-keys/<SOME_RANDOM_WORDS>.pem openclaw@<IP_ADDRESS>`
- The dashboard URL: `https://<HOSTNAME>.<TAILNET>:18789`

# 6. Post deployment steps

1. Make sure your local device is also on the tailnet
2. Access the dashboard at `https://<FULL_DNS>:18789` e.g. `https://ubuntu-8gb-hel1-2.tailae3453.ts.net:18789`
3. This should open the openclaw dashboard with an error message saying `disconnected (1008): unauthorized: gateway token missing (open the dashboard URL and paste the token in Control UI settings)`
4. Get the openclaw token
   - Login to the server as openclaw using the ssh command that was printed after the successful deployment, something like `ssh -i ssh-keys/gigabyte-seclusion-battery.pem openclaw@<IP_ADDRESS>`
   - Run the command `podman exec openclaw-agent openclaw config get gateway.auth.token`
   - Copy this token
5. Paste it in the dashboard `Overview -> Gateway Access -> Gateway Token`
6. Now you need to pair your machine with openclaw
   - Login to the server as openclaw using the ssh command that was printed after the successful deployment, something like `ssh -i ssh-keys/gigabyte-seclusion-battery.pem openclaw@<IP_ADDRESS>`
   - Run the command `podman exec openclaw-agent openclaw devices list --json`
   - This will list all the device requests for the dashboard, copy the requestId for your machine
   - Then run `podman exec openclaw-agent openclaw devices approve <REQUEST_ID> --profile operator` to approve


This should now make your agent come alive and you can use it to set the rest of the things

- Give your bot a persona
- Ask it to set telegram
- Ask it to set its email account
