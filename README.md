### Personalized fork of [NetData](https://github.com/netdata/netdata/)
_I wasnt able to get alert and notifiaction for agent connection and reconnect. So i had to change the repo code itself and use manually built netdata_

Context: [Github Discussion](https://github.com/netdata/netdata/discussions/21542) | [r/netdata](https://www.reddit.com/r/netdata/comments/1qatode)

### Intro
- Whenever a node connects or disconnects to parent it executes the bash script `/etc/netdata/custom-child-disconnect.sh` with node hostname, node ip, and "CONNECTED" or disconnection_reason as arguments.

### Build
- Uninstall existing nedata installation: `./kickstart.sh --uninstall`
- git clone the repo and cd into it
- `git submodule update --init --recursive`
- Update and install required packages:
  - sudo apt update
  - sudo apt install -y build-essential cmake ninja-build git pkg-config
  - sudo apt install -y uuid-dev libuv1-dev zlib1g-dev z
  - sudo apt install -y liblz4-dev libssl-dev libjson-c-dev libyaml-dev libcurl4-openssl-dev libmnl-dev libcap-dev
  - sudo apt install -y protobuf-compiler libprotobuf-dev
  - sudo apt install -y golang-go
  - sudo apt install flex bison
- Fianlly build and install with: sudo ./netdata-installer.sh --dont-wait

### Usage
- sudo systemctl unmask netdata
- sudo systemctl daemon-reload
- sudo systemctl enable --now netdata

### Node Connect/Disconnect Notification Script
- /etc/netdata/custom-child-disconnect.sh
```
#!/bin/bash

# Configuration
DISCORD_WEBHOOK_URL="https://canary.discord.com/api/webhooks/xxx/xxx"
LOG_FILE="/var/log/netdata/child_disconnects.log"

HOSTNAME="$1"
IP="$2"
STATUS="$3" 
TIMESTAMP=$(date "+%Y-%m-%d %H:%M:%S")

# 1. Local Logging
echo "[$TIMESTAMP] EVENT - Host: $HOSTNAME | IP: $IP | Status: $STATUS" >> "$LOG_FILE"

# 2. Logic for Discord appearance
if [ "$STATUS" == "CONNECTED" ]; then
    TITLE="✅ Child Node Reconnected"
    COLOR=3066993  # Green
    DESCRIPTION=""
else
    TITLE="🚨 Child Node Disconnected"
    COLOR=15158528 # Red
    DESCRIPTION="**Reason:** $STATUS"
fi

# 3. Discord Notification
if [ -z "$DISCORD_WEBHOOK_URL" ]; then exit 0; fi

PAYLOAD=$(cat <<EOF
{
  "embeds": [{
    "title": "$TITLE",
    "description": "$DESCRIPTION",
    "color": $COLOR,
    "fields": [
      { "name": "Hostname", "value": "$HOSTNAME", "inline": true },
      { "name": "IP Address", "value": "$IP", "inline": true },
      { "name": "Time", "value": "$TIMESTAMP", "inline": false }
    ],
    "footer": { "text": "Netdata Parent: $(hostname)" }
  }]
}
EOF
)

curl -s -X POST -H "Content-Type: application/json" -d "$PAYLOAD" "$DISCORD_WEBHOOK_URL" > /dev/null
```
- sudo chmod +x /etc/netdata/custom-child-disconnect.sh

---

That's all!

---
