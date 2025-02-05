---
layout: post
title: "Home Assistant - Automate and centralize your smart home"
date: 2025-02-05 10:00:00 0000
categories: services home-automation
tags: homelab home-automation automation homeassistant home-assistant

---

[Home Assistant - Home Automation](https://www.home-assistant.io/) Open source home automation that puts local control and privacy first. Powered by a worldwide community of tinkerers and DIY enthusiasts. Perfect to run on a Raspberry Pi or a local server. 

### SSO
Home Assistant does not support SSO like OAuth or anything so we are using Header Authentication via [https://github.com/BeryJu/hass-auth-header](https://github.com/BeryJu/hass-auth-header) and Proxy Provider in Authentik

config.yml
```yaml
auth_header:
    username_header: X-authentik-username
```

### Resetting Password
`auth reset --username <username> --password <newpassword>`

Via Console (not SSH addon)

If this doesn't work you can manually update the `/config/.storage/auth_provider.homeassistant` file using this python3 script

`pip install bcrypt`
```python
#!/usr/bin/env python3
import bcrypt
import base64
import getpass

password = getpass.getpass('Password: ')
hashed = bcrypt.hashpw(password.encode(), bcrypt.gensalt(rounds=12))
hashed = base64.b64encode(hashed)
print(hashed)
```

remove the b prefix and the ' suffix

### HA Cloudflare Zero Trust mTLS Certificate for Android App
![ha1.png](/assets/img/old/ha1.png)

![ha2.png](/assets/img/old/ha2.png)

### Addons
#### [Cloudflared](https://github.com/cloudflare/cloudflared)
Cloudflared connects your Home Assistant Instance via a secure tunnel to a domain or subdomain at Cloudflare. Doing that, you can expose your Home Assistant to the Internet without opening ports in your router. Additionally, you can utilize Cloudflare Teams, their Zero Trust platform to further secure your Home Assistant connection

#### [ESPHome](https://esphome.io/)
ESPHome is a system to control your microcontrollers by simple yet powerful configuration files and control them remotely through Home Automation systems

#### [Google Drive Backup](https://github.com/sabeechen/hassio-google-drive-backup)
Automatically manage backups between Home Assistant and Google Drive.

#### [Mosquitto Broker (MQTT)](https://github.com/home-assistant/addons/tree/master/mosquitto)
An open-source (EPL/EDL licensed) message broker that implements the MQTT protocol. Mosquitto is lightweight and is suitable for use on all devices from low power single board computers to full servers.

#### [Node-RED](https://github.com/hassio-addons/addon-node-red)
Node-RED is a programming tool for wiring together hardware devices, APIs and online services in new and interesting ways.

It provides a browser-based editor that makes it easy to wire together flows using the wide range of nodes in the palette that can be deployed to its runtime in a single click.

#### [Zigbee2MQTT](https://github.com/zigbee2mqtt/hassio-zigbee2mqtt/tree/master/zigbee2mqtt)
Allows you to use your Zigbee devices without the vendors bridge or gateway.

It bridges events and allows you to control your Zigbee devices via MQTT. In this way you can integrate your Zigbee devices with whatever smart home infrastructure you are using.