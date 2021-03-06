# Dash Listener

A simple Python 3 script that listens for an Amazon Dash button connecting to the
network and prints to stdout. Output can be piped into another program, such as
one that turns Philips Hue lights on or off.

## Inspiration

The idea and techniques behind the implementation for this came from this
[blog post](https://medium.com/@edwardbenson/how-i-hacked-amazon-s-5-wifi-button-to-track-baby-data-794214b0bdd8).

## Philips Hue Adapter Setup

In order to use the phue package, you need to configure it. One way to do that is:

1. Press your hue bridge's button
2. Within 30 seconds, execute the following in the python3 shell
```python
from phue import Bridge
b = Bridge('<your hue bridge IP here>')
b.get_api()
exit()
```

The commands will create a `.python_hue` file which has your authentication details.
If you plan to run the scripts as root, copy `~/.python_hue` to the root directory.
On the Raspberry Pi running Raspbian, this is apparently `/root/`.

## Constants

Edit the contents of `constants.py` to match your constants. You can add as
many button MACs as you'd like to the array. I'd recommend using the amazon-dash
python library to determine your dash buttons' MAC addresses.

You can remove the `HUE_ROOM` constant if you'd like the service to control all lights.

## Quick Usage

First, install dependencies:

```
$ pip install -r requirements.txt
```

For use with the Philips Hue adapter:

```
$ (sudo ./arp_listener.py) | ./hue_adapter.py
```

Currently, this script only runs on Linux (not BSD/OS X) systems, as these are
the only systems whose socket APIs support `AF_PACKET`. Note that the first
script, `arp_listener.py`, requires root privileges because it opens a raw
socket. Since root access is generally scary, the only things the script does
are listening for packets and publishing any ARP packets that contain a
particular MAC address.

## systemd Service Setup

Edit the `dash-listeners.service.app` file to have the proper `WorkingDirectory`,
which should be the folder that you cloned this repository into. Then run
```bash
sudo make install
```

That's it. You can control the service using `systemctl` now.

## How this works

This is better explained in the above-linked blog post, but here is a short
description.

The [Amazon Dash button](http://www.amazon.com/b/?node=10667898011&lo=digital-text)
is a branded WiFi button that, when pushed, will order a pre-set household good
from Amazon. To save battery, these are designed to be off until the button is
pushed. When the button is pushed, the device connects to the WiFi network.

Whenever a device connects to a network, it announces itself by broadcasting
ARP probes to every host on the network. These packets can be picked up and,
upon examination, can be tied to a particular device, which in this case is the
Dash button. Thus, we can set up tasks that can be triggered when a particular
ARP packet is found.

In my particular project, the ARP probe with the MAC address of the button
is picked up by `arp_listener.py`. The script outputs this information to
stdout, which is piped to stdin of another script, `hue_adapter.py`. The
second script reads the status of my light bulbs from the bridge (the device on
my network that manages the bulbs) and either turns my lights on or off.
