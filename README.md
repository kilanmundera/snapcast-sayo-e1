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

The script does not rely on a fixed `/dev/input/eventX` number, since this number may change after a reboot or when the USB device is reconnected.

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

## Controls

| E1 action                | Snapcast action |
| ------------------------ | --------------- |
| Rotate clockwise         | Volume +        |
| Rotate counter-clockwise | Volume -        |
| Press knob               | Next stream     |
| Hold button              | Ignored         |
| Rotate while holding     | Ignored         |
| `KEY_NEXTSONG`           | Ignored         |
| `KEY_PREVIOUSSONG`       | Ignored         |

## Requirements

* Linux
* Python 3
* The Python evdev package : `apt install python3-evdev`
* Snapclient
* Network access to the Snapserver


## Installation

### 1. Install the controller

Copy the script to:

```bash
sudo cp snapcast-sayo-e1 /usr/local/bin/snapcast-sayo-e1
sudo chmod +x /usr/local/bin/snapcast-sayo-e1
```

### 2. Create the configuration

Copy the example configuration:

```bash
sudo cp snapcast-sayo-e1.conf.example /etc/snapcast-sayo-e1.conf
```

Edit it:

```bash
sudo nano /etc/snapcast-sayo-e1.conf
```

Example:

```ini
[Snapcast]

SERVER=snap.local
PORT=1705
CLIENT_ID=dc:a6:32:34:c0:8c
VOLUME_STEP=5
```

`CLIENT_ID` is the Snapcast client that should be controlled by the E1 (check on your snapweb page)

The corresponding Snapcast group is detected automatically.

### 3. Install the systemd service

```bash
sudo cp snapcast-sayo-e1.service /etc/systemd/system/snapcast-sayo-e1.service
```

Reload systemd:

```bash
sudo systemctl daemon-reload
```

Enable the service at boot:

```bash
sudo systemctl enable snapcast-sayo-e1.service
```

Start it:

```bash
sudo systemctl start snapcast-sayo-e1.service
```

Check its status:

```bash
systemctl status snapcast-sayo-e1.service
```

Follow the logs:

```bash
journalctl -u snapcast-sayo-e1.service -f
```

## Multiple Snapcast clients

The same script can be installed on multiple machines.

Each machine simply has its own `/etc/snapcast-sayo-e1.conf`.

For example:

### Kitchen

```ini
[Snapcast]
SERVER=snap.local
PORT=1705
CLIENT_ID=b8:27:eb:72:2a:7d
VOLUME_STEP=5
```

### Bathroom

```ini
[Snapcast]
SERVER=snap.local
PORT=1705
CLIENT_ID=dc:a6:32:34:c0:8c
VOLUME_STEP=5
```

### Living room

```ini
[Snapcast]
SERVER=snap.local
PORT=1705
CLIENT_ID=a6:80:8f:ab:dd:56
VOLUME_STEP=5
```

The Python script and systemd service remain identical on every machine.


## Automatic E1 detection

The controller automatically searches `/dev/input/event*` for the SayoDevice E1 `Consumer Control` interface.


## Streams

Pressing the E1 knob switches to the next Snapcast stream.

The script retrieves **all streams** from the Snapserver, regardless of their current state.

For example:

```text
Radio A
Radio B
Spotify
Airplay
MPD
```

Streams marked as `idle` are intentionally included.

The cycling order follows the order returned by Snapcast's `Server.GetStatus` API.

## Example startup output

```text
CLIENT CONFIGURED : dc:a6:32:34:c0:8c
Associated group  : 8ab1c6a1-8ed2-de9c-f9dd-c86e199f5def
Initial volume    : 7%
Initial stream    : FIP

Available streams:
  1. FIP  <-- current
  2. France Culture
  3. France Inter
  4. Pomme dApi
  5. Spotify
  6. Airplay
  7. MPD

E1 detected      : /dev/input/event7
Device           : SayoDevice SayoDevice E1 Consumer Control

Controller ready.
  Rotate right : volume +
  Rotate left  : volume -
  Click        : next stream
```

## Snapcast JSON-RPC

`snapcast-sayo-e1` communicates with Snapserver through its JSON-RPC interface.

The default port is:

```text
1705
```

The main methods used are:

```text
Server.GetStatus
Client.SetVolume
Group.SetStream
```

