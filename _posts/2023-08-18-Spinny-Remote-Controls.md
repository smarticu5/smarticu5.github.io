---
title: "Spinny Remote Controls"
- tags:
  - homeassistant
  - automation
---

This is hopefully the first of multiple posts details home automations I've written for [home-assistant.io]() which are interesting enough to talk about. The posts are probably going to be a reminder for me about what I wrote and how, rather than a tutorial. 

## Hardware

I've got a few Sonos speakers scattered throughout the house. These mostly just do their thing without needing any changes, but sometimes I want to quickly play/pause/change volume without needing to get my phone. I picked up a couple of IKEA rotary Symfonisk remotes, which have since been sadly discontinued and I can't even find a link to them on the IKEA website. If anyone's interested, they have [this entry on Zigbee2MQTT's Wiki](https://www.zigbee2mqtt.io/devices/E1744.html). 

The devices expose a few actions, and a "brightness" setting. If you're using them with the official IKEA hub there's no need to even set up anything tricky, the marketing suggests that everything just works in-app. That doesn't work for me, because who would use vendor hubs when you can spend too long half-assing a custom solution.

## Software

Adding the spinny remote to Home Assistant was dead easy. So was making the play-pause functionality work:

```yaml
trigger:
  - platform: device
    domain: mqtt
    device_id: abcd1234
    type: action
    subtype: play_pause
action:
  - choose:
      - conditions:
          - condition: or
            conditions:
              - condition: device
                domain: media_player
                entity_id: media_player.sonos_office
                type: is_paused
              - condition: device
                domain: media_player
                entity_id: media_player.sonos_office
                type: is_playing
        sequence:
          - service: media_player.media_play_pause
            data: {}
            target:
              entity_id: media_player.sonos_office
```

The automation above basically says "If the Sonos in the office is playing or paused, toggle the play/pause state". I've added another condition to say if the state is stopped, pick a random playlist and start that, using the play_media calls. 

What was a bit trickier was getting the volume control right. I would adjust the volume on my phone, then mess with the remote and have the volume suddenly jump. This was because the remote has no idea of what the volume actually is, only what its own "brightness" field is set to. 

The workaround for this was to check if the remote was being rotated left or right. If it's left, the volume is going down, so set the volume to the lower of the remote's setting or the speaker's current volume. If it's right, the opposite. This means you might need to turn the remote a bit more, but you never end up spinning the remote to turn the volume down and have it suddenly launch way higher. 

```yaml
service: media_player.volume_set
data:
  volume_level: >
    {% raw %}
    {% set office_remote_volume =
    float(state_attr('sensor.spinny_remote_action', 'brightness')/255) |
    round(2)%}  {%- if trigger.id == "volume_up" -%}
    {{float(max(office_remote_volume,state_attr("media_player.sonos_office","volume_level")))}}
    {%- elif trigger.id == "volume_down" -%}
    {{float(min(office_remote_volume,state_attr("media_player.sonos_office",
    "volume_level")))}}  {%- endif -%}
    {% raw %}
target:
  entity_id: media_player.sonos_office
```

Incidentally, posting this blog also taught me that Jekyll tries to render Home Assistant templates, and that makes GitHub Actions fail. Sad times. 