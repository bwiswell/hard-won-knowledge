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

#### This section was last updated in March 2026.

### RPi Headless Setup

The solutions outlined below assume that you are using a headless Raspbian system that can be SSH-ed into for CLI purposes. This is typically done by using the [Raspberry Pi Imager](https://www.raspberrypi.com/software/) to install an Raspbian OS onto an SD card using a desktop computer or laptop before inserting the SD card into your RPi. 'Headless' refers to a system that operates without a dedicated monitor, keyboard, or mouse; control is executed over SSH (see [SSH Control](#ssh-control) below). The Raspbian 'Lite' distros available through the Raspberry Pi Imager correspond to headless OS systems. The solutions listed below are likely unnecessary if you're using a full Raspbian distro with a monitor, keyboard, and mouse.

To set up a headless RPi, first download the Raspberry Pi Imager onto your desktop or laptop. Next, insert your SD card into the machine you installed the Imager on (you may need a USB SD card reader depending on your hardware). Open the Raspberry Pi Imager, and follow the prompts in the wizard to select your RPi device type and your SD card as the storage option. Choose a 'Lite' image as per your system needs (generally, the latest 'Lite' 32-bit image is suitable for most purposes). This may require clicking into a `Raspberry Pi OS (other)` menu option to see the full list of available images. The latest image of this type at time of writing is the 32-bit 'Lite' version of Debian Trixie.

When the Imager asks if you would like to customize OS settings, select 'Yes' to setup your system for future SSH use (again, see [SSH Control](#ssh-control)) below. Make sure the following settings are enabled and set correctly for your specific setup:

```
General:
- Hostname: this is going to become the 'name' for your device - pick something memorable that is relevant to your application.
- Set username and password: these are the login credentials you'll use when SSH-ing into your RPi
- Configure wireless LAN: these are your WiFi credentials (the same that you use to login to your WiFi on your computer/phone)

Services:
- Enable SSH: make sure to set this to 'Use password authentication'
```

When you're done editing the settings, click 'Save' and then 'Yes' on the original edit settings prompt. The wizard will warn you that all existing data on your storage device will be erased while flashing the image (this is normal). Take a moment to ensure that you have the *correct* storage device (your SD card) selected, and then select 'Yes'. The image will take a few moments to install, and the software will let you know when it is safe to remove the SD card from your computer. Move the SD card over to your RPi and power it up; it will take several minutes to boot for the first time and connect to your WiFi.

### SSH Control

Headless systems are typically managed and monitored over SSH. Wait for your RPi to boot up and connect to your WiFi before following the steps below. It can be hard to tell when this has happened - sometimes you can monitor devices connected to your LAN via an ISP service like the Xfinity app or a `ping` command from another computer on the network. Usually, this process takes a few minutes the first time RPi boots after a new image is flashed, but subsequent boots will be much more rapid.

#### SSH client on Mac/Linux

Connecting to your RPi via SSH is simple from a Mac or Linux device - just open the terminal and run the following command:

```bash
ssh <username>@<hostname>
```

where the `username` is the username you set in the Raspberry Pi Imager and the `hostname` is the hostname you set - make sure that you use  the form `<devicename>.local`.

#### SSH client on Windows

On Windows systems, it's usually best to install a dedicated SSH program such as [MobaXterm](https://mobaxterm.mobatek.net/) or [PuTTY](https://putty.software/). In either case, you will prompted to enter a `hostname`. As above, the hostname must take the form `<devicename>.local`. `MobaXterm` will also require filling the `username` field, on `PuTTY` fill the `hostname` field with `<username>@<hostname>`. The `port` field will likely be prefilled, but if not then you can enter the standard `22` port used for SSH. Once you've entered the `hostname`, click `Open` or `Connect`.

At this point, you should be prompted for your password (the one you set in Raspberry Pi Imager). Enter it, hit 'Return', and you should be logged into the RPi.

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

First, install `raspotify` on your RPi by running:

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

Getting a fully-featured Spotify client to run on a headless system can be difficult due to the lack of an official Linux Spotify client and the need to authenticate through Spotify's website - something that is not typically possible through a terminal system alone. The solution described below uses [spotify_player](https://github.com/aome510/spotify-player) installed via [Cargo](https://github.com/rust-lang/cargo), [tmux](https://github.com/tmux/tmux), and [Carbonyl](https://github.com/fathyb/carbonyl) installed via [NPM](https://www.npmjs.com/) to achieve a visual browser with JavaScript support to enable authentication in the terminal while keeping the `spotify_player authenticate` process alive.

### Cargo

`Cargo` is package manager for the `Rust` programming language, which can be used to download and build `spotify_player`. `Cargo` has a convenience script that makes installation easy:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

If prompted for which installation option you want, choose 'default' (option 1). When the Cargo installation is complete, you'll need to restart your RPi:

```bash
sudo reboot
```

This will close the current SSH session. Wait for the RPi to reboot, SSH back in ([SSH Control](#ssh-control)), and you should be all set.

### tmux

`tmux` is a terminal multiplexer that allows for multiple terminal 'windows' to be active at the same time through a single terminal instance. Installation on Raspbian is incredibly simple:

```bash
sudo apt update
sudo apt install tmux
```

This will come in handy in a few steps.

### Carbonyl

`Carbonyl` is a (mindblowingly) feature-rich visual browser for the terminal environment. Importantly, it supports JavaScript - which is necessary to complete the web-based authentication flow for Spotify. The easiest path for installing and running `Carbonyl` on a Raspbian system (in my experience) is via `NPM`. First, install `NPM` (there are better ways to do this if you plan on using `NPM` heavily or with different versions - see [NVM](https://github.com/nvm-sh/nvm)):

```bash
sudo apt update
sudo apt install nodejs npm
```

Then use `NPM` to install `Carbonyl` globally:

```bash
npm install --global carbonyl
```

That should set you up to proceed with installing and authenticating `spotify_player`.

### spotify_player

First, install the dependencies needed by `spotify_player`:

```bash
sudo apt update
sudo apt install libssl-dev libasound2-dev libdbus-1-dev
```

Now you're ready to install `spotify_player`. There are several interesting optional features that you may want to include, so be sure to peruse the documentation thoroughly before installing (this can sometimes take *up to an hour* to install and build - so get it right the first time). Note that while other audio backends are possible, this document assumes that `ALSA` will be used as the audio backend.

#### Image feature

If you want to be able to display (heavily pixelated) album art images via the client, you'll want to use the `image` feature, probably with `pixelate`. Unless you have a specific reason to use a different image rendering solution, passing the `pixelate` feature option is sufficient (`image` will *automatically* be added if `pixelate` is present as a feature, so there's no need to add it to the feature list as well).

#### Daemon feature

If you want to be able to run `spotify_player` in the background (i.e. as a daemon, see [Fully-featured Spotify Daemon for Headless RPi](#fully-featured-spotify-daemon-for-headless-rpi)) you'll need to add the `daemon` optional feature.

#### Fuzzy search feature

Fuzzy search will allow you to find songs, artists, etc. even when you have small typos or mistakes in your search strings. This feature can be enabled by adding the `fzf` feature.

Once you've decided on a feature set, run the install command:

```bash
cargo install spotify_player --features alsa-backend
```

To add in your optional features, just append them to the end of the command in a comma-separated list without spaces. For example:

```bash
cargo install spotify_player --features alsa-backend,pixelate,daemon,fzf
```

This can take a looooooong time to install, so go grab lunch and come back in an hour or so.

### Authentication

Now it's time to authenticate `spotify_player` - here's where installing `tmux` and `Carbonyl` comes in handy. First, start a `tmux` session by running:

```bash
tmux
```

Yup.

Now you need to 'split' your terminal: hit `Ctrl+B`, release, and then press `"`. Your terminal should now be split horizontally into two windows. The bottom window will now have focus; you can switch between windows by pressing `Ctrl+B`, releasing, and then the 'up' or 'down' arrow key.

In one of your terminal windows, run:

```bash
spotify_player authenticate
```

This will spit out a URL and ask you to navigate to it. Copy the URL and switch to the other `tmux` window. In this window, run:

```bash
carbonyl <your-url>
```

Spotify's auth site should open (this can be slow - just be patient). Things will likely be a little 'blocky', but functional - you can complete the Spotify auth flow as you usually would by entering your username, pressing 'Submit', and then completing the 2FA challenge code sent to your phone/email. Once you're done, you should be able to observe the `spotify_player authenticate` process in the first window complete automatically.

Once you're authenticated, you can use `Ctrl+D` to end the `tmux` session. Now just run:

```bash
spotify_player
```

to open the Spotify client. Happy listening!

## Fully-featured Spotify Daemon for Headless RPi

This is pretty easy - first, follow the steps above in [Fully-featured Spotify Client for Headless RPi](#fully-featured-spotify-client-for-headless-rpi) and make sure to use the `daemon` build option ([Daemon feature](#daemon-feature)).

If you want to *manually* start the daemon every time, you can simply run `spotify_player -d`. If you want the daemon to automatically start on boot, you'll need to create a service `conf` file. First, create and open a new service file:

```bash
sudo nano /etc/systemd/system/spotify-player.service
```

Add a single line to your new file:

```toml
ExecStart=/home/bwzlr/.cargo/bin/spotify_player
```

Save and exit `nano`. Then reload `systemd`, enable the service, and start it up:

```bash
sudo systemctl daemon-reload
sudo systemctl enable spotify-player.service
sudo systemctl start spotify-player.service
```

That should do it :)