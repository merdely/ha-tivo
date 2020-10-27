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

This is just the relevant parts of my configuration.yaml.  There is obviously more.

<pre>
tts:
  - platform: google_translate

dialogflow:

google_assistant: !include google_assistant.yaml
shell_command: !include <a href="homeassistant/shell_commands.yaml">shell_commands.yaml</a>
intent_script: !include <a href="homeassistant/intent_scripts.yaml">intent_scripts.yaml</a>
</pre>

### My Tivo Scripts

* [tivo_tivobutton](tivo-scripts/tivo_tivobutton): Press the Tivo Button on a Tivo
* [tivo_livetv](tivo-scripts/tivo_livetv): Press the Live TV Button on a Tivo
* [tivo_setch](tivo-scripts/tivo_setch): Change the channel on a Tivo
* [tivo_play](tivo-scripts/tivo_play): Press Play on a Tivo
* [tivo_pause](tivo-scripts/tivo_pause): Press Pause on a Tivo
* [tivo_record](tivo-scripts/tivo_record): Record the currently active show with default settings on a Tivo
* [tivo_enter](tivo-scripts/tivo_enter): Send the Enter button to a Tivo

## Set Up Instructions

1. Assuming Home Assistant is installed in /srv/docker/homeassistant (you don't have to install here, substitute your Home Assistant install path for mine)
1. Create a 'bin' directory in /srv/docker/homeassistant and copy the tivo-scripts to that directory
1. Modify configuration.yaml to include the lines from my configuration file above.
1. Copy the [shell_commands.yaml](homeassistant/shell_commands.yaml) and [intent_scripts.yaml](homeassistant/intent_scripts.yaml) to the same directory as your configuration.yaml file.
1. Restart Home Assistant.
1. Log into [Google DialogFlow](https://dialogflow.cloud.google.com/)
1. Configure DialogFlow following the [Home Assistant DialogFlow documentation](https://www.home-assistant.io/integrations/dialogflow/)
1. Create @channel Entity:
   1. Leave 'Define synonyms' checked; check 'Regexp entity' and 'Allow automated expansion'
   1. Add '(channel )?\d{3}' for a row.
   1. Add '(channel )?\d \d \d' as another way of inputing the same thing with spaces in another row.
   1. Add '(channel )?(five|six|seven|eight|nine) (oh|zero|one|two|three|four|five|six|seven|eight|nine) (oh|zero|one|two|three|four|five|six|seven|eight|nine)' in case Google interprets the channel numbers as words for a third row.
   1. Add specific channels in another row with '(CW|NBC|Fox|my TV|ABC|CBS|USA|TNT|TBS|FX|WGN|ESPN|masn|MASN|NFL Network|Tennis Channel|CNN|CNBC|Weather Channel|Discovery Channel|National Geographic|Animal Planet|TLC|Lifetime|QVC|Food Network|HGTV|TruTV|Syfy|Sci-Fi|A and E|a n e|Bravo|Ovation|BBC America|Comedy Central|pop|freeform|MTV|VH1|AMC|IFC|TV Land|Nick|Nickelodeon|Cartoon Network|Disney Channel|HBO)'
1. Create @tv_device Entity:
   1. Leave 'Define synonyms' checked and nothing else.
   1. Enter names for your Tivos as the reference value and different words for how you might refer to it as synonyms.  For example, my 'tivo-familyroom' can sometimes be referred to as 'Family Room Tivo', 'Family Room TV', and 'God Damn Tivo'.  So I added those synonyms.  Note that 'tivo-familyroom' is the DNS name for my Tivo for my home network.
   1. Add synonyms for all of your Tivos.
1. Either set up DNS or /etc/hosts file entries on your Home Assistant server for all of your tivos so that Home Assistant can access the tivos using the 'reference value' from the previous @tv_device Entity in the scripts.
1. Create Intents for each action you want DialogFlow to perform.  I have:
   - ChangeTivoChannel:
     1. Add Training Phrases:
        - 'to channel 504 on God Damn Tivo'
        - 'Change God Damn Tivo to channel 504'
     1. Select "channel 504"; a popup should let you assign it to an Entity, choose @channel
     1. Select "God Damn Tivo"; a popup should let you assign it to an Entity, choose @tv_device
     1. Enter 'ChangeTivoChannel' as Action Name and add entries for @channel, @channel, $channel with a prompt of "To which Channel?" and @tv_device, @tv_device, $tv_device with a prompt of "Which Tivo?"
     1. Under Responses, enter "Channel changed", check "Set this intent as end of conversation", select Google Assistant and check 'Use responses from the DEFAULT tab as the first responses'.
     1. Under Fulfillment, check 'Enter webhook call for this intent'
     1. Click Save
