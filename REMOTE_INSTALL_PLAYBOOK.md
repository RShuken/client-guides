# OpenClaw Remote Installation Playbook

A guide for remotely setting up OpenClaw on client machines via SSH. Based on real installs with lessons learned.

---

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Establishing SSH Access](#establishing-ssh-access)
3. [Post-Install Configuration](#post-install-configuration)
4. [Security Hardening](#security-hardening)
5. [Client Documentation](#client-documentation)
6. [Troubleshooting](#troubleshooting)
7. [Client Install Logs](#client-install-logs)

---

## Prerequisites

### On Your Machine (Ryan's)
- `sshpass` installed: `brew install hudochenkov/sshpass/sshpass`
- Tailscale installed and running (for network connectivity)
- Access to client guide templates in `/Users/home/clawd-main/clients/`

### On Client Machine
- macOS (tested on 12.7.6 Monterey and newer)
- Homebrew installed
- Node.js via nvm (or system Node 22+)
- OpenClaw already installed via `npm install -g openclaw`
- Telegram bot created via BotFather

---

## Establishing SSH Access

### Option A: Tailscale (Recommended for Ongoing Access)

**Client runs:**
```bash
# Install Tailscale
brew install tailscale

# Start daemon (if CLI install)
sudo tailscaled --state=/var/lib/tailscale/tailscaled.state > /dev/null 2>&1 &

# Or if GUI app: just open Tailscale from Applications

# Connect with SSH enabled
tailscale up --ssh

# Get their Tailscale IP
tailscale ip -4
```

**If separate Tailscale accounts:**
1. Client goes to https://login.tailscale.com/admin/machines
2. Click ⋮ on their machine → Share
3. Enter your Tailscale email

**If Tailscale SSH ACL blocks access:**
```bash
# Disable Tailscale SSH, use regular macOS SSH instead
tailscale up --ssh=false
sudo systemsetup -setremotelogin on
sudo launchctl load -w /System/Library/LaunchDaemons/ssh.plist
```

**Connect:**
```bash
sshpass -p 'PASSWORD' ssh -o StrictHostKeyChecking=no -o PubkeyAuthentication=no -o PreferredAuthentications=keyboard-interactive,password USERNAME@TAILSCALE_IP
```

### Option B: ngrok (One-Time Access, No Account Sharing)

**Client runs:**
```bash
brew install ngrok
# Sign up at ngrok.com, get authtoken
ngrok config add-authtoken <token>
sudo systemsetup -setremotelogin on
ngrok tcp 22
```

Gives you a hostname:port like `0.tcp.ngrok.io:12345`

**Connect:**
```bash
sshpass -p 'PASSWORD' ssh -o StrictHostKeyChecking=no USERNAME@0.tcp.ngrok.io -p 12345
```

---

## Post-Install Configuration

### 1. Check Current State
```bash
# Set PATH for nvm-installed OpenClaw
export PATH="/Users/USERNAME/.nvm/versions/node/VERSION/bin:$PATH"

# Check status
openclaw status
openclaw doctor --fix
```

### 2. Update Config (Models, Fallbacks, Agent Settings)

Create updated config and push via scp:
```bash
sshpass -p 'PASSWORD' scp -o StrictHostKeyChecking=no -o PubkeyAuthentication=no -o PreferredAuthentications=password /tmp/client-config.json USERNAME@IP:~/.openclaw/openclaw.json
```

**Standard config changes:**
```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-opus-4-5",
        "fallbacks": ["openai-codex/gpt-5.2"]
      },
      "memorySearch": { "provider": "local" },
      "contextPruning": { "mode": "cache-ttl", "ttl": "1h" },
      "heartbeat": { "every": "1h" },
      "maxConcurrent": 2,
      "subagents": { "maxConcurrent": 4 }
    }
  },
  "tools": {
    "web": {
      "search": { "enabled": true },
      "fetch": { "enabled": true }
    }
  }
}
```

### 3. Restart Gateway
```bash
openclaw gateway stop
openclaw gateway start
# or
openclaw gateway install  # if service not loaded
```

### 4. Push Workspace Files

**NIGHTSHIFT.md** — Autonomous task queue
**HEARTBEAT.md** — Periodic check configuration  
**AGENTS.md** — Workspace behavior instructions

```bash
sshpass -p 'PASSWORD' scp -o PreferredAuthentications=password FILE USERNAME@IP:~/.openclaw/workspace/
```

### 5. Create Cron Jobs
```bash
openclaw cron add --name "nightshift" --cron "0 2 * * *" --tz "America/Denver" --session isolated --message "NIGHTSHIFT: Read NIGHTSHIFT.md and work through the task queue autonomously."
```

---

## Security Hardening

### Run Security Audit
```bash
openclaw security audit --deep
```

### Common Fixes
```bash
# Fix state directory permissions
chmod 700 ~/.openclaw

# Fix log file permissions
chmod 600 ~/.openclaw/logs/*.log
```

### Expected Warnings (Safe to Ignore)
- `gateway.trusted_proxies_missing` — Only matters with reverse proxy, not for local-only gateway

---

## Client Documentation

### Create Explainer Website
1. Copy template from `/Users/home/clawd-main/clients/ian-ferguson/guide.html`
2. Customize for client:
   - Assistant name, owner name
   - Telegram bot username
   - Machine type (iMac vs Mac Mini)
   - macOS version
   - Available integrations
   - Model fallback chain
3. Save to `/Users/home/clawd-main/clients/CLIENT_NAME/guide.html`
4. Push to GitHub:
```bash
cd /Users/home/clawd-main/clients
git add CLIENT_NAME/
git commit -m "Add CLIENT_NAME client guide"
git push origin main
```

**Live URL:** `https://rshuken.github.io/client-guides/CLIENT_NAME/guide.html`

---

## Troubleshooting

### SSH Issues

| Problem | Solution |
|---------|----------|
| `Permission denied` after password | Add `-o PubkeyAuthentication=no -o PreferredAuthentications=keyboard-interactive,password` |
| `Too many authentication failures` | SSH is trying keys first; use options above |
| `Host key changed` warning | `ssh-keygen -R IP_ADDRESS` |
| `sudo` commands get "Stopped" | Run `sudo echo "ok"` first to cache password, then background the command |
| Tailscale SSH `policy does not permit` | Run `tailscale up --ssh=false` and use regular SSH |

### OpenClaw Issues

| Problem | Solution |
|---------|----------|
| `openclaw: command not found` | PATH not set; use full path or `export PATH="~/.nvm/versions/node/VERSION/bin:$PATH"` |
| Gateway won't start | Run `openclaw gateway install` first |
| iMessage not working | Requires macOS 14+; use BlueBubbles on older versions |
| `imsg` won't install | Same as above; `imsg` CLI needs Sonoma |

### Integration Issues

| Problem | Solution |
|---------|----------|
| BlueBubbles won't install | Will install but may show Gatekeeper warning; allow in System Preferences |
| BlueBubbles can't read Messages | Grant Full Disk Access in System Preferences → Privacy |

---

## Client Install Logs

### Rosey (Eric) — 2026-02-02

**Machine:** Roseys-iMac, macOS 12.7.6 Monterey  
**Node:** v24.13.0 via nvm  
**OpenClaw:** Already installed  
**Telegram Bot:** @roseyTheClawd_bot

#### Goals
- Configure models (Claude Opus 4.5 + GPT-5.2 fallback)
- Match Telegram policies to Ryan's setup
- Enable agent settings (heartbeat, memory search, context pruning)
- Set up nightshift autonomous work
- Fix iMessage (if possible)
- Security hardening
- Create explainer website

#### What We Tried & Results

| Task | Attempt | Result | Fix/Workaround |
|------|---------|--------|----------------|
| SSH via Tailscale SSH | `tailscale up --ssh` | ACL blocked: "policy does not permit" | Disabled Tailscale SSH (`--ssh=false`), used regular macOS SSH |
| Tailscale node sharing | Separate accounts | Initially couldn't reach | Client shared node via admin console |
| SSH auth | Password auth | "Too many auth failures" | Added `-o PubkeyAuthentication=no -o PreferredAuthentications=password` |
| iMessage via `imsg` | `brew install steipete/tap/imsg` | Failed: "requires macOS Sonoma" | `imsg` CLI needs macOS 14+; documented BlueBubbles as alternative |
| BlueBubbles install | `brew install --cask bluebubbles` | ✅ Installed (with deprecation warning) | Deferred setup to later; added to TODOs |
| Config update | SCP new config | ✅ Success | — |
| Gateway restart | `openclaw gateway stop/start` | LaunchAgent needed reinstall | `openclaw gateway install` |
| Security audit | `openclaw security audit --deep` | 3 warnings | Fixed permissions: `chmod 700 ~/.openclaw`, `chmod 600 logs/*.log` |
| Nightshift cron | `openclaw cron add` | ✅ Created | Runs at 2 AM MST daily |

#### Files Pushed
- `~/.openclaw/openclaw.json` — Full config
- `~/.openclaw/workspace/AGENTS.md`
- `~/.openclaw/workspace/NIGHTSHIFT.md`
- `~/.openclaw/workspace/HEARTBEAT.md`

#### Pending TODOs (for Eric)
- [ ] BlueBubbles setup (requires GUI interaction)
- [ ] GOG (Google Workspace) integration
- [ ] Brave Search API key
- [ ] iCal integration

#### Explainer Website
- Created: `/Users/home/clawd-main/clients/rosey/guide.html`
- Live: https://rshuken.github.io/client-guides/rosey/guide.html

---

### Vishnu (Ian Ferguson) — Prior Install

**Machine:** Mac Mini  
**Telegram Bot:** @Vishnuexu_bot

#### Configuration
- Primary: Claude Opus 4.5
- Fallbacks: Claude Sonnet 4 → Gemini 3 Pro → GPT-4o (4-model chain)
- Tier 2 API access ($40 deposit)
- Multi-provider failover configured

#### Features Enabled
- Web search (Brave API)
- Web fetch
- Browser control (managed browser)
- Heartbeat monitoring
- Nightshift autonomous work

#### Explainer Website
- Live: https://rshuken.github.io/client-guides/ian-ferguson/guide.html
- Includes: Rate limit explainer, token visualization, troubleshooting

---

## Quick Reference Commands

### SSH Connection Template
```bash
sshpass -p 'PASSWORD' ssh -o StrictHostKeyChecking=no -o PubkeyAuthentication=no -o PreferredAuthentications=keyboard-interactive,password USERNAME@IP 'export PATH="~/.nvm/versions/node/VERSION/bin:$PATH"; COMMAND'
```

### SCP File Transfer
```bash
sshpass -p 'PASSWORD' scp -o StrictHostKeyChecking=no -o PubkeyAuthentication=no -o PreferredAuthentications=password LOCAL_FILE USERNAME@IP:REMOTE_PATH
```

### Full Setup Sequence
```bash
# 1. Check status
openclaw status

# 2. Run doctor
openclaw doctor --fix

# 3. Security audit
openclaw security audit --deep

# 4. Fix permissions
chmod 700 ~/.openclaw
chmod 600 ~/.openclaw/logs/*.log

# 5. Create nightshift cron
openclaw cron add --name "nightshift" --cron "0 2 * * *" --tz "America/Denver" --session isolated --message "NIGHTSHIFT: Read NIGHTSHIFT.md..."

# 6. Verify
openclaw cron list
openclaw status
```

---

## Notes

- Always save client passwords securely (they'll need them for future access)
- Document macOS version — affects available integrations
- Test Telegram bot connection before leaving
- Push workspace files (AGENTS.md, NIGHTSHIFT.md, HEARTBEAT.md)
- Create and deploy explainer website
- Add client to `/Users/home/clawd-main/memory/` for future reference
