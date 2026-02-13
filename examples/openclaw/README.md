# OpenClaw on SkyPilot

Deploy [OpenClaw](https://github.com/openclaw/openclaw) - a personal AI assistant for WhatsApp, Telegram, Slack, Discord, and more - on a cloud VM via SkyPilot.

This gives you an isolated sandbox environment separate from your local machine, with the built-in WebChat interface accessible via SSH tunnel. All state is persisted to an S3 bucket, so you can tear down and relaunch the cluster without losing configuration, credentials, or conversation history.

## Prerequisites

- [SkyPilot](https://docs.skypilot.co/) installed and cloud credentials configured (`sky check`)
- An [Anthropic API key](https://console.anthropic.com/)
- AWS credentials with S3 access (used for persistent storage)

## Quickstart

1. Create an S3 bucket for persistent state (one-time):

```bash
aws s3 mb s3://openclaw-state
```

2. Launch the gateway:

```bash
sky launch -c openclaw openclaw.yaml --env ANTHROPIC_API_KEY
```

3. Open an SSH tunnel and access WebChat:

```bash
ssh -L 18789:localhost:18789 openclaw
```

Then open http://localhost:18789 in your browser. On first launch, the gateway auth token is printed in the setup logs - enter it in the WebChat Settings panel to connect. On subsequent launches, the token is reused from the persisted config; retrieve it with `grep token ~/.openclaw/openclaw.json`.

<p align="center">
  <img src="https://i.imgur.com/gdF6JDX.png" alt="OpenClaw Gateway Dashboard" width="600">
</p>

<p align="center">
  <img src="https://i.imgur.com/YHCnb7C.png" alt="OpenClaw WebChat interface" width="600">
</p>


## Data persistence

All OpenClaw state is stored in an S3 bucket (default: `openclaw-state`) mounted at `~/.openclaw` on the VM using `MOUNT_CACHED` mode. This uses rclone with local disk caching, which supports full POSIX write operations (including the append writes OpenClaw uses for session transcripts) and asynchronously syncs changes back to S3.

Persisted data includes:

| Data | Path | Why it matters |
|------|------|----------------|
| Configuration | `openclaw.json` | Model selection, channel settings, gateway auth token |
| Cron jobs | `cron/jobs.json` | Scheduled task definitions |
| Canvas UI | `canvas/index.html` | Mobile node Canvas interface |
| Update check | `update-check.json` | Tracks last version check |

Additional paths appear as you configure more features (e.g. `credentials/whatsapp/` for WhatsApp sessions, `workspace/` for agent files).

This means you can `sky down openclaw` and later `sky launch` again - the gateway picks up right where it left off with all channels connected and history intact.

To use a custom bucket name:

```bash
aws s3 mb s3://my-openclaw-bucket
sky launch -c openclaw openclaw.yaml \
  --env ANTHROPIC_API_KEY \
  --env OPENCLAW_BUCKET=my-openclaw-bucket
```

## Next steps

Once the gateway is running and you can access WebChat, here are the key ways to make your deployment more useful.

### Connect messaging channels

SSH into the cluster and run the onboarding wizard, or configure channels individually:

```bash
ssh openclaw

# Interactive wizard - walks through all available channels
openclaw onboard

# Or connect channels directly:
openclaw channels login --channel whatsapp   # scan QR code to link
openclaw channels login --channel telegram   # requires a BotFather token
openclaw channels login --channel discord
```

### Lock down access

The default configuration uses `pairing` mode - unknown senders receive a code and are ignored until you approve them. Review and approve pending requests:

```bash
openclaw pairing list whatsapp
openclaw pairing approve whatsapp <CODE>
```

For tighter control, edit `~/.openclaw/openclaw.json` on the VM (changes hot-reload automatically):

```json5
{
  "channels": {
    "whatsapp": {
      "dmPolicy": "allowlist",
      "allowFrom": ["+15551234567"],
      "groups": { "*": { "requireMention": true } }
    }
  }
}
```

Run `openclaw security audit` to check for common misconfigurations.

### Customize the agent persona

OpenClaw reads context files from the workspace directory on the first turn of each session. SSH in and edit these to shape behavior:

```bash
ssh openclaw
cd ~/.openclaw/workspace  # or wherever agents.defaults.workspace points

# Key files:
#   AGENTS.md   - operating instructions and memory
#   SOUL.md     - persona, tone, and boundaries
#   TOOLS.md    - tool usage notes
#   IDENTITY.md - agent name and emoji
#   USER.md     - user profile and preferred form of address
```

### Set up cron jobs

Schedule recurring tasks that run inside the gateway:

```bash
ssh openclaw
openclaw cron add --name "Morning brief" \
  --cron "0 7 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize my calendar and unread messages"
```

### Enable sandboxing

For additional isolation, run agent tool execution inside Docker containers on the VM:

```json5
// In ~/.openclaw/openclaw.json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "all",
        "scope": "session"
      }
    }
  }
}
```

This requires Docker to be installed on the VM. You can add `sudo apt-get install -y docker.io` to the `setup` block in `openclaw.yaml`.

For full configuration reference, see [docs.openclaw.ai](https://docs.openclaw.ai/).

## Teardown

```bash
# Stop the cluster - all state remains in S3
sky down openclaw

# Relaunch later - resumes with existing state
sky launch -c openclaw openclaw.yaml --env ANTHROPIC_API_KEY
```

To permanently delete all persisted data, remove the S3 bucket:

```bash
aws s3 rb s3://openclaw-state --force
```
