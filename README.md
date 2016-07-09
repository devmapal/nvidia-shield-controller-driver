# NVidia Shield Controller Driver
A Work In Progress Linux driver for the NVidia SHIELD Controller over WLAN.

The SHIELD Controller uses ozproto (Ethertype 0x892E), exposing an USB HID (the
controller) and 3 USB audio devices (headphone audio sink, headphone audio
source if it has a microphone and controller internal audio source
(microphone).

## Building the driver
Apply the patch set in `driver` to your kernel sources (currently only tested
with a 4.4 kernel) and make sure the following configuration options are set.
```
CONFIG_USB_HID=y
CONFIG_STAGING=y
CONFIG_USB_WPAN_HCD=m
CONFIG_SND_USB_AUDIO=y (Optional, for audio support)
```

You probably also want
```
CONFIG_INPUT_JOYDEV=y
CONFIG_INPUT_EVDEV=y
```

## Usage
The SHIELD Controller uses WPS Push Button method for pairing, with the PC
setting itself as the autonomous Group Owner. Pairing the controller using
wpa_supplicant works as follows.

```
# wpa_cli -i wlp3s0 p2p_group_add
# wpa_cli -i p2p-wlp3s0-0 wps_pbc
```

Now hold the Nvidia button on the controller until it flashes. When the button
stops flashing, pairing was successful.

You can now load the driver with

```
# modprobe ozwpan g_net_dev=p2p-wlp3s0-0
```

After a few seconds, the controller should be available.

```
# lsusb
...
Bus 003 Device 002: ID 0955:7210 NVidia Corp.
...

# lsusb -t
...
/:  Bus 03.Port 1: Dev 1, Class=root_hub, Driver=ozwpan/8p, 12M
    |__ Port 1: Dev 2, If 0, Class=Human Interface Device, Driver=usbhid, 12M
    |__ Port 1: Dev 2, If 1, Class=Audio, Driver=snd-usb-audio, 12M
    |__ Port 1: Dev 2, If 2, Class=Audio, Driver=snd-usb-audio, 12M
    |__ Port 1: Dev 2, If 3, Class=Audio, Driver=snd-usb-audio, 12M
...
```

Note that these instructions assume that wireless interface created is called
`p2p-wlp3s0-0`.

## Choppy audio
If audio is choppy, try the following PulseAudio configuration.

In `/etc/pulse/daemon.conf`
```
default-sample-rate = 32000
default-fragments = 5
default-fragment-size-msec = 2
```

In `/etc/pulse/default.pa`
```
load-module module-udev-detect tsched=0
```

## Wireshark dissector
`wireshark-dissector` contains an ozproto wireshark dissector based on
https://github.com/chunkeey/ozwpan/blob/master/wireshark/0001-OZWPAN-add-initial-OZWPAN-support.patch
