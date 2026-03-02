# Headless Raspbian Spotify Player

***IMPORTANT DISCLAIMER***

This document is still a work in progress. It is intended to provide information based on my own personal experiences working with various Spotify solutions for headless RPi environments. There are *many* open-source projects mentioned - in fact, this document merely provides information about my experience working with such projects, and there is *zero* of my own code at play (for now). The goal is to provide a basic outline of the 'path of least resistance' installation for each of the solutions listed below on a typical headless RPi Debian distro; by no means is an exhaustive list of features, configuration options, or installation solutions provided (check the credited open-source projects for comprehensive information about their feature sets).

I have made every effort to credit the appropriate creators and maintainers - any omissions are merely an error on my part, and not an intentional disinclusion. If I have missed crediting or linking one of the tools below, kindly let me know so I may update the document.

If you find this information useful, please check the original repositories and documentation - and support the fantastic developers behind these projects where possible. Thank you :)


## Contents

- [General Knowledge](#general-knowledge)
- [Headless RPi as a LAN Speaker](#headless-rpi-as-a-lan-speaker)
- [Fully-featured Spotify Client for Headless RPi](#fully-featured-spotify-client-for-headless-rpi)
- [Fully-featured Spotify Daemon for Headless RPi](#fully-featured-spotify-daemon-for-headless-rpi)


## General Knowledge

### Sound Devices

All of the solutions mentioned in this document require knowing which 'card' and 'device' your preferred audio output device is assigned on your RPi. This will generally take the form of `hw:x,y`, where `x` is the card and `y` is the device. This might seem unclear to users unfamiliar with Linux, but is actually easy to determine by running `aplay -l` in the terminal. This will give an output that looks something like this (this is the literal output from my latest project running headless Debian Bookworm on a Raspberry Pi 4B):

```bash
**** List of PLAYBACK Hardware Devices ****
card 0: Headphones [bcm2835 Headphones], device 0: bcm2835 Headphones [bcm2835 Headphones]
  Subdevices: 8/8
  Subdevice #0: subdevice #0
  Subdevice #1: subdevice #1
  Subdevice #2: subdevice #2
  Subdevice #3: subdevice #3
  Subdevice #4: subdevice #4
  Subdevice #5: subdevice #5
  Subdevice #6: subdevice #6
  Subdevice #7: subdevice #7
card 1: Device [USB PnP Audio Device], device 0: USB Audio [USB Audio]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 2: vc4hdmi0 [vc4-hdmi-0], device 0: MAI PCM i2s-hifi-0 [MAI PCM i2s-hifi-0]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 3: vc4hdmi1 [vc4-hdmi-1], device 0: MAI PCM i2s-hifi-0 [MAI PCM i2s-hifi-0]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```

This output indicates that the headphone jack corresponds to card 0, device 0 (`hw:0,0`), the third-party plug-and-play USB speaker I'm using is card 1, device 0 (`hw:1,0`), and the remaining two entries refer to the two micro HDMI port (`hw:2,0` and `hw:3,0` respectively).


## Headless RPi as a LAN Speaker

#### This section was last updated in March 2026.

### Overview

[`raspotify`](https://github.com/dtcooper/raspotify) provides an easy-to-implement solution for using a headless RPi system as a speaker for Spotify clients on a shared LAN network. This is conceptually similar to using a Bluetooth speaker for Spotify from your desktop or phone; Spotify does not run on the RPi itself and the RPi will not function as a Spotify Connect device (i.e. the RPi will not be able to control Spotify or playback).

The benefit of this solution is the low bar for installation and usage - the RPi device will simply appear in your other Spotify clients as an available speaker with essentially zero configuration or authentication. If you want to be able to broadcast music from your desktop or phone to an RPi speaker system without needing control from the RPi itself, this is an excellent option.

### Installation

First, install `raspotify` by running:

```bash
sudo apt-get -y install curl && curl -sL https://dtcooper.github.io/raspotify/install.sh | sh
```


Now you need to update the `conf` file for `raspotify` to reflect your desired playback device. This may work for you by default; but if not you can fix this easily. First, open the `conf` file by running:

```bash
sudo nano /etc/raspotify/conf
```

Then uncomment the `LIBRESPOT_ALSA_MIXER_DEVICE` entry and set it to your desired device (see [Sound Devices](#sound-devices)). My preferred device was card 1, device 0; so I updated the `conf` entry as follows:

```toml
LIBRESPOT_ALSA_MIXER_DEVICE="hw:1,0"
```

I found it necessary to update the Systemd service `conf` as well - and you may want to raise or lower the default playback volume as needed. This can be accomplished by running:

```bash
sudo nano /lib/systemd/system/raspotify.service
```

And then scrolling to the bottom of the file to update the `ExecStart` entry:

```toml
ExecStart=/usr/bin/librespot --device "hw:1,0" --initial-volume=100
```

The last step is to reload the daemon and restart `raspotify` so the new configuration can take effect:

```bash
sudo systemctl daemon-reload
sudo systemctl restart raspotify
```

That's it - you should be able to immediately start using your RPi as a LAN speaker - open up Spotify on your desktop or phone and a device called `raspotify (<your-device-name>)` should be available as a speaker. Easy!


## Fully-featured Spotify Client for Headless RPi



## Fully-featured Spotify Daemon for Headless RPi

