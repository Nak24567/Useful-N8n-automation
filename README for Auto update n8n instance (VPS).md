# n8n Auto-Update Workflow (Self-Updating on Same Instance)

This workflow **automatically updates the very same n8n instance it runs on** — without you ever opening a terminal again.

Every hour (or when you manually click **“Execute Workflow”**), it checks whether a newer Docker image exists for the `latest` tag. It does this by comparing the **image digest** of the currently running container with the digest published on Docker Hub — a rock-solid method that works even when version numbers are misleading.

If an update is available, the workflow fetches the official release notes from GitHub, feeds them into **Google Gemini** (or any LLM you prefer), and generates a short, plain-English summary that highlights only what matters:

- New nodes  
- Breaking changes  
- Security fixes  
- Major enhancements  

It **ignores** dependency updates, documentation tweaks, and minor bug fixes.

Next, it sends you a **Telegram message** showing the new version number and the AI summary, along with two buttons:  
**Approve** or **Reject**. You stay in full control.

Only when you tap **Approve** does the update begin. The workflow connects to the **same server** via **SSH**, changes into the directory that holds your `docker-compose.yml`, and runs this exact command in the background:

```bash
nohup sh -c "docker compose pull && docker compose down && docker compose up -d [worker_command]" &gt; update.log 2&gt;&1 &
Here’s why that works even though n8n is shutting itself down:
	•	nohup detaches the process from the SSH session.
	•	& runs it in the background without turning off your instance.
	•	All output and errors are saved to update.log in your project folder.
Within seconds everything is done and you won’t even feel it.

What You’ll Need (and Exactly How to Get Each One)

1. Your n8n Running via Docker Compose on a VPS
Make sure you started it with:
docker compose up -d
Note the exact folder where docker-compose.yml lives — you’ll enter this path in the workflow.

P.s: if you use hostinger one click install chances are the path is /root

2. SSH Access to That Same VPS (Key Recommended)
Generate an SSH key pair on your local machine:
ssh-keygen -t ed25519 -C "n8n-auto-updater"
Press Enter three times (default location, no passphrase).
Copy the public key:
cat ~/.ssh/id_ed25519.pub
Log into your VPS (password or existing key):
ssh root@your-vps-ip
Add the public key:
mkdir -p ~/.ssh
nano ~/.ssh/authorized_keys
Paste the key, save (Ctrl+O → Enter → Ctrl+X).
Secure permissions:
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh
Test passwordless login:
exit
ssh root@your-vps-ip
→ No password prompt = success.
In n8n → Credentials → New Credential → SSH Key
	•	Paste the private key (cat ~/.ssh/id_ed25519)
	•	Enter VPS IP and username (usually root)

3. Telegram Bot Token + Your Personal Chat ID
	1	Open Telegram → talk to @BotFather
	2	Send /newbot → follow prompts → receive token
	3	Start a chat with your new bot
	4	Open in browser: https://api.telegram.org/bot/getUpdates → Find "from":{"id":123456789} → that number is your chat ID
In n8n → Credentials → Telegram API → paste token In the Telegram node, set Chat ID to your number.

4. Google Gemini API Key (for AI Summaries)
	1	Visit Google AI Studio
	2	Sign in → Create API key → copy it
	3	In n8n → Credentials → Google Gemini (PaLM) API → paste key (Swap with OpenAI, Anthropic, etc. if preferred)

5. WEBHOOK_URL Environment Variable
This tells the workflow where this same n8n is reachable.
If using n8n.cloud: Settings → Variables → Add
WEBHOOK_URL = https://your-subdomain.n8n.cloud/
If self-hosted (same VPS): Edit .env in your n8n folder:
WEBHOOK_URL=https://n8n.yourdomain.com/
Then restart n8n.

Final Configuration Steps Inside the Workflow
	1	Import the provided JSON.
	2	Open the Docker Path node → change: "docker_path": "/root"
	3	 → to your actual docker-compose.yml directory (e.g., "/home/ubuntu/n8n"). Optional: set worker_command like "--scale n8n-worker=3".
	4	In the Telegram node, update Chat ID to yours.
	5	Assign credentials:
	◦	Both SSH nodes → your SSH key credential
	◦	Telegram node → your bot credential
	◦	Gemini node → your API key credential
	6	Save and activate.

What Happens After Setup?
	•	Hourly: Silent check for new image digest
	•	Update found → AI summary → Telegram alert
	•	You tap Approve → background update runs
	•	n8n restarts fresh → you’re done
	•	Check update.log only if curious

Docker 
Now every new version installs silently within minutes.

In Short
Set up once on your own n8n → it watches Docker Hub → summarizes changes with AI → asks you on Telegram → updates itself in the background → no terminal, no downtime, no hassle — forever.

