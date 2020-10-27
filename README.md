# Home Assistant / Tivo Integration

This project is to help with using Home Assistant to manage Tivo devices

## Requirements

1. Home Assistant
1. Tivos
1. Enabling the network remote access on the Tivos
1. Google Account (for DialogFlow and Actions)

## My Set Up

I have Home Assistant running in a Docker container on a Raspberry Pi.  I followed [the documentation from the Home Assistant website](https://www.home-assistant.io/docs/installation/docker/) and am using the 'homeassistant/aarch64-homeassistant:latest' image.

### My systemd unit file
```
[Unit]
Description=HomeAsssistant Docker Container
Documentation=https://www.home-assistant.io/docs/installation/docker/
After=network.target docker.service
Requires=docker.service

[Service]
StandardOutput=append:/var/log/docker-homeassistant.log
StandardError=inherit
TimeoutStartSec=300
RestartSec=10
Restart=always
ExecStartPre=-/usr/bin/docker rm -f homeassistant
# This takes too long, so it's commented out and I do it manually every so often
# ExecStartPre=-/usr/bin/docker pull homeassistant/aarch64-homeassistant:latest
ExecStart=/usr/bin/docker run --rm --name="homeassistant" --net=host \
  -v /srv/docker/homeassistant:/config -v /etc/localtime:/etc/localtime:ro \
  homeassistant/aarch64-homeassistant:latest

[Install]
WantedBy=multi-user.target
```

### My Home Assistant Configuration file
```
...
tts:
  - platform: google_translate

dialogflow:

google_assistant: !include google_assistant.yaml
...
shell_command: !include [shell_commands.yaml](https://github.com/merdely/ha-tivo/blob/main/homeassistant/shell_commands.yaml)
intent_script: !include [intent_scripts.yaml](https://github.com/merdely/ha-tivo/blob/main/homeassistant/intent_scripts.yaml)
...
```
