---
type: post
title:  "Getting My Bluetooth Headphones working with 16.04"
date:   2017-07-25 10:05:55 -0600
categories: linux
---

I have had a lot of issues getting my Bose Soundlink Headphones working in 16.04.  I had gotten it working before, after looking at several websites, but I could never reproduce it working.

I just got it working again after a fresh install of 16.04, so I though I would compile a little list of instructions to get it working.

## Disable LE bluetooth

 edit `/etc/bluetooth/main.conf` and change (starting at line 48) [source link](https://askubuntu.com/questions/833322/pair-bose-quietcomfort-35-with-ubuntu-over-bluetooth)
```bash
 # Restricts all controllers to the specified transport. Default value
# is "dual", i.e. both BR/EDR and LE enabled (when supported by the HW).
# Possible values: "dual", "bredr", "le"
# ControllerMode = dual
```
to
``` bash
 # Restricts all controllers to the specified transport. Default value
# is "dual", i.e. both BR/EDR and LE enabled (when supported by the HW).
# Possible values: "dual", "bredr", "le"
ControllerMode = bredr
```

At this point you should be able to pair your headphones with `blueman-manager` (`sudo apt install blueman`), but it will only work with HSA, and if you try to switch to a2dp, it will say something like `failed to change profile to ad2p-sink`.


## Change when pulseaudio loads the bluetooth discover module

edit `/etc/pulse/default.pa` and disable the bluetooth discover module by commenting out that block (starting at line 73)

```bash
.ifexists module-bluetooth-discover.so
load-module module-bluetooth-discover
.endif
```

to

```bash
# .ifexists module-bluetooth-discover.so
# load-module module-bluetooth-discover
# .endif
```

Now, edit `/usr/bin/start-pulseaudio-x11` to load the discover module after the session manager is already loaded.  After the block starting at line 34, add the following lines so it looks like this:

```bash
    if [ x"$SESSION_MANAGER" != x ] ; then
      /usr/bin/pactl load-module module-x11-xsmp "display=$DISPLAY session_manager=$SESSION_MANAGER" > /dev/null
    fi

    # Added per https://bbs.archlinux.org/viewtopic.php?id=194006
    /usr/bin/pactl load-module module-bluetooth-discover
```

After restarting my machine, I was able to pair my headphones, and I could enable the ad2p sink as well!  Yay!

## Automatically switch audio device when connecting headphones

Just add

```bash
    load-module module-switch-on-connect
```

to `/etc/pulse/default.pa` [original link](https://askubuntu.com/questions/158241/automatically-change-sound-input-output-device)
