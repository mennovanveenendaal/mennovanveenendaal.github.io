---
title: Avoiding Future VPN Disruptions; IP Monitoring with Python
layout: post
categories: scripting, python
image:
  path: /assets/vpn.png
  alt: VPN
---
During my vacation, I ran into a frustrating issue: my VPN connection to my home server stopped working after an ISP outage. This was because my home’s WAN IP address had changed, which meant the VPN client couldn’t reconnect to my server anymore. Since I rely heavily on my home server for automation and other services, this was an issue I didn’t want to experience again. So, after some thought, I decided to write a script that would notify me whenever my WAN IP address changes.


# Lost Connection: The Problem
In our household, I’ve automated nearly everything, from lights and appliances to notifications and security. To manage all of these services, I run several servers in a hypervisor environment. Some servers handle my home automation apps, while others are used for testing purposes, like experimenting with networking, firewalls, and forensics. All of these servers are isolated from the internet and are only accessible via my local home network.

To ensure I could monitor these systems remotely, I set up a VPN server that would allow me to connect to my home network securely while I was away. The VPN client on my phone connects via the WAN IP address, routing my connection back home.

This setup had been working perfectly, until this summer. On the second day of my vacation, I realized my phone couldn’t connect to the internet when the VPN was enabled. Disabling the VPN restored connectivity, but the VPN logs showed a handshake failure whenever the client attempted to connect.

A quick Google search confirmed that my ISP had an ongoing outage. After several hours, internet service was restored, but my VPN still wouldn’t connect. I suspected that my WAN IP address had changed, but since I was far from home and had no way to access my network, I couldn’t verify this. I had to wait until I returned from vacation to check.

Sure enough, when I got home, I found that my WAN IP address had changed, which explained why the VPN failed to connect. To prevent this from happening again, I decided to create a script that would notify me whenever my WAN IP changed. I could have used an external service like DuckDNS to manage dynamic IP addresses, but I wanted full control over the entire process. Plus, I wanted to try to script in Python, and this looked like a good purpose.

# IP Monitoring: The Solution
First, I set up a headless Raspberry Pi to run the script. I found a simple Python [script](https://melanee-melanee.medium.com/monitor-your-public-ip-address-using-python-7956d18be9ec) that checked the WAN IP by sending a request to `https://api.ipify.org`. The script uses the `requests` module to fetch the current WAN IP:
`WanIP.py`
```python
import requests  

def get_wanIP():
	response = requests.get('https://api.ipify.org').text
	return response
```
With this working, I wrote a script that compared the current WAN IP with the one stored in a SQL database. If the IP changed, the script would log the new IP address and send me a notification via Telegram. Here’s the Python code that checks the current IP:
`IP_Processing.py`
```python
from IPAddress import IPAddress
import requests
from IPDatabase import *

def get_currentIPAddress() -> str:
	response = requests.get('https://api.ipify.org').text
	return response

def get_latestIPAddressFromDatabase() -> str:
	IPAddress = get_latestIPAddress()
	return IPAddress.ip

def isCurrentIPChanged() -> bool:
	publicIP = get_currentIPAddress()
	latestKnowIP = get_latestIPAddressFromDatabase()

	if publicIP != latestKnowIP:
		return True
	elif publicIP == latestKnowIP:
		return False
```
The database management is handled with SQLite. Here's the code for inserting and retrieving IP addresses from the database:
`IPDatabase.py`
```python
from contextlib import nullcontext
import sqlite3
import IPAddress
import Telegram
from sqlite3 import Error
import os.path

def create_connection():
    conn = None
    BASE_DIR = os.path.dirname(os.path.abspath(__file__))
    db_path = os.path.join(BASE_DIR, "IP.db")

    try:
        conn = sqlite3.connect(db_path)
        conn.row_factory = sqlite3.Row
    except Error as e:
        print(e)
    print("Opened database successfully")

    return conn

def insert_newIPAddress(IPAddress):
    conn = create_connection()
    
    values_to_insert = [IPAddress.ip, IPAddress.date]

    query = "INSERT INTO main.IP(ip,date) VALUES(?,?);"

    conn.execute(query, values_to_insert)

    conn.commit()
    print("Records created successfully")
    conn.close()

def get_latestIPAddress():
    conn = create_connection()

	    IPObject = conn.execute("SELECT ID,ip,date FROM main.IP ORDER BY ID DESC;").fetchone()
    
    latest_IP = IPAddress.IPAddress()
    latest_IP.ID = IPObject[0]
    latest_IP.ip = IPObject[1]
    latest_IP.date = IPObject[2]

    if latest_IP is not None:
        return latest_IP.ip
        print("Record loaded successfully")
    else:
        print("Record loaded failed")
        return "1.1.1.1"
    conn.close()
```
The main script  for checking if the WAN IP has changed. If it has, it logs the new IP address and sends a notification:
```python
from datetime import datetime
import IP_Processing
from IPAddress import IPAddress
import time

import Telegram

def main():
     interval = 5

     isIPChanged = IP_Processing.isCurrentIPChanged()

     if isIPChanged == True:
	     IP = IPAddress()
         IP.ip = IP_Processing.get_currentIPAddress()
         IP.date = datetime.today().strftime('%Y-%m-%d %H:%M:%S')
         print(f"IP-address changed to {IP.ip}")

	     IP_Processing.insert_newIPAddress(IP)
	     
		 Telegram.send_telegram_message(f"IPaddress changed to {IPAddress.ip}")
     elif isIPChanged == False:
          print("IPaddress hasn't changed")
     
if __name__ == "__main__": 
```

## Automated Notifications via Telegram

The script uses the `python-telegram-bot` library to send notifications to my phone whenever the WAN IP address changes. This ensures that no matter where I am, I’ll be alerted if the IP address changes:
`Telegram.py`
```python
from telegram import Bot
import asyncio

chat_id = "<CHAT-ID>"
TOKEN = "<BOT-TOKEN>"

bot = Bot(TOKEN)

def send_telegram_message(message):
    asyncio.run(bot.send_message(chat_id, text=message))
```

## Running the script
As this script doesn't need to run all the time, I used cron to run it every two hours as the local user. Al the logging is discarded.  
```bash
0 */2 * * *  python3 /home/<USER>/IPmonitor/Main.py >/dev/null 2>&1
```

# Overengineering with a Telegram Bot
In true overengineering fashion, I also created a Telegram bot used the [echobot example](https://docs.python-telegram-bot.org/en/v21.4/examples.echobot.html) from python-telegram-bot. This allows me to manually check the current WAN IP by sending a simple `/ip` message. Here’s how it works:
```python
#!/usr/bin/env python
import logging

from telegram import ForceReply, Update
from telegram.ext import Application, CommandHandler, ContextTypes, MessageHandler, filters

from WanIP import get_public_ip

logging.basicConfig(
    format="%(asctime)s - %(name)s - %(levelname)s - %(message)s", level=logging.INFO
)

logging.getLogger("httpx").setLevel(logging.WARNING)

logger = logging.getLogger(__name__)

async def ip(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    await update.message.reply_text("Current WAN-ip is " + get_public_ip())

def main() -> None:
    application = Application.builder().token("<BOT-TOKEN>").build()
    application.add_handler(CommandHandler("ip", ip))
    application.run_polling(allowed_updates=Update.ALL_TYPES)

if __name__ == "__main__":
    main()
```

## Running the Telegram-bot script as a service
To make the Telegram bot script run continuously and respond to requests, I needed to set it up as a systemd service. The idea was that the script should always be active and ready to notify me whenever I requested the current WAN IP address. After some research, I found a great [guide](https://github.com/thagrol/Guides/blob/main/boot.pdf) on running a program at startup. The relevant section for my setup was "Installing a Service Per User."

### Enabling User Linger

First things first, I needed to enable "linger" for my user account, allowing it to run background services even when the account wasn't actively logged in. This was a simple command:
`sudo loginctl enable-linger <USER>`

With that done, the local user account was permitted to keep services running persistently, even when logged out, as described in the [loginctl documentation](https://www.freedesktop.org/software/systemd/man/latest/loginctl.html#enable-linger%20USER%E2%80%A6).

### Creating the .service Directory
Next, I created the directory to hold the service configuration files:
`mkdir -p ~/.config/systemd/user`

### Writing the .service File
The service itself is defined in a `.service` file. Here’s what I wrote to configure the systemd service:
```bash
[Unit]
Description=Telegrambot to monitor WAN IP-address
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/home/monitor/TelegramBot/telegrambot.py
Restart=always

[Install]
WantedBy=default.target
WantedBy=multi-user.target
```
This file holds the information that systemd needs to [operate the service](https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files)  in different sections. 
The `[Unit]` section specifies that the bot should start only after the network is fully online. This ensures the bot can communicate via Telegram from the moment the system boots. The `[Service]` section tells systemd to start the Python script that runs the bot, and to automatically restart it if it crashes. Finally, the `[Install]` section configures systemd to include this service in the boot process.

### Deploying the Service
With the `.service` file ready, I copied it into the user-specific system directory:
`$ cp telegrambot.service ~/.config/systemd/user/`

Then, I needed to reload systemd’s internal state so it would recognize the new service:
`$ systemctl --user daemon-reload`

### Enabling and Starting the Service
Once reloaded, I enabled the service to start at boot, and then manually started it to test the setup:
`$ systemctl --user enable telegrambot.service`
`$ systemctl --user start telegrambot.service`

Checking the status of the service showed that it was up and running smoothly:
```bash
● telegrambot.service - Telegrambot to monitor WAN IP-address  
Loaded: loaded (/home/<USER>/.config/systemd/user/telegrambot.service; enabled; preset: enabled)  
Active: active (running) since Mon 2054-01-01 10:11:12 CEST; 5s ago  
Main PID: 12345 (python3)  
Tasks: 2 (limit: 762)  
CPU: 1.079s  
CGroup: /user.slice/user-1000.slice/user@1000.service/app.slice/telegrambot.service  
└─12345 python3 /home/<USER>/TelegramBot/telegrambot.py  
  
Jan 01 10:11:12 monitor systemd[12098]: Started telegrambot.service - Telegrambot to monitor WAN IP-address.  
Jan 01 10:11:12 monitor python3[12346]: 2054-01-01 10:11:12,345 - telegram.ext.Application - INFO - Application started
```
Everything was looking good—my bot was running and waiting for commands!

## The Multi-User Target Issue
However, when I rebooted the Raspberry Pi, I realized the script wasn’t running as expected. After some troubleshooting, I [found](https://unix.stackexchange.com/questions/666509/systemd-service-does-not-start-wantedby-multi-user-target) that the `WantedBy=multi-user.target` line in the `.service` file was causing the issue. It turns out that services defined with `--user` don’t work with `multi-user.target` as they would for system-level services.

```
[Install]
WantedBy=default.target
```

After another reboot, I checked again, and this time, the Telegram bot was running as expected, automatically starting with the system and happily messaging back in the Telegram chat.
![telegram](/assets/telegram.png)

# Finally
With the service now running reliably at startup, my over-engineered solution was finally complete. The Telegram bot automatically monitors and responds to WAN IP requests, even after system reboots. No more wondering if I’ll lose access to my servers again due to ISP outages! All I have to do is message the bot with `/ip`, and it promptly delivers the updated IP address. A perfect blend of automation and control. And perhaps a little more than strictly necessary, but that's how I like it.

_Cover Photo by [Olha Ruskykh](https://www.pexels.com/@olha-ruskykh/)_