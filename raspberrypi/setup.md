---
title: RaspberryPi Setup Notes
description: A few notes that I frequently use to set up my raspberry pi(s)
published: true
date: 2022-06-22T18:17:19.245Z
tags: raspberrypi, setup
editor: markdown
dateCreated: 2022-06-22T18:17:19.245Z
---

# Raspberry Pi Setup

## On-screen 5 inch display

For my RaspberryPi 3B I use a 5 inch display (XPT2046) that neatly fits onto the Pi without much hassle and can serve as a nice status indicator. It also has touch functionality, but I haven't used that yet.

### Correct resolution

The settings that ship with raspbian are not adequate for this display. Here's what works:

/boot/config.txt:
```
hdmi_group=2
hdmi_mode=1
hdmi_mode=87
hdmi_cvt=800 480 60 6 0 0 0
```

### Mirroring any terminal

It's nice to have a display, but I want to be able to manipulate its contents from an SSH connection. 
That's why I set up the Pi to automatically spawn a `tmux` session on startup, so that I can then attach to it from anywhere.

/etc/inittab:
```
T0:23:respawn:/bin/login -f pi tty1 </dev/tty1 >/dev/tty1 2>&1
```

/home/pi/.bashrc:
```
if [ $(tty) == /dev/tty1 ]; then
    tmux
fi
```