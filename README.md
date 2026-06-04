# Argo Discord Bot

Brief setup notes for running Argo as a `systemd` service on a Linux server.

## Server setup

Steps were something like:

```bash
cd /~
git clone <repo-url> argo-discord-bot
cd argo-discord-bot
python3 -m venv /~/venv
source /~/venv/bin/activate
pip install -r requirements.txt
```

Create a `.env` file in the repo directory with the required Discord/Poe settings. The `.env` customization is straightforward and is not covered here:

```ini
DISCORD_TOKEN=
POE_API_KEY=

POE_MODEL=claude-sonnet-4.6
POE_VISION_MODEL=claude-sonnet-4.6

POE_MAX_TOKENS=350
POE_TEMPERATURE=0.65

SUMMARY_MAX_TOKENS=120
SUMMARY_MODEL=claude-sonnet-4.6

PROACTIVE_CHANNEL_ID=1495930794735308861
PROACTIVE_MIN_HOURS_BETWEEN=12
PROACTIVE_SKIP_IF_ACTIVE_WITHIN_MINUTES=60
PROACTIVE_QUIET_HOURS_START=5
PROACTIVE_QUIET_HOURS_END=12
PROACTIVE_LOOP_MIN_MINUTES=60
PROACTIVE_LOOP_MAX_MINUTES=120
PROACTIVE_MESSAGE_MAX_TOKENS=70

USER_TIMEZONE=America/Halifax
```

## systemd service

```bash
sudo nano /etc/systemd/system/argo-discord-bot.service
```

```ini
[Unit]
Description=Argo Discord Bot
After=network.target

[Service]
Type=simple
WorkingDirectory=/root/argo-discord-bot
ExecStart=/root/venv/bin/python -u /root/argo-discord-bot/bot.py
Restart=always
RestartSec=5
User=root

[Install]
WantedBy=multi-user.target
```

If the repo or venv lives somewhere else on a new server, update these paths:

```text
WorkingDirectory=/root/argo-discord-bot
ExecStart=/root/venv/bin/python -u /root/argo-discord-bot/bot.py
```

## Enable and start

```bash
sudo systemctl daemon-reload
sudo systemctl enable argo-discord-bot
sudo systemctl start argo-discord-bot
```

Check status and logs:

```bash
sudo systemctl status argo-discord-bot
sudo journalctl -u argo-discord-bot -f
```

Restart after code or `.env` changes:

```bash
sudo systemctl restart argo-discord-bot
```

## Notes for moving servers

When moving Argo to another server, copy over any runtime files you want to preserve, such as:

```text
.env
memory.json
conversation_summaries.json
proactive_state.json
recent_history.json
```

Make sure the Linux user running the service has read/write access to the repo directory so Argo can update its memory and summary files.
