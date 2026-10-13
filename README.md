# Snapcast Sayo-E1

A small physical USB controller for [Snapcast](https://github.com/badaix/snapcast), using the **SayoDevice E1** rotary controller.

The goal is simple: control a Snapcast client without using a phone, web browser, computer interface, or Snapweb.

## Features

* 🎛️ Rotate right → volume up
* 🎛️ Rotate left → volume down
* 🖱️ Press the knob → next stream
* 🎵 All Snapcast streams are included, including `idle` streams
* 🔍 Automatic detection of the SayoDevice E1
* ⚙️ Per-machine configuration
* 🔄 Automatic startup with systemd

## Hardware

* SayoDevice E1 / E1 RGB USB controller
* A Linux machine running `snapclient`
* A Snapcast server accessible over the network

The Sayo E1 is detected through its Linux `Consumer Control` input interface.

## How it works

```text
                 ┌──────────────────┐
                 │   SayoDevice E1  │
                 │                  │
                 │  rotation → USB │
                 │  click    → USB │
                 └────────┬─────────┘
                          │
                          ▼
                 ┌──────────────────┐
                 │    snapcast-sayo-e1   │
                 │                  │
                 │  Python / evdev  │
                 └────────┬─────────┘
                          │
                          │ JSON-RPC
                          ▼
                 ┌──────────────────┐
                 │    Snapserver    │
                 │                  │
                 │      :1705       │
                 └────────┬─────────┘
                          │
                          ▼
                 ┌──────────────────┐
                 │  Snapcast client │
                 │                  │
                 │ volume / stream  │
                 └──────────────────┘
```

## Requirements

* Linux
* Python 3
* The Python evdev package : `apt install python3-evdev`
* Snapclient
* Network access to the Snapserver


## Installation

### 1. Install the dependencies and the controller

```bash
git clone https://github.com/kilanmundera/snapcast-sayo-e1
cd snapcast-sayo-e1/
sudo cp snapcast-sayo-e1 /usr/local/bin/snapcast-sayo-e1
sudo chmod +x /usr/local/bin/snapcast-sayo-e1
```

Copy the example configuration and edit it :

```bash
sudo cp snapcast-sayo-e1.conf.example /etc/snapcast-sayo-e1.conf
sudo vim /etc/snapcast-sayo-e1.conf
```
Insert your `CLIENT_ID`
`CLIENT_ID` is the IF of the Snapcast client that should be controlled by the E1 (check on your snapweb page)

Example:

```ini
[Snapcast]

SERVER=snap.local
PORT=1705
CLIENT_ID=dc:a6:32:34:c0:8c
VOLUME_STEP=5
```

### 3. Install the systemd service

```bash
sudo cp snapcast-sayo-e1.service /etc/systemd/system/snapcast-sayo-e1.service
sudo systemctl daemon-reload
sudo systemctl enable snapcast-sayo-e1.service
sudo systemctl start snapcast-sayo-e1.service
```

Follow the logs:

```bash
journalctl -u snapcast-sayo-e1.service -f
```

