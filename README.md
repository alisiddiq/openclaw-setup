# OpenClaw Hardened Deployment (Ansible)

Based on [Next-Kick/openclaw-hardened-ansible](https://github.com/Next-Kick/openclaw-hardened-ansible) with additional adjustments for Tailscale access, skill installation, and LiteLLM proxy routing.

# Set up local ssh keys

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
   - Location Helsinki
   - Use Ubuntu
   - In SSH keys section, add a ssh key, copy and paste the contents of your public key in here
   - Leave everything else the same
5. This will spin up a new server for you, copy its IP address


Test it by using command `ssh root@<IP_ADDRESS>`

You should be able to login using yours ssh keys (password not needed)


# 2. Tailscale set up

- Sign up to https://tailscale.com/
- Create an account 
- Go to admin console -> DNS -> Tailnet DNS Name, note that down, will be something like `tailae3453.ts.net`
- Download the app locally, and add your local device to your network

# Skills setup

Highly recommended, but not necessary, also I have not tested this without these skills

- Agentmail, give your its own email address, just sign up and note down the api key
- github, again sign up, and generate a PAT token from github that gives the agent full access to admin the repos

# Local setup

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

# Build and set up the server


```aiignore
git clone <this-repo>
cd openclaw-setup
chmod +x deploy.sh

⏺ ./deploy.sh \                                                                                                                                                                                                                                                                                                          
    --target <SERVER_IP> \                                                                                                                                                                                                                                                                                               
    --ssh-user root \                                                                                                                                                                                                                                                                                                    
    --ssh-key ~/.ssh/<PRIVATE_SSH_KEY_FILE_NAME> \                                                                                                                                                                                                                                                                                        
    --provider anthropic \                                                                                                                                                                                                                                                                                               
    --model claude-opus-4-6 \                                                                                                                                                                                                                                                                                            
    --key <ANTHROPIC_API_KEY> \
    --tailnet <TAILNET_DNS_NAME (from step 2)> \                                                                                                                                                                                                                                                                                        
    --github-token <GITHUB_PAT> \                                                                                                                                                                                                                                                                                        
    --github-user <GITHUB_USERNAME> \
    --agentmail-key <AGENTMAIL_API_KEY>
```

This will print out some success message with key info, copy the line that looks like 

`🌐 Connect using: ssh -i ssh-keys/<SOME_RANDOM_WORDS>.pem openclaw@<IP_ADDRESS>`

You will need the above ssh command later

Add the server to your tailscale network

```aiignore
ssh root@<IP_ADDRESS>
sudo tailscale up
```

This will bring up a login link, login to your account and authorize to add the server to your network

Note down the hostname of the server

`tailscale dns status`

This will print something like ...

```aiignore
MagicDNS: enabled tailnet-wide (suffix = tailae3453.ts.net)

Other devices in your tailnet can reach this device at ubuntu-8gb-hel1-2.tailae3453.ts.net.
```

Note down the `ubuntu-8gb-hel1-2.tailae3453.ts.net` 

# Post deployment steps


1. Make sure your local device is also on the tailnet
   - Download tailscale app for your device
   - Login
   - Add it to the network
3. Access the dashboard at `https://<FULL_DNS>:18789` e.g. `https://ubuntu-8gb-hel1-2.tailae3453.ts.net:18789`
4. This should open the openclaw dashboard with an error message saying `disconnected (1008): unauthorized: gateway token missing (open the dashboard URL and paste the token in Control UI settings)`
5. Get the openclaw token
   - Login to the server as openclaw using the ssh command that was printed after the successful deployment, something like `ssh -i ssh-keys/gigabyte-seclusion-battery.pem openclaw@<IP_ADDRESS>`
   - Run the command `podman exec openclaw-agent openclaw config get gateway.auth.token`
   - Copy this token
6. Paste it in the dashboard `Overview -> Gateway Access -> Gateway Token`
7. Now you need to pair your machine with openclaw
   - Login to the server as openclaw using the ssh command that was printed after the successful deployment, something like `ssh -i ssh-keys/gigabyte-seclusion-battery.pem openclaw@<IP_ADDRESS>`
   - Run the command `podman exec openclaw-agent openclaw devices list --json`
   - This will list all the device requests for the dashboard, copy the requestId for your machine
   - Then run `podman exec openclaw-agent openclaw devices approve <REQUEST_ID> --profile operator` to approve


This should now make your agent come alive and you can use it to set the rest of the things

- Ask it to set telegram
- Ask it to set its email account
